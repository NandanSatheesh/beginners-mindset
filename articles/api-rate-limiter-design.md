

# API Rate Limiter Design

Allow a client to send a fixed number of requests in a given time window. If the number of requests in the time window exceeds, then deny the current request.

## Requirements

### Non Functional Requirements

- Highly Available. 
- The system should take minimum time to process and should not increase the wait or loading time for the user.  

### Functional Requirements
- Should be configurable for plug and play for any client to use.
- Should be configurable to accept the number of requests and the time window for the same. 

  

## Algorithms 
- Fixed Window - 0-100ms, 100-200ms, etc. 
- Sliding Window - Checks the window from the current time and checks if the request can be processed.   


## Low-Level Design - In Memory Solution

  A basic interface RateLimiter would be created with a function that will accept `clientId`

```java
public interface RateLimiter {  
  
    boolean canAllow(String clientId);  
}
```

Now this interface can be extended to provide the capabilities required. It can use Fixed Window Algorithm or Sliding Window Algorithm to evict previous entries.

A concrete implementation for the same with Sliding Window would look like this. 
```java
public class BasicSlidingWindowRateLimiter implements RateLimiter {  
 private ConcurrentHashMap<String, Queue<Long>> map;  
 private final int TIME_WINDOW = 60000; // 1 minute  
 private final int REQUEST_LIMIT = 5;  
  
 public BasicSlidingWindowRateLimiter() {  
   this.map = new ConcurrentHashMap<>();  
 }  
  
 @Override  
 public boolean canAllow(String clientId) {  
   long currentTimeInMillis = System.currentTimeMillis();  
   Queue<Long> queue = map.getOrDefault(clientId, new ConcurrentLinkedDeque<>());  
   if (queue.isEmpty()) {  
     queue.add(currentTimeInMillis);  
     map.put(clientId, queue);  
     map.put(clientId, queue);  
     return true;
   }  
  
   while (!queue.isEmpty() && queue.peek() < currentTimeInMillis - TIME_WINDOW) {  
     queue.poll();  
  }  
  
  if (queue.size() > REQUEST_LIMIT) {  
    return false;  
  } else {  
    queue.add(currentTimeInMillis);  
    map.put(clientId, queue);  
    return true;  
  }  
 }
}
```

### Memory Estimation 
Queue Memory Approximate Estimation - 

| Variable | Space Assumed  |
| --- | ----------- |
| size | 4 bytes |
| Node first | 4 bytes |
| Node last | 4 bytes |
| Total | 12 bytes |

HashMap Memory Approximate Estimation for 1 ClientId - 

| Variable | Space Assumed  |
| --- | ----------- |
| key (String Reference | 4 bytes |
| Queue Reference - Assuming 10 Entries at peak for 1 Bucket | 4 bytes |
| hashCode for Map | 4 bytes |
| next - Map Property | 4 bytes 
| hashCode for Map | 4 bytes |
| loadFactor, initalCapcity and size (Map Properties) | 8 bytes |
| header (Map Property) | 8 bytes |
| Total | 32 bytes |

Assuming the scale for about 100K active clients - 
We'll end up with - 
(Key Reference 4 bytes + Queue Reference 4 bytes + Header for each record 8 bytes + Queue Size of 10 for each Record of approximately 8 bytes) * 100K = 96,00,000 bytes = 9.6MB



## Scaling 
The above in-memory solution looks great on paper but when it comes to real-world scenarios for example - 100k Requests per Minute with varing traffic and request conditions, it can go out of memory as well. 

One of the solves for that would be to keep the `HashMap<String, Queue<Long>>` structure in a Redis. 

### Caching and Sharding 
  The above solution can also be cached and shard based on `clientId`. Consistent Hashing can be used here. Cache Eviction Policy can be LRU. 

