---
layout: page
title: Designing a Paste Bin  
permalink: /designing-paste-bin/
---

### Background 
- Pastebin like services enable users to store plain text or images over the network (typically the Internet) 
    - and generate unique URLs to access the uploaded data. 
    - Such services are also used to share data over the network quickly, 
    - as users would just need to pass the URL to let other users see it.

### Functional Requirements:
- Users should be able to upload or “paste” their data and get a unique URL to access it.
- Users will only be able to upload text.
- Data and links will expire after a specific timespan automatically; users should also be able to specify expiration time.
- Users should optionally be able to pick a custom alias for their paste.
- When clicked on URL, it should redirect to pasted content


### Non-Functional Requirements:
- The system should be highly reliable, any data uploaded should not be lost.
- The system should be highly available. This is required because if our service is down, users will not be able to access their Pastes.
- Users should be able to access their Pastes in real-time with minimum latency.
- Paste links should not be guessable (not predictable)
- Others 
    - User should not exhaust the api (should limit the url creation)
    - Exposing the service through REST APIs
    - Analytics: Recording different parameters
    - Able to create private/ public shorten url

### Brainstorming 
- What should be the limit on the amount of text user can paste at a time? 
    - We can limit users not to have Pastes bigger than 10MB to stop the abuse of the service.
- Should we impose size limits on custom URLs? 
    - Since our service supports custom URLs, 
    - users can pick any URL that they like, 
    - but providing a custom URL is not mandatory. 
    - However, it is reasonable (and often desirable) to impose a size limit on custom URLs, 
    - so that we have a consistent URL database.

### Traffic estimates: 
- Pastebin services are not expected to have traffic similar to Twitter or Facebook, 
- let’s assume here that we get one million new pastes added to our system every day. 
- This leaves us with five million reads per day.
- New Pastes per second:
    - 1M / (24 hours * 3600 seconds) ~= 12 pastes/sec
- Paste reads per second:
    - 5M / (24 hours * 3600 seconds) ~= 58 reads/sec
    
### Storage estimates: 
- Users can upload maximum 10MB of data; 
- commonly Pastebin like services are used to share source code, configs or logs. 
- Such texts are not huge, so let’s assume that each paste on average contains 10KB.
    - At this rate, we will be storing 10GB of data per day.
    - 1M * 10KB => 10 GB/day
    - If we want to store this data for ten years, we would need the total **storage capacity of 36TB**.
- With 1M pastes every day we will have 3.6 billion Pastes in 10 years. 
    - We need to generate and store keys to uniquely identify these pastes. 
    - If we use base64 encoding ([A-Z, a-z, 0-9, ., -]) we would need six letters strings:
        - 64^6 ~= 68.7 billion unique strings
- If it takes one byte to store one character, total size required to store 3.6B keys would be:
    - 3.6B * 6 => 22 GB
- 22GB is negligible compared to 36TB. 
    - To keep some margin, we will assume a 70% capacity model 
    - (meaning we don’t want to use more than 70% of our total storage capacity at any point), 
    - which raises our storage needs to 51.4TB.
    
### Bandwidth estimates: 
- For write requests, we expect 12 new pastes per second, resulting in 120KB of ingress per second.
    - 12 * 10KB => 120 KB/s
- As for the read request, we expect 58 requests per second. Therefore, total data egress (sent to users) will be 0.6 MB/s.
    - 58 * 10KB => 0.6 MB/s
- Although total ingress and egress are not big, we should keep these numbers in mind while designing our service.

### Memory estimates: 
- We can cache some of the hot pastes that are frequently accessed. Following the 80-20 rule, meaning 20% of hot pastes generate 80% of traffic, we would like to cache these 20% pastes
- Since we have 5M read requests per day, to cache 20% of these requests, we would need:
- 0.2 * 5M * 10KB ~= 10 GB 

### System APIs
- We can have SOAP or REST APIs to expose the functionality of our service. 
- Following could be the definitions of the APIs to create/retrieve/delete Pastes:
    - addPaste(api_dev_key, paste_data, custom_url=None user_name=None, paste_name=None, expire_date=None)
        - Returns: (string)
        - A successful insertion returns the URL through which the paste can be accessed, otherwise, returns an error code.
    - getPaste(api_dev_key, api_paste_key)
    - deletePaste(api_dev_key, api_paste_key)
- Database Schema:
    - We would need two tables, one for storing information about the Pastes and the other for users’ data.
