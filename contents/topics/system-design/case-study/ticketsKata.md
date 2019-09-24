---
layout: page
title: Tickets Kata 
permalink: /tickets-kata/
---


# Tickets Reservation System Kata

### Description

Your startup company is creating big online tickets reservation system for many events (concerts, sports). Your role is to design subsystem responsible for online reservation.
<br><br>
You know that there will be interactive map for each event, and users will be able to reserve and buy tickets by selecting places on the map and then providing their personal details.
<br>
You must create subsystem responsible for online tickets reservation.

### Functional requirements

- Subsystem has tickets, which can be in 3 states FREE -> RESERVED -> SOLD
- Tickets can be stored for many events. Events are defined and maintained in another subsystem.
- Each event has its own sales start and end date. Tickets can be only reserved and sold during that periond of time.
- Each Ticket has its own category, sector, row, and seat
- Category of ticket has its own price.
- Category of ticket can have restriction how many tickets for category can be reserved by one user (For example 4 places for VIP category only)
- Tickets can be assigned to user based by userId, which should income to endpoint.
- Tickets are can be reserved only in reservation (basket) which is assigned to user and event.
- Every event has its own ticket category, which comes from another subsystem
- User can only reserve max 8 tickets per event
- Reservation for event per user expires after 10 minutes
- Each reserved ticket msut be assigned to one person with given name, surname, date of birth
- After tickets reservation we return total price of reserved tickets
- System should check whether all tickets are bought or not. If yes, then system should emit event
(with meaningful name and eventId) that tickets reservation for event is already closed.

# System API's

- Create endpoint for pool of tickets creation. After another subsystem create event for some show, we should recieve JSON with information about tickets pool. We should create ticket categories, then create tickets for event.

```json
{
    "event": {
        "id": "dasasds-12387213-asdasd",
        "salesStartDate": "1231232132132",
        "salesEndDate": "1231232132132"
        },
    "categories":[
        {
            "code":"ST",
            "name":"STANDARD",
            "price":{
                "value":144.99,
                "currency":"USD"
            }
        },
        {
            "code":"GD",
            "name":"GOLDEN",
            "price":{
                "value":399.99,
                "currency":"USD"
            },
            "restriction": {
                "maximumTicketsPerReservation": 3
            }
        }
    ],
    "tickets":[
        {
            "sector":"A",
            "rows":10,
            "seatsInRow":12,
            "categoryCode":"ST"
        },
        {
            "sector":"B",
            "rows":10,
            "seatsInRow":12,
            "categoryCode":"ST"
        },
        {
            "sector":"C",
            "rows":10,
            "seatsInRow":12,
            "categoryCode":"GD"
        }
    ]
}
```

- Create endpoint for fetching info about all tickets for event. There should be distinction for free tickets and occupied tickets.
Additionaly ticket should have info about price, category, section, row and seat. This info will be used on UI interactive map.

- Create endpoint for ticket reservation creation. Reservation took place when user go into reservation mode. If user has no reservation,
then reservation is created and be valid for 10 minutes. If user already has reservation, then we should inform user about it.

```json
{
    "userId":"asd2sa21fbhf234t5",
    "eventId":"1234sdfjher3adt"
}
```

- Create endpoint for add ticket to reservation (update reservation). 
Example JSON:

```json
{
    "ticketId": "1231233-12323-132213",
    "dateTime": "12312341142",
    "person": {
      "firstName": "Adam",
      "lastName": "Robinson",
      "dateOfBirth": "14/06/1976"
    }
}
```

- Create endpoint for remove ticket from reservation (update reservation).

- Create endpoint for finalize reservation. This endpoint will be triggered from another subsystem responsible for payments just after transaction finalization. So the only responsibility here is to turn reservation tickets into SOLD state. 
After that, reservation can not be updated anymore.

- Create endpoint for reservation info - actual tickets list and total price. 

### Technical requirements

- Domain Driven Design.
- Clean code and clean maintanable architecture.
- Code created with TDD approach with unit tests, integration tests and acceptance tests as well.
- Application should be runnable without any external dependencies (like installing DB on machine to run app)
- Application should be created as web-app.
- Concurrency should be handled.
- Business logic must be fully covered.
- Expired reservations should be removed as soon as possible. Interactive map must show real tickets state.
- Subsystem must expose REST API, which will be used by another subsystems and by front end apps.
- Exceptions and failures (i.e. reservation expired) should be handled and returned from API with meaningful description and HTTP status code.