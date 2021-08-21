
# Decoding Bounce App's Self Help Screens

-  [ Decoding Self Help Screens in Bounce](#decoding-self-help-screens-in-bounce)
	- [Requirements ](#requirements)
	- [Screens - UI](#screens---ui)
	- [Capacity Estimations](#capacity-estimations)
	- [APIs](#apis)
	- [Database Design](#database-design)
	- [HLD](#hld)

  Note that this is an attempt to understand the architecture of the backend for the Self Help feature. 
  
## Requirements 

### Non Functional Requirements 

- At the time of development of this feature, Bounce was clocking more than 1.5M rides daily. The number of people who were facing issues with bike were also in a high number. 

- The Customer Care was unable to solve all the queries as the number of calls were huge in volume. Some had to wait for 10 min to talk to a executive. 

- On deep dive to the problem, it was found that the number of people who had issues during the ride were more than the number of people who had not started the ride but still had issues. (Ride Booking Flow vs In-Trip Flow).

- Solution was to have help options on the screen so that some of the users can unblock themselves on their own without having to call the Customer Care. 
- 
- This also meant to keep separate action items for both the ride states. 

### Functional Requirements 
 
- The Help Screen content should be configurable for both In-Trip States and Ride-Booking Flow and General App as well. 

- The templates must be configured at the time of development. The content should be added on the fly without having an app release. 

- Different Call Actions for In-Trip and Ride-Booking to be critical as the call volumnes in In-Trip are bound to be a lot higher than any other. 

- The action items like Call Us or Email Us can require an app release. But a generic CTA should be there to handle most of the scenarios on app. 

- Content helpful or not helpful to be tracked from CleverTap for improving the content or scraping the content for something new.  

- Should be built like a platform which can be used across the app for any new intiatives as well like Bounce Club, Bouce Pool, etc. 

### Future Scope 

- Integration of Service Desk Management Tools for Tracking the Issues.

## Screens - UI 

![Screen 1](https://raw.githubusercontent.com/NandanSatheesh/beginners-mindset/master/images/bounce-self-help/bounce-self-help-image-1.jpeg)

![Screen 1](https://raw.githubusercontent.com/NandanSatheesh/beginners-mindset/master/images/bounce-self-help/bounce-self-help-image-2.jpeg)

## Capacity Estimations 

### Traffic Estimation 

This system is going to be read heavy with 99.99% of time as reads and 0.01% times as write. 
Writes are mostly addition of new help content and actions .

Expecpted Daily Active Users - 10M
Considering an average of 6 requests for viewing 2 help screen items. Expected Daily Requests - 60M
Total API Requests per second = 60000000/86400 = ~700 requests/second

### Storage Estimation 

Cache Estimation can be - 50% of the read traffic as the data is not bound to change much. 
Cache Invalidation must also be done when a new item gets added so that new items are served to the users also. 

## APIs

CRUD APIs for Addition of Help Content 

Help Section APIs

GET `v1/self-help/topics` 

-  This would get the user details from the token. From, that find out what state the user is in - `IN-TRIP` / `BOOKING-CONFIRMED`. 

- Find the active sectionId from Main Section Items.

- Find all the active help topics to show based on the sectionId and the tripState. 

GET `v1/self-help/topics/content?contentId={contentId}`

- This API would serve the data for the given topic from the previous API.  

## Database Design 

Action (CTA) Details Table - Actions can included Call Us / Email Us with specific value. 
```
id - PK
actionType
actionData
metaData
createdAt
updatedAt
isActive
```

Content Data Table 
```
id - PK
title 
heading
description
actionId - FK of Action Details Table
isActive
createdAt
updatedAt
```


Main Section Items - The reason for keeping this would be to show if its a Grid View of `m` X `n` or just a horizonatal list view of items. 
```
id 
sectionType
sectionDescription
orientation
tripState - IN-TRIP/BOOKING-CONFIRMED
createdAt
updatedAt
isActive
```


Help Topics - Help Topics 
```
id 
title 
sectionId - FK of Main Section Items
contentId - FK of Content Data Table
topicLogo
tripState - IN-TRIP/BOOKING-CONFIRMED
createdAt
updatedAt
isActive 
```

## HLD

This is a basic application which serves dynamic data. The current appraoch uses MySQL for mapping items. This can be solved with 1-2 documents with MongoDB as well. 

### Caching

As this application is going to have lot of read traffic and the data doesn't change much. 
Around 50% of Read Traffic can be cached. 
Redis can be used here. 
Eviction Policy - LRU

### Load Balancing
LB Between Client and Server or Server and DBs or both.  
Approaches - Round Robin, Least Connections, Network Hashes.
