### The Event-Driven Architecture - Right here. Right now
- Simple Events
    - Every second of every day you are talking about IT strategy, designing blueprints for the future, and selling your architectural vision, 
    - real business events are occurring. 
    - In the hour or so it takes you to make a pitch to management for investment in a new integration platform, orders are coming in, customers are leaving, money is being spent. 
    - All these exist right now in your present architecture, however poorly designed and non-strategic you may think it is.

- These are Simple Events and are represented by a collection of attributes and one or more time-stamps to record for posterity when the event occurred and/or was captured. 
- A simple order-received event, in JSON, might be something as plain as:
```json
{
	customerid		: 71689,
	item			: "widget1",
	quantity		: 3,
	value			: 45.00,
	channel			: "web",
	time-ordered	: "7/12/2008 11:34:52 AM",
	time-captured	: "7/12/2008 11:35:01 AM"
}
```

- Complex Events
    - When we are alerted to multiple simple events occurring within some sort of relationship, we can deduce complex events - essentially new event information.
    - The example used on Wikipedia contains bells ringing, a man in a suit, a woman in a big white dress and rice being thrown - you can surmise that there’s a wedding taking place without being explicitly told so.
    - In fact if you further examined the attributes of these events and discovered the bells were of type church, 
        - the suit was of type tuxedo, you could additionally infer that this is a US wedding, 
        - whereas church + morning suit would imply UK wedding, 
        - stringed instruments + red and gold sari implies Indian wedding, 
        - white dress + white dashiki implies West African wedding, and so on.
    - The point is that businesses already have ready access to simple events, but if we combine them the results can be enormously powerful in two ways:
    - Firstly, we can track events that we know happen but which we cannot measure directly, 
        - so an order-received event plus a matching order-dispatch event could create an order-fulfilled event with attributes that tell us something about the whole order process 
        - (automatically separating out order-received events that were not completed, or order-dispatch events that don’t have matching order-received events for product replacements or perhaps a sign of fraudulent activity).
    - And, using basic inference rules, we can generate other events to suggest what might be happening.
- Composite Events
    - Composite events are a special subset of complex events. 
    - Once you have identified your primitive (simple) events you can combine them different ways to make Composite Events and use these to make great leaps in understanding what customers are up to. 
    - Let’s keep this straightforward and take a widget-purchase event, an account-closure event and a customer-complaint event.
    - There are many types of composite event, a few interesting examples for the Gristle and Flint management team might be:
        - The Sequence: Event A (customer bought a widget) completed before Event B (customer closed their account) OR Event C (customer complained)
        - The Conjunction: Event A (customer bought a widget of one type) happened in any relation to Event B (customer bought a widget of another type)
        - The Iteration: Event A (customer bought a widget) happened more than once
        - The Negation: Event A (customer bought a widget) did not happen between time T1 and time T2
        - Certain types of Sequence would alert us to the fact that there’s something not quite right with one of our widgets; 
            - Conjunctions identify both loyal customers and popular widgets combinations that might be ripe for promotion; 
            - Iterations tell us something about unit quantities and possibilities for multi-buy discounts; 
            - Negations can tell us about failing promotions.
    - Understanding Composite Events doesn’t just help us run our businesses better, we can feed back what we learn directly to customers. If you’ve ever looked at a product page on Amazon and seen the “customers who bought this also looked at..” or the “buy this and get this too..” messages, you have witnessed the results of composite event processing.
    - Composite events are a collection of simple events that map to some kind of pattern or regular expression.
- The Event Spaghetti
- For the purposes of an introduction to event-based thinking, knowing that there are 
    - simple events (stuff that happened) and 
    - complex events (stuff we can generate in various ways from stuff that happened) is enough to be going on with. 
    - I added the composite event examples because they’re a neat way to see direct business value and also easy to program inside a regular project, 
    - without needing to make a big deal about adopting EDA to a sceptical business sponsor.
- If you want to read more on the taxonomy of events Opher Etzion has a nice Venn diagram in his post 
- “On relationships among: derived event, composite event, complex event and situation” showing how complex and composite events relate to the class of derived events and topics like situation theory.

#### The Event-Driven Architecture

- No enterprise architecture is comprised of just one something. 
    - EDA, SBA, SOA, WOA, ROA are just styles, not one-size-fits-all, next-big-thing, this-is-better-than-that replacements for the previous fad. 
    - EDA is not SOA 2.0. Like most styles they can compliment each other quite happily. Thank you for listening.