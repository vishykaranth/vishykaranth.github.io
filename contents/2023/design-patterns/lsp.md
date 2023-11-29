---
layout: page
title: design-patterns-lsp
permalink: /design-patterns-lsp/
---


## LSP 

### Basics 
- At a high level, the LSP states that in an object-oriented program, if we substitute a superclass object reference with an object of any of its subclasses, the program should not break.
- The LSP is applicable when there’s a supertype-subtype inheritance relationship by either extending a class or implementing an interface. We can think of the methods defined in the supertype as defining a contract. Every subtype is expected to stick to this contract. If a subclass does not adhere to the superclass’s contract, it’s violating the LSP.
- How can a method in a subclass break a superclass method’s contract? There are several possible ways:
    - Returning an object that’s incompatible with the object returned by the superclass method.
    - Throwing a new exception that’s not thrown by the superclass method.
    - Changing the semantics or introducing side effects that are not part of the superclass’s contract.
- Java and other statically-typed languages prevent 1 (unless we use very generic classes like Object) and 2 (for checked exceptions) by flagging them at compile-time. It’s still possible to violate the LSP in these languages via the third way.

### Why is the LSP Important?
- LSP violations are a design smell. We may have generalized a concept prematurely and created a superclass where none is needed. Future requirements for the concept might not fit the class hierarchy we have created.
- If client code cannot substitute a superclass reference with a subclass object freely, it would be forced to do instanceof checks and specially handle some subclasses. If this kind of conditional code is spread across the codebase, it will be difficult to maintain.
- Every time we add or modify a subclass, we would have to comb through the codebase and change multiple places. This is difficult and error-prone.
- It also defeats the purpose of introducing the supertype abstraction in the first place which is to make it easy to enhance the program.
- It may not even be possible to identify all the places and change them - we may not own or control the client code. We could be developing our functionality as a library and providing them to external users, for example.


### Fixing the Design

#### Mantra 
- Let’s revisit the design and create supertype abstractions only if they are general enough to create code that is flexible to requirement changes. We will also use the following object-oriented design principles:
    - Program to interface, not implementation
    - Encapsulate what varies
    - Prefer composition over inheritance

#### New Design Implementation 
- To start with, what we can be sure of is that our application needs to collect payment - both at present and in the future. It’s also reasonable to think that we would want to validate whatever payment details we collect. Almost everything else could change. So let’s define the below interfaces:

~~~java
interface IPaymentInstrument  {
  void validate() throws PaymentInstrumentInvalidException;
  PaymentResponse collectPayment() throws PaymentFailedException;
}
class PaymentResponse {
  String identifier;
}
~~~

- PaymentResponse encapsulates an identifier - this could be the fingerprint for credit and debit cards or the card number for rewards cards. It could be something else for a different payment instrument in the future. The encapsulation ensures IPaymentInstrument can remain unchanged if future payment instruments have more data.
- PaymentProcessor class now looks like this:

~~~java 
class PaymentProcessor {
  void process(
      OrderDetails orderDetails, 
      IPaymentInstrument paymentInstrument) {
    try {
      paymentInstrument.validate();
      PaymentResponse response = paymentInstrument.collectPayment();
      saveToDatabase(orderDetails, response.getIdentifier());
    } catch (...) {
      // exception handling
    }
  }

  void saveToDatabase(OrderDetails orderDetails, String identifier) {
    // save the identifier and order details in DB
  }
}
~~~

- There are no runFraudChecks() and sendToPaymentGateway() calls in PaymentProcessor anymore - these are not general enough to apply to all payment instruments.
- Let’s add a few more interfaces for other concepts which seem general enough in our problem domain:

~~~java
interface IFraudChecker {
  void runChecks() throws FraudDetectedException;
}
interface IPaymentGatewayHandler {
  PaymentGatewayResponse handlePayment() throws PaymentFailedException;
}
interface IPaymentInstrumentValidator {
  void validate() throws PaymentInstrumentInvalidException;
}
class PaymentGatewayResponse {
  String fingerprint;
}
~~~

- And here are the implementations:

~~~java
class ThirdPartyFraudChecker implements IFraudChecker {
  // members omitted
  
  @Override
  void runChecks() throws FraudDetectedException {
    // external system call omitted
  }
}
class PaymentGatewayHandler implements IPaymentGatewayHandler {
  // members omitted
  
  @Override
  PaymentGatewayResponse handlePayment() throws PaymentFailedException {
    // send details to payment gateway (PG), set the fingerprint
    // received from PG on a PaymentGatewayResponse and return
  }
}
class BankCardBasicValidator implements IPaymentInstrumentValidator {
  // members like name, cardNumber etc. omitted

  @Override
  void validate() throws PaymentInstrumentInvalidException {
    // basic validation on name, expiryDate etc.
    if (name == null || name.isEmpty()) {
      throw new PaymentInstrumentInvalidException("Name is invalid");
    }
    // other basic validations
  }
}
~~~

- Let’s build CreditCard and DebitCard abstractions by composing the above building blocks in different ways. We first define a class that implements IPaymentInstrument :

~~~java
abstract class BaseBankCard implements IPaymentInstrument {
  // members like name, cardNumber etc. omitted
  // below dependencies will be injected at runtime
  IPaymentInstrumentValidator basicValidator;
  IFraudChecker fraudChecker;
  IPaymentGatewayHandler gatewayHandler;

  @Override
  void validate() throws PaymentInstrumentInvalidException {
    basicValidator.validate();
  }

  @Override
  PaymentResponse collectPayment() throws PaymentFailedException {
    PaymentResponse response = new PaymentResponse();
    try {
      fraudChecker.runChecks();
      PaymentGatewayResponse pgResponse = gatewayHandler.handlePayment();
      response.setIdentifier(pgResponse.getFingerprint());
    } catch (FraudDetectedException e) {
      // exception handling
    }
    return response;
  }
}
class CreditCard extends BaseBankCard {
  // constructor omitted
  
  @Override
  void validate() throws PaymentInstrumentInvalidException {
    basicValidator.validate();
    // additional validations for credit cards
  }
}
class DebitCard extends BaseBankCard {
  // constructor omitted
}
~~~

- Though CreditCard and DebitCard extend a class, it’s not the same as before. Other areas of our codebase now depend only on the IPaymentInstrument interface, not on BaseBankCard. Below snippet shows CreditCard object creation and processing:

~~~java
IPaymentGatewayHandler gatewayHandler = 
  new PaymentGatewayHandler(name, cardNum, code, expiryDate);

IPaymentInstrumentValidator validator = 
  new BankCardBasicValidator(name, cardNum, code, expiryDate);

IFraudChecker fraudChecker = 
  new ThirdPartyFraudChecker(name, cardNum, code, expiryDate);

CreditCard card = 
  new CreditCard(
    name,
    cardNum,
    code,
    expiryDate,
    validator,
    fraudChecker,
    gatewayHandler);

paymentProcessor.process(order, card);
~~~

- Our design is now flexible enough to let us add a RewardsCard - no force-fitting and no conditional checks. We just add the new class and it works as expected.

~~~java
class RewardsCard implements IPaymentInstrument {
  String name;
  String cardNumber;

  @Override
  void validate() throws PaymentInstrumentInvalidException {
    // Rewards card related validations
  }

  @Override
  PaymentResponse collectPayment() throws PaymentFailedException {
    PaymentResponse response = new PaymentResponse();
    // Steps related to rewards card payment like getting current 
    // rewards balance, updating balance etc.
    response.setIdentifier(cardNumber);
    return response;
  }
}
~~~

- And here’s client code using the new card:

~~~java
RewardsCard card = new RewardsCard(name, cardNum);
paymentProcessor.process(order, card);
~~~


#### Advantages of the New Design
- The new design not only fixes the LSP violation but also gives us a loosely-coupled, 
    - flexible set of classes to handle changing requirements. 
    - For example, adding new payment instruments like Bitcoin and Cash on Delivery is easy 
    - we just add new classes that implement IPaymentInstrument.
- Business needs debit cards to be processed by a different payment gateway? 
    - No problem - we add a new class that implements IPaymentGatewayHandler and inject it into DebitCard. 
    - If DebitCard’s requirements begin to diverge a lot from CreditCard’s, 
        - we can have it implement IPaymentInstrument directly instead of extending BaseBankCard - no other class is impacted.
- If we need an in-house fraud check for RewardsCard, 
    - we add an InhouseFraudChecker that implements IFraudChecker, 
    - inject it into RewardsCard and only change RewardsCard.collectPayment().     
- The LSP is a very useful idea to keep in mind both when developing a new application and when enhancing or modifying an existing one.
- When designing the class hierarchy for a new application, the LSP helps make sure that we are not prematurely generalizing concepts in our problem domain.
- When enhancing an existing application by adding or changing a subclass, being mindful of the LSP helps ensure that our changes are in line with the superclass’s contract and that the client code’s expectations continue to be met.           