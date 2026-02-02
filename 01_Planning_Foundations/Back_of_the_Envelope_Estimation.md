# Back-of-the-Envelope Estimation

## 1. Why Should I Care About This?
- **The "Reality Check"**: Can we store 10 years of logs on a single SSD? If you don't do the math, you might build a system that is physically impossible to run.
- **Budgeting**: Knowing if you need 10 servers or 10,000 servers changes the project from a "$5k/month" to a "$5M/month" conversation.
- **Design Validation**: It tells you if you need a Cache, a CDN, or a completely different Database Strategy before you write a single line of code.

## 2. What Problem Does This Solve?
- **Unbounded Design**: Prevents designing a system for "infinite traffic" which is wasteful, or for "minimal traffic" which will crash on day one.
- **Naive Approach**: "We'll just scale horizontally." This fails when you realize the network bandwidth required for synchronization exceeds the capacity of the NICs.

## 3. Core Idea (Intuition First)
It's about **rough orders of magnitude**. You don't need to know if the storage is 4.2TB or 4.8TB; you need to know if it's **5TB** or **500TB**. If it's 5TB, it fits on one drive. If it's 500TB, you need a distributed storage cluster.

## 4. How It Actually Works
1.  **Start with DAU (Daily Active Users)**: e.g., 100M.
2.  **Estimate QPS (Queries Per Second)**: 
    - `100M users * 10 actions/day = 1B actions/day`.
    - `1B / 86400 seconds (roughly 100k) ≈ 12,000 QPS`.
    - **Peak QPS**: `Average * 2 ≈ 25,000 QPS`.
3.  **Storage Estimation**: 
    - `100M users * 1 photo/day * 200KB = 20TB/day`.
    - `5 years = 20TB * 365 * 5 ≈ 36PB`.
4.  **Memory (RAM)**: Use the **80/20 rule** (20% of data generates 80% of traffic). Cache that 20%.

## 5. Real-World Example
**Netflix Video Streaming**:
- If Netflix has 200M users and a movie is 2GB, they cannot stream everything from one central server. The math quickly shows that the total egress bandwidth would exceed the internet's capacity in certain regions, necessitating **CDNs (Open Connect)**.

## 6. Code Example (If Applicable)
Estimation logic often sits in **Capacity Planning** docs, but it informs code constants:
```python
# Constants informed by Back-of-the-Envelope Estimation
CACHING_STRATEGY = {
    "expected_qps": 25000,
    "avg_item_size_kb": 2,
    "cache_memory_limit_gb": 64, # Calculated to hold 20% of daily hot data
    "eviction_policy": "LRU"
}
```

## 7. Design Variations & Alternatives
- **Bottom-Up**: Calculate based on current hardware benchmarks (e.g., "1 Redis node handles 100k QPS").
- **Top-Down**: Calculate based on user growth projections.
- **The "Cheat Sheet" numbers**: (10^3 = K, 10^6 = M, 10^9 = G, 10^12 = T, 10^15 = P).

## 8. Pros and Cons
### Pros
- **Decision Speed**: Discard impossible designs in seconds.
- **Credibility**: Shows you understand the scale and the hardware.

### Cons
- **Input sensitivity**: If your initial assumption (e.g., "1 photo/day") is off by 10x, your final result is off by 10x.
- **Ignoring Overhead**: Forgetting TCP headers, replication factor (3x storage), and metadata.

## 9. Failure Scenarios & What Can Go Wrong
- **Precision Trap**: Spending 20 minutes calculating 1024 * 1024 instead of just using 1,000,000.
- **Missing the "Long Tail"**: Estimating average traffic but failing at the 99th percentile burst.
- **Storage vs. Throughput**: Having enough space for the data but not enough IOPS (Input/Output Operations per Second) to read it.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Tells you when a single leader database is no longer enough (usually > 10k Write QPS).
- **Cost**: `36PB of storage * $0.02/GB/month` is a very important number for the CFO.

## 11. When NOT to Use This
- **Micro-optimizations**: Don't use rough estimations for low-level performance tuning where microseconds matter.
- **Static/Toy Projects**: If the scale is "a few hundred users," any modern framework on a $5 VPS will work.

## 12. Interview Perspective
- **What interviewers expect**: Clean powers of 10 math. Round numbers (e.g., use 100k seconds in a day instead of 86,400).
- **Common numbers to know**: L1 cache latency (0.5ns), SSD seek (100μs), Round trip within same DC (500μs).
- **Red flags**: Being unable to calculate how much storage 100 million 1KB records take (≈ 100GB).

## 13. Connections to Other Topics
- **Latency Numbers Every Programmer Should Know**: The fundamental building blocks.
- **Sharding**: Usually triggered when the estimation exceeds the limits of one machine.

## 14. Visual Aids
- **Magnitude Chart**: From Bits to Petabytes.
- **Latency Comparison**: Visualizing the difference between RAM and Disk speed (The "City Block vs. Moon" analogy).

## 15. Summary (TL;DR)
Back-of-the-envelope estimation is your **engineering compass**. It doesn't tell you the exact coordinate, but it tells you if you're heading toward a workable solution or a cliff.

## 16. Further Reading & Resources
- *Latency Numbers Every Programmer Should Know* (Interactive Gist)
- *High Scalability* - "How to estimate capacity" guides.
- "The Power of 10" - Rule of thumb for scaling.
