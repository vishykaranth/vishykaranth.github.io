### Uber 
#### INitial 
- Python - celery
#### SOA 
- Taxi, Food and Cargo
- Meet Demand and Supply 
- Mobile Traffic - everythng works on GPS 
- 2 service 
- Supply Service 
- Demand Service 
- Dispatcher Optimizer
- Google S2 library 
    - tiny cells of 1 sq km, 
    - joining cell gives full map, 
    - each cell has unique id 
    - easily go to server 
    - we could do consistent hashing
- Match Rider to Driver 
    - Get Drivers in 2 km radius 
    - Calculate ETA or distance connected by Road (1.8 km) 
    - Fetch minimum distance or ETA 
    - Send message to Driver 
    - Match Driver and Rider
- System      
    - Supply - Cabs 
    - Demand - User Request 
    - Request comes via the Kafka REST API           
 
 ### Twitter Design 
 #### Requirements 
 - Tweets
 - Timeline
    - Home 
    - User 
    - Search  
 - Trends 
 #### Characteristics 
 - 300 million user
 - 600 tweets write/sec
 - 600,000 tweets reads/sec
 - Read Heavy 
 - Eventually Consistent 
 - Storage 
    - 240 char/tweet
    - How we scale storage   
#### Architecture 
- Redis and DB 
    - Redis is faster all data is stored in-memory 
    - copy of all data in database 
- Schema 
    - User table 
    - Tweets table 
    - Followers table 
- Relationships 
    - (User) 1--->Many (Tweets)
    - (User) Many--->Many (Followers)           