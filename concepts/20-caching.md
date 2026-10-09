# Caching

Caching is the engineering decision to reuse previous work instead of repeating it, while making sure the reused result is still appropriate for the current request.

## 1. The Problem

Imagine we are building an application. It stores data in a database, and whenever a user requests something, the backend retrieves the data and returns it.

For example, imagine a system has a database containing users, saved analyses, and reports. When a user requests a report, the backend retrieves it from the database.

Initially, this works well. But applications grow.

Imagine a popular report is requested repeatedly:

```
Request 1 → Database query
Request 2 → Database query
Request 3 → Database query
Request 4 → Database query
Request 5 → Database query
```

Every request retrieves the same report. Now imagine thousands of users requesting the same information. The database must repeatedly perform the same work, which causes several problems:
 
- The database processes unnecessary queries.
- Users may experience increased response times.
- Database CPU, memory, and connections are consumed.
- The database has less capacity available for other operations.
- As traffic increases, the database may become a bottleneck.
The problem is not necessarily that the database is poorly designed. The problem is that **we keep retrieving the same data when we could reuse a result we already obtained.**
 
> Can we avoid repeatedly querying the database for information that has already been requested?
>
> Yes. We introduce a cache.

### The Question
 
We already solved *"How do we store and retrieve application data reliably?"* with **databases**.
 
Now we need to solve *"How do we avoid repeatedly retrieving the same data when it is requested frequently?"* with **caching**.
 
## 2. What Is Caching?
 
Caching is the practice of **storing a copy of data in a location where it can be accessed more quickly when needed again.**
 
Instead of retrieving data from the original source every time, the application checks whether a cached copy already exists.
 
**Without caching**, every request reaches the database:

```
User Request
     │
     ▼
  Backend
     │
     ▼
  Database
     │
     ▼
  Return Data
```
 
**With caching**, the application checks the cache first:

```
User Request
     │
     ▼
  Backend
     │
     ▼
    Cache
     │
     ├── Data Found ──────► Return Cached Data
     │
     └── Data Not Found
              │
              ▼
           Database
              │
              ▼
          Store in Cache
              │
              ▼
          Return Data
```

If the data is available, the cached copy is returned. If it is missing, the application retrieves it from the database, stores a copy in the cache, and returns the result.
 
> We pay the cost of retrieving the data once, then reuse the result for subsequent requests whenever the cached copy remains usable.
 
The database remains the **source of truth**. The cache simply makes frequently accessed data faster to retrieve.
 
## 3. How Does Caching Work?
 
Imagine our application stores user profiles in PostgreSQL, and a user requests the profile of user 42.

### First request
 
```
Request: Get user 42
        │
        ▼
   Check Cache
        │
        ▼
     Cache Miss
        │
        ▼
  Query Database
        │
        ▼
  Retrieve Profile
        │
        ├──────────────► Return Profile
        │
        ▼
   Store in Cache
```
 
The cache doesn't contain the profile yet, so the application queries the database and then stores a copy in the cache:

```json
Key: user:42
 
Value:
{
    "name": "Alex",
    "city": "Harare"
}
```
 
The **key** identifies the cached data, and the **value** contains the data itself.
 
### Second request
 
```
Request: Get user 42
        │
        ▼
   Check Cache
        │
        ▼
     Cache Hit
        │
        ▼
 Return Cached Profile
```
This time the profile is found in the cache and PostgreSQL is never queried. This is the fundamental benefit of caching: **the first request populates the cache, and subsequent requests reuse the stored result.**
 
## 4. Cache Hits and Cache Misses
 
We've just encountered two important concepts.
 
### Cache hit
 
A cache hit occurs when the requested data is available in the cache and can be used.
 
```
Request → Cache → Data Found → Return Result
```
 
The application avoids retrieving the data from the original source.
 
### Cache miss
 
A cache miss occurs when the requested data is not in the cache, or cannot be used.

```
Request → Cache → Data Not Found → Database Query → Store Result in Cache → Return Result
```
 
A cache miss is not necessarily a failure. It is a normal part of caching, especially when data is requested for the first time.
 
### Why does this matter?
 
Imagine an application receives 1,000 requests. 900 are served from the cache, and 100 require database queries.
 
```
Cache Hit Rate = Cache Hits / Total Requests
               = 900 / 1000
               = 90%
```

A 90% hit rate can significantly reduce the number of database queries. However, a high hit rate does not automatically guarantee good performance. We must also consider cache lookup time, the cost of misses, and whether cached results are sufficiently fresh.
 
## 5. Why Not Store Everything in the Cache?
 
If retrieving data from a cache is faster, why not put all our application data there?
 
Because caching introduces new problems. Each one of them drives a later section of this guide.
 
### Problem 1: Memory is limited
 
Caches commonly use memory for fast access, and memory is finite. If the database holds 500 GB and the cache has 10 GB available, we cannot store everything. We need to decide which data is worth keeping. Frequently requested data is usually a better candidate than rarely accessed data.
 
*This leads to [eviction policies](#10-cache-eviction-policies).*

### Problem 2: Data can become outdated
 
Imagine a product costs $800, and the application caches that value. Later, the seller changes the price to $750.
 
```
Database: Product Price = $750
Cache:    Product Price = $800
```
 
The application may return the old price, because the cached copy no longer reflects the database. We need a way to decide when cached data should expire or be updated.
 
*This leads to [TTL](#7-ttl-time-to-live), [cache invalidation](#8-cache-invalidation), and [write strategies](#9-cache-writing-strategies).*

### Problem 3: The cache can fail
 
A cache is another component in our system. It can become unavailable, run out of memory, or lose its contents. If the application depends on the cache for every request, a cache failure can affect the entire application. We need to decide what happens when the cache is unavailable. For example, the application might fall back to the database, if the database can safely handle the additional load.
 
### Problem 4: The cache can become ineffective
 
Imagine every request asks for completely different data:
 
```
Request 1 → Product A
Request 2 → Product B
Request 3 → Product C
Request 4 → Product D
```

If the same data is rarely requested again, the cache provides little benefit, and the application spends resources storing data that is never reused. Caching is most useful when the workload contains data that is requested repeatedly.
 
## 6. Where Does the Cache Live?
 
So far we've treated the cache as a single component, but caches can exist in different places depending on the problem being solved.
 
### 6.1 Application memory
 
An application can store data in its own process memory. For example, a Python application could use a dictionary:

```python
cache = {}
 
cache["user:42"] = {
    "name": "Alex",
    "city": "Harare"
}
```
 
When the application needs the profile, it checks the dictionary before querying the database. This is simple and fast.
 
However, each application instance has its own memory. With three backend servers:
 
```
Backend 1 → Cache A
Backend 2 → Cache B
Backend 3 → Cache C
```

If Backend 1 caches a profile, Backends 2 and 3 do not automatically receive it, so each instance may have a different view of the cached data. The cached data also normally disappears when the process restarts.
 
For small applications this is perfectly acceptable. For larger systems with multiple instances, we may need a shared cache.
 
### 6.2 Distributed cache
 
Instead of giving every instance its own cache, we can use a separate caching service shared by all of them, for example **Redis**.
 
```
Backend 1 ──┐
Backend 2 ──┼──► Redis
Backend 3 ──┘
```
If Backend 1 stores a profile, Backend 2 can retrieve it from Redis. This makes it easier to reuse cached data across instances. However, the cache now requires network communication and introduces another service that must be managed. Redis is not the only option, but it is a common technology for distributed caching.
 
### 6.3 Browser cache
 
Caching can also happen on the user's device. When a user visits a website, their browser may store images, stylesheets, and JavaScript files.
 
```
First visit:   Browser ─────► Web Server ─────► Download Files ─────► Browser Cache
 
Second visit:  Browser ─────► Check Local Cache ─────► Reuse Valid Files
```

This reduces unnecessary network requests and can make repeat visits faster. The browser must still respect the relevant caching rules to decide whether a stored file can be reused.
 
### 6.4 Content Delivery Network (CDN)
 
A CDN caches eligible content at servers distributed across different geographical locations. Imagine a website hosted in the United States, accessed from Zimbabwe. Without an edge cache, the request may need to travel to the origin server. A CDN stores copies of eligible content closer to the user.
 
```
               Origin Server
                     │
             ┌───────┴───────┐
             ▼               ▼
        CDN Edge A       CDN Edge B
        Zimbabwe         Europe
             │               │
             ▼               ▼
         Local Users     Local Users
```

When cached content is available at an appropriate edge location, the user can retrieve it without every request reaching the origin. CDNs are particularly useful for static assets, images, and video delivery.
 
Application-memory caching, distributed caching, browser caching, and CDN caching solve related problems at different layers of a system. The rest of this guide focuses mainly on application-level caches (memory and distributed), where freshness and write handling matter most.
 
## 7. TTL: Time to Live
 
We know cached data can become outdated. One way to manage this is **TTL (Time to Live)**, which specifies how long a cached entry may remain valid under the cache's expiration policy.
 
```
Key: product:101
 
Value:
{
    "name": "Laptop",
    "price": 800
}
 
TTL: 300 seconds
```

This entry has a TTL of 300 seconds (five minutes). After it expires, a subsequent request can retrieve the product from the database and cache the updated result.
 
```
Store Data in Cache
        │
        ▼
   Start TTL
        │
        ▼
   Five Minutes
        │
        ▼
   Entry Expires
        │
        ▼
 Next Request May Query Database
```

TTL limits how long an entry remains usable without being refreshed. But it introduces a trade-off:
 
| TTL | Benefit | Cost |
| --- | ------- | ---- |
| **Long** | Fewer database queries | Outdated data may remain available for longer |
| **Short** | Better freshness | More frequent cache misses |
 
The appropriate TTL depends on how frequently the data changes and how much staleness the application can tolerate.
 
> **Important:** TTL and cache invalidation are related, but not the same. TTL handles expiration **based on time**. Invalidation removes or marks an entry unusable because the application **knows** the data has changed or should no longer be used.
>## 8. Cache Invalidation
 
TTL is a passive solution: it waits for time to pass. But what if we know the data has changed *right now*?
 
Imagine a user updates their profile. The database is updated successfully, but the cache still contains the old profile. We could wait for the TTL to expire, but we may want the new profile to be available immediately.
 
**Cache invalidation** is the process of removing or marking cached data as unusable when it is no longer appropriate to serve.



```
User Updates Profile
        │
        ▼
Update Database
        │
        ▼
Invalidate Cached Profile
        │
        ▼
Next Request
        │
        ▼
Cache Miss
        │
        ▼
Retrieve Updated Profile
        │
        ▼
Store New Cached Copy
```

The next request retrieves the updated profile and repopulates the cache. This avoids intentionally serving the old entry once the update and invalidation have succeeded.
 
However, real systems must consider failures and concurrency:
 
- What happens if the database update succeeds, but invalidating the cache fails?
- What happens if an old database query finishes *after* another request has already cached a newer value?
These are important consistency problems in distributed systems. Cache invalidation can be more complicated than caching the data in the first place.
 
Notice what we just did: we updated the database and then dealt with the cache as a separate step. That is one *choice* about how writes interact with the cache. There are others, and which one we pick changes how fresh, fast, and safe our data is.
 
## 9. Cache Writing Strategies

### The problem
 
Until now we've focused on **reading** cached data. But what happens when data is **written** or updated?
 
Imagine a user changes a product's price from $800 to $750. We now have two copies of the data:
 
```
Cache                 Database
 
Price: $800           Price: $750
   OLD                    NEW
```
 
The database has been updated, but the cache still holds the old value. If another user requests the product, the application might return $800. This is the staleness problem from section 5, now seen from the write side.
 
> When data changes, how do we keep the cache and database sufficiently consistent?
 
TTL waits for time to pass, and invalidation removes the old copy after a write. A **write strategy** goes one step further: it defines **how the write itself flows through the cache and the database**. Three common strategies are Write-Through, Write-Back (Write-Behind), and Write-Around.

### 9.1 Write-Through
 
Every write updates the **cache and the database** before the write is considered complete.
 
```
Application: update price to $750
        │
        ▼
      Cache     → Price: $750
        │
        ▼
     Database   → Price: $750
        │
        ▼
 Write acknowledged
```
 
Both copies are updated as part of the write operation, and once both succeed, the write is acknowledged.
 
**Why it's useful:** after the write completes, cache reads see the new value, assuming the system correctly coordinates the two writes. This directly addresses the stale price problem above.


**Trade-offs:**
 
- Every write involves both the cache and the database, which can increase write latency.
- Failures between the two writes must be handled carefully.
- Written data is cached even if it is never read again, potentially wasting memory (which connects back to [Problem 1](#problem-1-memory-is-limited)).
> **Mental model:** Update the cache and database together on every write.

### 9.2 Write-Back (Write-Behind)
 
What if we don't want every write to wait for the database?
 
With write-back, the application writes to the **cache first**. The cache acknowledges the write immediately, and the database is updated **later, asynchronously**.
 
```
Application: update price to $750
        │
        ▼
      Cache     → Price: $750
        │
        ▼
 Write acknowledged
        ┆
        ┆  (later, asynchronously)
        ▼
     Database   → eventually Price: $750
```
 
**Why it's useful:**

- Writes can be acknowledged more quickly.
- Multiple updates can sometimes be combined before being written to the database.
- The database may experience fewer individual write operations.
**Trade-offs:**
 
- The database can temporarily contain older data than the cache.
- More importantly, if the cache loses a modified value before it is persisted, **that update can be lost**. This ties back to [Problem 3](#problem-3-the-cache-can-fail). Reliable implementations need mechanisms such as durable logging, retries, and recovery.
> **Mental model:** Write to the cache now; persist to the database later.

### 9.3 Write-Around
 
What if we don't want every write to populate the cache?
 
Write-around sends writes **directly to the database**, bypassing the cache. The cache is populated later, only when the data is read and a cache miss occurs.
 
```
Application: update price to $750
        │
        ▼
     Database   → Price: $750      (cache not updated by this write)
 
Later, a user requests the product:
   Cache miss → read database → populate cache
```
 
**Why it's useful:** some data is written frequently but rarely read again. If every write populated the cache, it could fill up with data that provides little benefit. Write-around avoids that, which helps with [Problem 4](#problem-4-the-cache-can-become-ineffective).
 
**Trade-offs:**

- The first read after a write may be a cache miss.
- Depending on the invalidation strategy, an old cached value could remain available unless the affected entry is invalidated or otherwise made unusable. This is exactly why write-around is normally paired with [invalidation](#8-cache-invalidation) or a [TTL](#7-ttl-time-to-live).
> **Mental model:** Write to the database; cache the data when it is read.
 
### 9.4 Comparing the three strategies
 
| Characteristic | Write-Through | Write-Back | Write-Around |
| -------------- | ------------- | ---------- | ------------ |
| **Write destination** | Cache and database | Cache first, database later | Database directly |
| **Database updated** | During the write operation | Asynchronously, later | During the write operation |
| **Write latency** | Usually higher than cache-only acknowledgement | Often lower | No cache write required |
| **Risk of losing acknowledged updates on cache failure** | Depends on write coordination | Higher if updates aren't durably recorded | Database has the update once its write succeeds |
| **Cache populated by writes?** | Yes | Yes | No |
| **Typical advantage** | Cache stays current after successful coordinated writes | Fast writes and potential batching | Avoids caching data that may not be reused |



These strategies describe different ways to coordinate cache and database writes. Real implementations may add invalidation, retries, transactional mechanisms, and other safeguards.
 
### 9.5 How do you choose?
 
Think about the workload:
 
- **Write-through:** when keeping cached data current after successful writes is important and the extra write latency is acceptable.
- **Write-back:** when write performance is a priority and you can safely manage delayed persistence and recovery.
- **Write-around:** when many writes are unlikely to be followed by reads, so caching every written value would waste space.
For example, an application caching frequently accessed product details might use write-through or update-based invalidation. A workload with high-volume writes might consider write-back if its durability requirements can be met. A system ingesting large amounts of data that is rarely queried immediately might benefit from write-around.
 
There is no universally best strategy. The right choice depends on freshness, durability, latency, and access patterns.

### 9.6 How this connects to everything else
 
| Concept | Role |
| ------- | ---- |
| **TTL** | Eventually expires entries, a safety net whatever write strategy you choose |
| **Invalidation** | Removes stale entries after a write (commonly paired with write-around) |
| **Write strategy** | Decides how a write flows through the cache and the database |
| **Eviction** | Decides what to remove when the cache is full, which write-through and write-back make more likely because writes also fill the cache |
 
Write strategies, TTL, and invalidation work together. They are layers of defense against stale data, not competing alternatives.
 
## 10. Cache Eviction Policies
 
TTL handles expiration based on time, and write strategies handle changes. But what happens when the cache **becomes full** before entries expire? Write-through and write-back make this more likely, since every write adds to the cache.
 
Suppose our cache can store only three items:
 
```
Cache: A, B, C
```

Now the application wants to store item D. The cache needs to make room. **Which item should it remove?** This is solved by an *eviction policy*.
 
### Least Recently Used (LRU)
 
LRU removes the item that has not been accessed for the longest time. Imagine the access sequence is:
 
```
A → B → C → A → D
```
 
Before D arrives, A was accessed most recently, while B and C have not been accessed again. The longest-idle item is evicted.
 
```
Before:                 A  B  C
After inserting D:      A  C  D   (B evicted: least recently used)
```
 
The intuition: recently accessed data may be more likely to be requested again.
 
### Least Frequently Used (LFU)

LFU removes the item with the lowest access frequency according to the policy's frequency-tracking rules.
 
```
A → 10 accesses
B →  2 accesses
C →  7 accesses
```
 
If the cache needs room, B may be selected for eviction because it has been accessed least. LFU can be useful when some data is consistently more popular than other data. However, implementation details matter, including how frequency is tracked and how ties are resolved.
 
### First In, First Out (FIFO)
 
FIFO removes the item that entered the cache earliest, regardless of how recently it was accessed.
 
```
Inserted first:  A
Inserted second: B
Inserted third:  C
```

When D arrives, A is removed. The intuition is simple: remove the oldest entry by insertion time.
 
### Why do these policies matter?
 
Different applications have different access patterns. Some benefit from retaining recently used data, others from retaining frequently used data. The eviction policy influences which data remains available, and therefore affects the **cache hit rate**. No single policy is best for every workload.
 
## 11. How Caching Improves Performance
 
Let's compare a simplified scenario. Suppose retrieving a result from the database takes **200 ms**, while retrieving it from the cache takes **5 ms**.
 
```
Without caching:   Request → Database Query → 200 ms
With caching (hit): Request → Cache Lookup  →   5 ms
```
 
For a cache hit, the reduction in latency is:
 
```
200 ms - 5 ms = 195 ms   (a 97.5% reduction in retrieval time)
```
 Now imagine 1,000 requests with a 90% hit rate:
 
```
Total Requests: 1,000
Cache Hits:       900
Cache Misses:     100
```
 
Instead of sending all 1,000 requests to the database, only about 100 require database retrieval. The cache reduces the database's read workload, which can improve:
 
- Response times for cache hits.
- Database capacity available for other operations.
- The number of requests the overall system can support.
- Infrastructure efficiency.
However, the entire application's response time will **not** necessarily fall by 97.5%. Other work, such as authentication, network communication, and processing, still takes time. Caching reduces one particular source of work, and its overall impact depends on how much of the request it eliminates.

## 12. Caching in a Real Application
 
Consider an AI-assisted data analysis system like DataPilot. A user uploads a CSV file and asks a question about the data. The application processes the request and generates an answer. Some operations may be expensive, and users may repeat requests or revisit previously generated results.
 
Could caching help? Potentially. Imagine DataPilot stores generated report results using a key associated with the dataset and the analysis parameters:
 
```
User Question
      │
      ▼
Analysis Request
      │
      ▼
Check Cache
      │
      ├── Cache Hit
      │      │
      │      ▼
      │   Return Cached Result
      │
      └── Cache Miss
             │
             ▼
       Process Analysis
             │
             ▼
       Store Result
             │
             ▼
        Return Result
```
 
If an equivalent request is made again and the cached result is still valid, the application may avoid repeating the expensive analysis.
 
But there is an important consideration: **two requests containing the same question are not necessarily equivalent.** The uploaded datasets may differ, the analysis parameters may differ, and the underlying data may have changed. So a cache key must identify the relevant inputs, not merely the user's question:
 
```
Dataset Version
      +
Analysis Operation
      +
Parameters
      =
Cache Identity
```
 
The cache should return a result only when the inputs and relevant conditions match. Other concepts from this guide apply too:
 
- **TTL / invalidation:** when a dataset is replaced, results tied to the old version must stop being served.
- **Write strategy:** newly generated reports could be written through to the cache and database so they are immediately reusable, or written around if most reports are rarely reopened.
- **Eviction:** a limited cache should keep recently used reports and drop those no one revisits.
> Caching is not just about storing a result. It is about determining when that result can safely be reused.
 
## 13. What Caching Does Not Solve
 
Caching improves performance, but it does not automatically guarantee:
 
- That cached data is always up to date.
- That the database can never become overloaded.
- That every request will be faster.
- That the cache will never fail.
- That multiple application instances will always observe the same value.
- That the system can scale indefinitely.
A poorly designed cache can even create additional problems. For example, if a popular cache entry expires and thousands of requests simultaneously try to retrieve it, all of them may query the database at once. This is known as a **cache stampede**. The application may need additional strategies, such as coordinating cache refreshes, to prevent that burst.
 
Caching reduces repeated work, but the system still needs to handle expiration, failures, concurrent requests, and consistency. For a small application, a simple cache may be sufficient. For a large distributed system, caching can require its own design decisions.
 
## 14. The Evolution So Far
 
Each concept addresses a limitation encountered at the previous stage:
 
```
Database
   │
   │ Problem: repeatedly retrieving the same data
   ▼
Caching
   │
   │ Problem: cached data becomes outdated
   ▼
TTL and Cache Invalidation
   │
   │ Problem: writes create two copies that can disagree
   ▼
Write Strategies (Write-Through, Write-Back, Write-Around)
   │
   │ Problem: cache capacity is limited
   ▼
Eviction Policies (LRU, LFU, FIFO)
```
 
- A **database** provides persistent storage and a source of truth.
- **Caching** reduces repeated retrieval work.
- **TTL and invalidation** help control the freshness of cached data.
- **Write strategies** define how updates flow between the cache and the database.
- **Eviction policies** determine what happens when the cache needs space.
Together, these concepts help us improve performance without ignoring correctness.
 
## 15. Key Takeaways
 
- Caching stores copies of data so they can be reused, because repeatedly retrieving the same data wastes time and resources.
- A **cache hit** means usable data was found in the cache. A **cache miss** means it must be retrieved elsewhere.
- A high hit rate can reduce database load, but does not guarantee good overall performance.
- Caches can live in application memory, distributed services, browsers, and CDNs.
- **TTL** determines how long a cached entry may remain valid.
- **Cache invalidation** removes or marks cached data unusable when it should no longer be served.
- **Write strategies** decide how writes flow between cache and database:
  - **Write-through:** write to the cache and database together.
  - **Write-back:** write to the cache first, the database later.
  - **Write-around:** write to the database and bypass the cache.
- **Eviction policies** such as LRU, LFU, and FIFO determine which entries are removed to make room.
- Caching introduces trade-offs involving memory, freshness, availability, and consistency.
- Cached results must be keyed appropriately so different inputs do not incorrectly share a result.
- Caching reduces latency and database workload, but does not eliminate the need for a well-designed underlying system.
> **The central lesson:** Caching is the engineering decision to reuse previous work instead of repeating it, while ensuring that the reused result is still appropriate for the current request.









