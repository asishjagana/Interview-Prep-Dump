# Pagination & Filtering

## 1. Why Should I Care About This?
- **The "OOM" (Out of Memory) Disaster**: If your database has 10 million users and you try to return them all in a single API call, your server will crash, your database will lock up, and the client's browser will freeze.
- **User Experience**: Nobody wants to scroll through 1,000 items at once. Users need to "sip" data in manageable chunks.
- **Bandwidth Usage**: Sending 100MB of JSON for a request that only needs the first 10 items is extremely expensive and slow.

## 2. What Problem Does This Solve?
- **Unbounded Queries**: Prevents a simple `SELECT * FROM table` from becoming a performance bottleneck as the data grows.
- **Naive Approach**: `Offset Pagination`. Using `LIMIT 10 OFFSET 1000000` is incredibly slow in SQL because the database must still read and discard 1 million rows to find the next 10.

## 3. Core Idea (Intuition First)
Pagination is like **Reading a Book**. 
- **Offset**: "Turn to page 45." (Fast if the book is thin, but hard to find if you have to count every page from the start).
- **Cursor**: "Pick up where you left off at the bookmark." (Instantly find the next page regardless of how many pages came before).

Filtering is like **The Index at the back of the book**. Instead of reading everything, you only look for specific keywords (e.g., `genre=fiction`).

## 4. How It Actually Works
### Pagination Strategies
1.  **Offset-based**: `/users?limit=20&offset=40`. Simple but slow for large datasets.
2.  **Keyset/Cursor-based**: `/users?limit=20&after_id=500`. Uses a unique, indexed field to skip directly to the next set. Much faster and prevents "skipped items" if new data is inserted while paging.

### Filtering Strategies
- **Exact Match**: `?color=red`.
- **Range Queries**: `?price_min=10&price_max=50`.
- **Full-Text Search**: Using specialized indices (`GIN` in Postgres or ElasticSearch).

## 5. Real-World Example
**Twitter / X Timeline**:
- Twitter uses **Cursors** (`since_id` and `max_id`). When you pull to refresh, the app says "Give me everything *newer* than the last tweet I saw." This ensures that even if 10,000 new tweets were posted while you were reading, you don't get duplicates or gaps.

## 6. Code Example
Implementing efficient Keyset (Cursor) pagination:
```python
# Efficient Keyset Pagination in SQL
def get_posts(after_id=None, limit=20):
    if after_id:
        # DB uses the index on ID to jump straight to the record
        query = "SELECT * FROM posts WHERE id > %s ORDER BY id ASC LIMIT %s"
        params = (after_id, limit)
    else:
        query = "SELECT * FROM posts ORDER BY id ASC LIMIT %s"
        params = (limit,)
    
    return db.execute(query, params)

# API Response
{
    "data": [...],
    "next_cursor": "521", # The 'id' of the last item in the list
    "has_more": true
}
```

## 7. Design Variations & Alternatives
- **Infinite Scroll**: A UI pattern that uses Cursor pagination in the background.
- **Load More Button**: Similar to infinite scroll but requires user action.
- **Numbered Pages**: Usually requires Offset pagination (harder to scale, but better for SEO/Direct links).

## 8. Pros and Cons
### Pros
- **Performance**: Keeps memory usage and DB load constant regardless of total table size.
- **Reliability**: Prevents timeouts on large data fetches.

### Cons
- **Limited Sorting**: Cursor pagination works best when sorting by a unique, immutable index (like `ID` or `CreatedAt`). Sorting by non-unique fields (like `FirstName`) is complex with cursors.
- **Complexity**: Cursors are harder for front-end developers to understand than simple "Page 1, Page 2."

## 9. Failure Scenarios & What Can Go Wrong
- **The "Deep Offset" Problem**: Performance degrading sharply once a user reaches "Page 10,000."
- **Inconsistent Results**: With Offset pagination, if an item is deleted from Page 1 while you are on Page 2, Page 2's first item will "shift" and you'll see a duplicate or skip an item.
- **Index Missing**: Adding a filter (e.g., `?status=active`) without a database index on that column, causing a full table scan for every request.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Cursor pagination is O(1) or O(log N) for fetching the next page, while Offset is O(N).
- **Cost**: Reduces server CPU and IOPS maintenance.

## 11. When NOT to Use This
- **Small Datasets**: If you only have 50 items total, just return them all and let the client filter/page locally.
- **Aggregates**: If a user needs a "Total Count" of 1M items, pagination doesn't help with the `COUNT(*)` overhead.

## 12. Interview Perspective
- **What interviewers expect**: Recognition of why `OFFSET` is bad for large data. Knowledge of **Cursor vs. Offset**. Ability to design a URL structure for filters.
- **Common question**: "How do you handle pagination in a system where data is frequently inserted?"

## 13. Connections to Other Topics
- **Indexing**: Necessary for performant filtering.
- **Consistent Hashing**: Relevant when pagination across shards.

## 14. Visual Aids
- **Offset vs Cursor Graph**: Showing the jump in DB time for high offsets.
- **Filter-to-Query Pipeline**: How `?q=search` is sanitized and converted to a SQL `WHERE` clause.

## 15. Summary (TL;DR)
**Never allow unbounded queries.** Use cursors for large-scale data and always index the columns you allow users to filter by.

## 16. Further Reading & Resources
- [Slack Engineering: Evolving API Pagination](https://slack.engineering/evolving-api-pagination-at-slack/)
- [Postgres: Why Offset is Slow](https://www.postgresql.org/docs/current/queries-limit.html)
- [Stripe: Cursor-based Pagination API Design](https://stripe.com/docs/api/pagination)
