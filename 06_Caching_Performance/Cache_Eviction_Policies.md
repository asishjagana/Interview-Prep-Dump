# Cache Eviction Policies

## 1. Why Should I Care About This?
- **Infinite Data, Finite RAM**: RAM is expensive and limited. You cannot store everything. Eviction policies define who gets to stay in the "Fast Lane" and who gets kicked back to the "Slow Disk."
- **Cache Hit Ratio**: Choosing the right policy can be the difference between a 95% hit ratio (fast app) and a 10% hit ratio (slow app). 
- **System Stability**: If a cache doesn't have an eviction policy, it will eventually run out of memory (OOM), potentially crashing your entire application or the Redis node itself.

## 2. What Problem Does This Solve?
- **Stale/Useless Data**: Removes data that is no longer being used to make room for fresh, high-demand data.
- **Memory Management**: Prevents the application from consuming all available system resources.
- **Naive Approach**: Clearing the entire cache every 1 hour. This is a disaster because it causes periodic spikes of 100% database load.

## 3. Core Idea (Intuition First)
Eviction is like **Managing a Small Bookshelf**.
- **FIFO (First-In-First-Out)**: You remove the oldest book you bought, even if you still read it every day. (Fair, but often dumb).
- **LRU (Least Recently Used)**: You remove the book you haven't touched in the longest time. (Usually very smart).
- **LFU (Least Frequently Used)**: You remove the book that has been read the fewest times total. (Smart for long-term popularity).

## 4. How It Actually Works
### 1. LRU (Least Recently Used)
- Tracks the "Time of Last Access." When the cache is full, it discards the item that hasn't been used for the longest period.
- **Best For**: Systems with "Temporal Locality" (if you used it once, you'll probably use it again soon).

### 2. LFU (Least Frequently Used)
- Tracks a "Counter" for every item. It discards the item with the lowest count.
- **Best For**: Static data that has long-term popularity (e.g., the homepage logo).

### 3. FIFO (First-In-First-Out)
- Discards the oldest item first, regardless of how often or recently it was accessed.
- **Best For**: Simple systems where access patterns are random.

### 4. Random Eviction
- Discards an item at random.
- **Best For**: Low-overhead systems where tracking access time/frequency is too expensive for the CPU.

## 5. Real-World Example
**The Amazon "Trending Now" Section**:
- **LRU** is perfect here. Items that people are clicking *right now* stay in the cache. As soon as a product stops being "Trendy" (no clicks for an hour), it gets evicted to make room for the next big thing.
- **LFU** might be used for the "Amazon Logo" or "Header Data." These aren't just trendy; they are used by *everyone, always*. They should never be evicted.

## 6. Code Example
Implementing a simple LRU Cache using Python's `OrderedDict`:
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key):
        if key not in self.cache:
            return -1
        # Move accessed item to the end (Mark as most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            # Pop from the beginning (Least recently used)
            self.cache.popitem(last=False)
```

## 7. Design Variations & Alternatives
- **TTL (Time To Live)**: Not an eviction policy per se, but an "Expiration" rule. Data dies after $X$ seconds regardless of usage.
- **Sliding Window**: Similar to LRU, but uses a time window instead of a strict capacity count.
- **Segmented LRU**: Using two caches (Hot and Cold) to prevent one-time "Scans" from wiping out the entire useful cache.

## 8. Pros and Cons
### LRU
- **Pros**: Matches 90% of real-world access patterns. Easy to implement.
- **Cons**: Can be "Polluted" by a one-time scan (e.g., a search engine indexing your site).

### LFU
- **Pros**: Protects long-term popular items.
- **Cons**: New items struggle to enter the cache because their "count" starts at 1. Older, "dead" items with high historical counts can linger too long.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Bouncing" Cache**: Capacity is too small, so Item A is evicted to make room for B, then B is evicted for A, 1,000 times a second. Your DB load stays at 100%. (Solution: Scale up the cache size).
- **Metadata Overhead**: If you have 1 billion tiny items, the memory used to track "Last Access Time" for LRU can be larger than the actual data!
- **Clock Dependency**: In distributed systems, if different nodes have different system clocks, LRU logic across nodes can become inconsistent.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Proper eviction ensures your cache remains a high-performance "Shield."
- **Cost**: Saves money by allowing you to use a smaller, cheaper RAM node to serve a huge percentage of traffic.

## 11. When NOT to Use This
- **Small Datasets**: If your entire database is 50MB, just load it all into a HashMap and forget about eviction.
- **Critical Data Only**: If you only cache data that *must* be there, use an error or a crash instead of an eviction.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **LRU vs LFU**. Knowing the time complexity ($O(1)$) and space complexity. Ability to whiteboard an LRU implementation using a **Doubly Linked List + HashMap**.
- **Common question**: "How would you design a cache that handles both recent spikes and long-term popularity?"

## 13. Connections to Other Topics
- **Content Delivery Networks (CDN)**: CDNs use massive LRU sets at the edge.
- **CPU Caching**: L1/L2/L3 caches in your processor use similar eviction (often PLRU).

## 14. Visual Aids
- **Ordered List Diagram**: Showing an item jumping from the middle to the front on "Access."
- **Hit Ratio Graph**: Visualizing the trade-off of cache size vs hit percentage.

## 15. Summary (TL;DR)
**Cache eviction policies are the rules that decide which data stays in fast memory and which is discarded, primarily aiming to maximize the cache hit ratio while staying within memory limits.**

## 16. Further Reading & Resources
- [Redis: Using Redis as an LRU Cache](https://redis.io/docs/manual/eviction/)
- [Wikipedia: Cache Replacement Policies](https://en.wikipedia.org/wiki/Cache_replacement_policies)
- [LeetCode: LRU Cache Problem](https://leetcode.com/problems/lru-cache/)
- [GeeksForGeeks: LFU Cache Implementation](https://www.geeksforgeeks.org/lfu-cache-implementation/)
