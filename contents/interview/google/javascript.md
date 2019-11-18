---
layout: page
title: Javascript
permalink: /google-javascript/
---

### Variables 

~~~javascript 
var a = 'word';
var b = false;
var c = true;
var d = 0
var e = 1
var f = 2
var g = null

console.log(a || b); // 'word'
console.log(c || a); // true
console.log(b || a); // 'word'
console.log(e || f); // 1
console.log(f || e); // 2
console.log(d || g); // null
console.log(g || d); // 0
console.log(a && c); // true
console.log(c && a); // 'word'
~~~

### Objects 

~~~javascript 
const bankAccount = (initialBalance) => {
  var balance = initialBalance;

  return {
    getBalance: function() {
      return balance;
    },
    deposit: function(amount) {
      balance = balance + amount;
      return balance;
    },
  };
};

const account = bankAccount(100);

console.log(account.getBalance()); // 100
console.log(account.deposit(10)); // 110
~~~

### Closure 


### Promise 

~~~javascript 
const isGreater = (a, b) => {
 return new Promise ((resolve, reject) => {
  if(a > b) {
   resolve(true)
  } else {
   reject(false)
  }
 })
}
isGreater(1, 2)
 .then(result => {
    console.log('greater')
 })
 .catch(result => {
    console.log('smaller')
 })
~~~

### Hoisting 
- In JavaScript, Hoisting is the default behavior of moving all the declarations at the top of the scope before code execution. 
    - Basically, it gives us an advantage that no matter where functions and variables are declared, they are moved to the top of their scope regardless of whether their scope is global or local.
- It allows us to call functions before even writing them in our code.
- JavaScript only hoists declarations, not the initialisations.
- Declaration –> Initialisation/Assignment –> Usage
- Hoisting is a term you will not find used in any normative specification prose prior to ECMAScript® 2015 Language Specification. 
    - Hoisting was thought up as a general way of thinking about how execution contexts (specifically the creation and execution phases) work in JavaScript. 
    - However, the concept can be a little confusing at first.
- Conceptually, for example, a strict definition of hoisting suggests that variable and function declarations are physically moved to the top of your code, but this is not in fact what happens. 
- Instead, the variable and function declarations are put into memory during the compile phase, but stay exactly where you typed them in your code.
- One of the advantages of JavaScript putting function declarations into memory before it executes any code segment is that it allows you to use a function before you declare it in your code. For example:
- Hoisting works well with other data types and variables. The variables can be initialized and used before they are declared.
~~~javascript
//Example 1 - Does not hoist
var x = 1; // Initialize x
console.log(x + " " + y); // '1 undefined'
var y = 2; // Initialize y
//This will not work as JavaScript only hoists declarations

//Example 2 - Hoists
var num1 = 3; //Declare and initialize num1
num2 = 4; //Initialize num2
console.log(num1 + " " + num2); //'3 4'
var num2; //Declare num2 for hoisting

//Example 3 - Hoists
a = 'Cran'; //Initialize a
b = 'berry'; //Initialize b
console.log(a + "" + b); // 'Cranberry'
var a, b; //Declare both a & b for hoisting
~~~

### IIFM 


### Strict Mode 
- The strict mode in JavaScript does not allow following things:
    - Use of undefined variables
    - Use of reserved keywords as variable or function name
    - Duplicate properties of an object
    - Duplicate parameters of function
    - Assign values to read-only properties
    - Modifying arguments object
    - Octal numeric literals with statement
    - eval function to create a variable

### Prototype 
- JavaScript is a dynamic language. You can attach new properties to an object at any time as shown below.
~~~javascript 
function Student() {
    this.name = 'Tree';
    this.gender = 'Male';
}

var studObj1 = new Student();
studObj1.age = 22; // New property !!!
console.log(studObj1.age); // 22

var studObj2 = new Student();
console.log(studObj2.age); // undefined
~~~