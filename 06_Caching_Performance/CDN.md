# CDN (Content Delivery Network)

## 1. Why Should I Care About This?
- **The "World Wide" Speed**: If your server is in New York and a user is in Tokyo, the signal has to travel 7,000 miles. A CDN puts your content in a server *in* Tokyo, reducing latency from 200ms to 20ms.
- **Bandwidth Savings**: CDNs serve the "Heavy" assets (Photos, Videos, JS). This saves your origin server from massive bandwidth costs and prevents it from crashing under the weight of static files.
- **DDoS Protection**: CDNs are massive. They can absorb a million-request-per-second attack that would vaporize a normal origin server.

## 2. What Problem Does This Solve?
- **Speed of Light Bottlenecks**: Solving the physical limit of how fast data can travel across oceans.
- **Origin Overload**: Prevents your main server from doing the "dumb" work of serving the same 5MB image a million times.
- **Naive Approach**: Serving all assets directly from your Go/Node.js/Python server. This is slow and expensive.

## 3. Core Idea (Intuition First)
A CDN is like **Corner Stores (7-Eleven)**.
- **Origin Server**: The giant **Warehouse** in the middle of nowhere. It has everything, but it's far away.
- **CDN Edge Node**: The **Corner Store** right down the street. It only has the most popular items (Bread, Milk, Viral Memes). 
- If you need bread, you go to the corner store. It's instant.
- If the store is out of bread, *they* call the warehouse, get a shipment, and then sell it to you (and keep some for others).

## 4. How It Actually Works
- **Edge Locations (PoPs)**: Points of Presence. Data centers placed in major cities worldwide.
- **Pull Model (On-Demand)**: The CDN only caches content when a user first requests it. (Most common).
- **Push Model**: You explicitly upload assets to the CDN ahead of time (e.g., for a new video game release).
- **TTL & Invalidation**: Telling the CDN when an asset is "Too old" and needs to be fetched again from the origin.

## 5. Real-World Example
**Netflix Video Streaming**:
- Netflix doesn't stream movies from California to the world. They use **Open Connect** (their own CDN). They place physical hardware boxes *inside* the buildings of Internet Service Providers (ISPs) like Comcast or Verizon. When you watch "Stranger Things," the data travels a few miles from an ISP closet to your house, not across the country.

## 6. Code Example
Integrating a CDN into a web application:
```html
<!-- BAD: Loading from your own server -->
<img src="/assets/hero-banner.jpg">

<!-- GOOD: Loading from a CDN -->
<!-- Static assets are served from a sub-domain pointing to Cloudflare/CloudFront -->
<img src="https://cdn.myapp.com/assets/hero-banner.jpg">

<!-- HOW IT WORKS INTERNALLY (Pseudo-logic on the CDN) -->
<!-- 
If request for /assets/hero-banner.jpg:
    If in local storage and not expired:
        Return image (HIT)
    Else:
        Fetch from origin.com/assets/hero-banner.jpg
        Save copy to disk
        Return image (MISS)
-->
```

## 7. Design Variations & Alternatives
- **Static CDN**: Caches images, CSS, JS.
- **Dynamic CDN (Edge Computing)**: Running small bits of code (Cloudflare Workers / Lambda@Edge) at the CDN level to handle things like auth checks or image resizing without hitting the origin.
- **P2P CDN**: Users sharing pieces of a file with each other to reduce server load even further.

## 8. Pros and Cons
### Pros
- **User Experience**: Drastic improvement in page load times.
- **Availability**: Your site stays "up" even if your origin is down (it just serves the last cached version).
- **Security**: Built-in WAF (Web Application Firewall) and DDoS mitigation.

### Cons
- **Cost**: CDNs charge for data egress (can be expensive for video).
- **Complexity**: "Cache Invalidation" is hard. If you push a bug in your CSS, it can take 15 minutes to clear it from 200 worldwide servers.
- **Local Dev**: Hard to test CDN behavior on `localhost`.

## 9. Failure Scenarios & What Can Go Wrong
- **Origin Shield Failure**: Too many "Misses" happen at once, and the CDN forwards all requests to your origin, crashing it (The "Thundering Herd" again).
- **Stale Asset Bug**: You update your website's JavaScript, but the CDN keeps serving the old version, causing the site to break for users who have a mix of new HTML and old JS.
- **Geo-Blocking Mistakes**: Accidentally blocking an entire country due to a misconfigured CDN security rule.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Infinite for static content.
- **Reliability**: Improves global uptime.
- **Cost**: Reduces origin server costs but adds a new "Usage-based" variable cost.

## 11. When NOT to Use This
- **Private Data**: Never cache personal user info (Bank balances, private messages) on a public CDN.
- **Internal Tools**: If your app is used only by people in one office building, a global CDN is a waste.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **Push vs Pull**. Understanding how CDNs improve **TTFB (Time to First Byte)**. Knowledge of how to handle **Cache Invalidation** (e.g., using versions in filenames: `style.v2.css`).
- **Common question**: "How would you optimize a website for a global audience?"

## 13. Connections to Other Topics
- **DNS**: How users are routed to the *closest* CDN server.
- **Caching Strategies**: The logic CDNs use internally.
- **Anycast Routing**: The network technology that makes CDNs work.

## 14. Visual Aids
- **The Distance Diagram**: Origin Server -> Ocean -> User (Slow) vs Edge Node -> User (Fast).
- **Edge Cache Flow**: User -> Edge (Check) -> Origin (If missing).

## 15. Summary (TL;DR)
**CDNs are a global network of proxy servers that cache content closer to users to improve speed, reduce origin server load, and provide security.**

## 16. Further Reading & Resources
- [Cloudflare: What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)
- [AWS: How CloudFront Works](https://aws.amazon.com/cloudfront/details/)
- [Akamai: The Benefits of a CDN](https://www.akamai.com/our-thinking/cdn/what-is-a-cdn)
- [Web.dev: Content Delivery Networks](https://web.dev/cdn/)
