# Pastebin System Design  
  
With Pastebin you can paste the text data in one place and share it with other users across the world with a unique URL. 

## Requirements 

### Non Functional Requirements 

- User should be able to share data - text only with a unique URL.
- The link should have capabilities like read once only (burn after read) or expiry after first read to expiry after the time specified.
- Support syntax highlighting. 

### Functional Requirements 

- The uploaded data should not be lost. 
- Links should be unique and random. 
- Highly Available. 

### Future Scope 

- Analytics 

## Capacity Estimations 

- The above system is being designed keeping only text data in mind with size of 1MB. In case of huge amounts of data, Object Stores on Cloud like S3 can be considered. 
- The System will be a read-heavy system. Assuming 10 reads for 1 write. 10 : 1 Ratio. 
- Assumptions: Expected Monthly Active users - 10M and Max Paste Size - 1MB


### Traffic Figures Estimation
Traffic Figures Estimation - with Read : Write Ratio of 10 : 1  
Expected MAU = 10M  
Expected Requests = 100M  
Total Read Requests a month = 100M Reads/Month  
Total ReadRequests per sec = 100M/(30 * 24 * 2600) = 38 read-requests/second  
Similarly for writes = 3.8 write-requests/second  

### Storage Figures Estimation
Expecting at-least 1 paste by every user
    Total Paste = 10M Users X 1MB = 10TB/month
    Around ~ 120 TB/year
    
Cache Estimation = 20% of read-traffic = 2TB/month
    
## APIs 

POST /v1/paste-bin/
Adds paste to the paste-bin

GET /v1/paste-bin/{paste-id}
Gets the paste from the paste-bin based on the paste-id sent

GET /v1/paste-bin/recent-items
Gets the recent public pastes

## Database Design 

User Details Table
```
id
userId - PK
userName
password
createdAt
updatedAt
socialAuthDetails
```

User Pastes Table 
```
id
pasteId
pasteData
syntaxHighlight
expiryDetails 
accessTimes
pasteName
pasteExposure 
size
url
userId
isActive
createdAt
updatedAt
```
    
syntaxHighlight - Highlight can be C, C++, Java etc. which will be handled on the front end side mostly. An enum can be stored here.  
expiryDetails - This can be an Enum too. There can be cases like burn after read, after 1 min to several days.  
pasteExposure - Public or Private Pastes.
url - This can be the unique short URL generated from KeyGen Service. 

As the paste data is stored in the database in the current design, a histroy table can also be used to know what were the items in the last save made. This table can be treated as something simialar to access logs. This will be insert only table for reference only. 

User Paste History
```
id
pasteId
userId
pasteData
syntaxHighlight
createdAt
size
```

## High Level Design

DB for User Details can be any RDBMS (MySQL)
DB for User Paste Details can be any NoSQL (MongoDB)
If we want to store the object data, cloud object stores like S3 can also be used. 
KeyGen Server - A seperate entity for creation of unique keys (6-8 character long) for providing unique keys as and when required. 
Async Cleanup Service - Service to mark paste details isActive as `false` when the expiryDetails conditions are met.
(This can also be done with a Scheduler Service for deletion when the time is specific like - 1 day, etc.).

Flow when a paste is created and accessed 
1. Requests hits the application 
2. Generate a unique key from KeyGen Service. 
3. Store the data in the DB/ Cloud Object Store and return the unique link. 

Flow when a paste has to be accessed
1. Request hits the application 
2. Find the unique URL from the userPasteDetails table 
3. Update the accessTimes and return it back. If the expiryDetails has burn after read - set isActive as false and return the paste details with meta data - syntax highlighting, etc. 
4. If the data from DB already has inActive as false, then send nothing found.

The Async Cleanup can be run everyday or the Scheduler can be made to run at a specific point of time. 

### Caching 
Caching - 20% of Read Traffic. Redis can be used here. 
Eviction Policy - LRU 

### Load Balancing
LB Between Client and Server or Server and DBs or both.  
Approaches - Round Robin, Least Connections, Network Hashes.   


   