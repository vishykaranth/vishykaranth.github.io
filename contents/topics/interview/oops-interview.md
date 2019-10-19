## Object-Oriented Design Interview Questions Developers Should Know
- Object-Oriented Design (OOD) skills are a major plus for software engineers. 
- They give interviewers an idea about the following:
    - Whether the candidate can translate a complex problem into a concrete set of objects 
        - and identify interactions among those objects to solve the problem at hand.
    - Whether the candidate can identify patterns while designing 
        - and, wherever applicable, effectively apply time-tested solutions instead of re-inventing the wheel.
### The approach to OOD interview questions:
- In Object Oriented Design questions, 
    - interviewers are looking for your understanding of the nuances of complex problems 
    - and your ability to transform the requirements into comprehensible Classes.
- OOD questions generally will all follow a very similar pattern. 
    - You will be provided with a vague problem and a set of constraints for a system to design, and very little else. 
    - It is then up to you, the candidate, to figure out the “level” of solution that the interviewer is looking for, 
    - what kind of functionality will be needed, and come up with a workable solution.
- Interviewers are looking for one main thing: finding the right balance between a solution that works immediately and is also adaptable to change in the future.
- To simplify things, you can take the following approach for any OOD question you encounter:
    - Clarify the requirements: 
        - Make sure you understand the expectations of the interviewer. 
        - Ask clarifying questions if at all necessary — the interviewer will not mind, and will likely appreciate it. 
        - For example, “are you looking for me to demonstrate the structure of a solution, or to fully implement it?” 
        - Doing this here will take about 5–10 seconds, but save tremendous amounts of time later.
    - Hash out the primary use cases: 
        - Think about, and then talk through, use cases. 
        - Make sure you understand all the different functionality your system is expected to have. 
        - Talking about it out loud can also help you to come across expectations or ideas you might not have realized if you just jumped right in.
    - Identify key Objects: 
        - Now, identify all the objects that will play a role in your solution. 
        - For example, if you’re designing a parking lot, these will be things like vehicles, parking spots, parking garages, entrances, exits, garage operators, etc.
    - Identify Operations supported by Objects: 
        - Work out all the behaviors you’d expect each object that you identified in the previous step to have. 
        - For example, a car should be able to move, park in a given spot, and hold a license plate. 
        - A parking spot should be able to accommodate a two-wheeled vehicle or a four-wheeled vehicle — and so on.
    - Identify Interactions between Objects: 
        - Map out the relationships between the different objects that will need to interface with each other. 
        - This is where it all comes together. 
        - For example, a car should be able to park in a parking spot. 
        - Parking garages should be able to fit multiple parking spots, and so on.
    - I’ll now walk through some of the top questions I’d recommend practicing. 
    -   For each one, I’ll also share some pointers about things the interviewer will probably be looking for in your answer to such a question.
### Usecases 
#### Design Amazon / Flipkart (an online shopping platform)
- Beyond the basic functionality (signup, login etc.), interviewers will be looking for the following:
- Discoverability: 
    - How will the buyer discover a product? 
    - How will the search surface results?
- Cart & Checkout: 
    - Users expect the cart and checkout to behave in a certain way. 
    - How will the design adhere to such known best practices while also introducing innovative checkout semantics like One-Click-Purchase?
- Payment Methods: 
    - Users can pay using credit cards, gift cards, etc. 
    - How will the payment method work with the checkout process?
- Product Reviews & Ratings: 
    - When can a user post a review and a rating? 
    - How are useful reviews tracked and less useful reviews de-prioritized?
#### Design a Movie Ticket Booking System
- Duplication: 
    - How are you handling instances, such as the same cinema having multiple cinema halls showing different movies simultaneously? 
    - Or the same movie being shown at different times in the same cinema/hall?
- Payment Handling: 
    - What would be the process for a user to purchase a ticket?
- Selection: 
    - How would user a pick a seat, ensuring it’s not already booked by someone else?
- Price Variances: 
    - How would discounted pricing be considered? For example, for students or children.
#### Design an ATM
- Overdrawing: 
    - What would you do when the ATM doesn’t have any cash left?
- Pin Verification: 
    - What if a user enters a wrong PIN multiple times?
- Card Reading: 
    - How would you detect if the card has been correctly inserted or not?
#### Design an Airline Management System
- Itinerary Complexity: 
    - How would multi-flight itineraries work? 
    - How would multiple passengers on the same itinerary be handled?
- Alerts: 
    - How are customers notified if there’s a change to the flight status?
- External Access: 
    - How would the system interact with other actors making reservations to the same flights, such as a front-desk operator for an airline?
#### Design Blackjack (a card game)
- Scoring: 
    - On what level of the system is scoring handled? 
    - What are the advantages and disadvantages of this?
- Rules: 
    - What kind of flexibility exists for playing with slightly different house rules if needed?
- Betting: 
    - How are bet payouts handled? 
    - How are odds factored in?
#### Design a Hotel Management System
- Room Complexity: 
    - How will the system support different room types within the same hotel?
- Alerts: 
    - How will the system remind users that their check-in date is approaching? 
    - What other alerts might be useful to factor in?
- Customization: 
    - How would users make special requests on their room? 
    - What kind of special requests would be supported?
- Cancellation / Modification: 
    - How would the system treat booking cancellation (within the allowed time period)? 
    - What about other changes? 
    - What types of modifications would be covered?
#### Design a Parking Lot
- Payment Flexibility: How are customers able to pay at different points (i.e. either at the customer’s info console on each floor or at the exit) and by different methods (cash, credit, coupon)?
- Capacity: How will the parking capacity of each lot be considered? What happens when a lot becomes full?
- Vehicle Types: How will capacity be allocated for different parking spot types — e.g. motorcycles, compact cars, electric cars, handicap vehicles, etc.?
- Pricing: How will pricing be handled? It should accommodate having different rates for each hour. For example, customers have to pay $4 for the first hour, $3.5 for the second and third hours, and $2.5 for all the remaining hours.
#### Design an Online Stock Brokerage System
- Watchlists: How would the system handle watchlists created by the user to save/monitor specific stocks?
- Transaction Types: How would the system handle different transaction types, e.g. stop loss and stop limit order? What types would be supported?
- Stock Lots: How will the system differentiate between different ‘lots’ of the same stock for reporting purposes if a user has bought the same stock multiple times?
- Reporting: How will the system generate reports for monthly, quarterly, and annual updates?
#### Design a Car Rental System
- Identification: How will each vehicle be uniquely identified and located within the parking garage?
- Fees: How would the system collect a late fee for late returns?
- Logs: How would the system maintain a log for each vehicle and for each member?
- Customization: How would the system handle members’ requests for additional services like roadside assistance, full insurance, and GPS?
#### Design Facebook — a social network
- Discoverability: How are users able to search other users’ profiles?
- Following: How are users able to follow/unfollow other users without becoming a direct connection?
- Groups / Pages: How are members able to create both groups and pages in addition to their own user profiles?
- Privacy: How will the system handle privacy lists with certain content to be displayed only to specified connections?
- Alerts: How will users be notified for pre-selected events?

### Thoughts / Ideas 
- Design Recommendation Engine for Amazon 
- Design Pricing Engine for Amazon  

### References 
- https://hackernoon.com/how-not-to-design-netflix-in-your-45-minute-system-design-interview-64953391a054
- https://hackernoon.com/anatomy-of-a-system-design-interview-4cb57d75a53f
- https://hackernoon.com/14-patterns-to-ace-any-coding-interview-question-c5bb3357f6ed