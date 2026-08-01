# Rate Limiting & DoS Protection (Security Focus)

## 1. Why Should I Care About This?
- **Availability (The "A" in CIA)**: If your server is busy processing a hacker's 10,000 fake requests, it can't help your real customers. A DoS attack is just a "Digital Lockdown" of your business.
- **Fraud Prevention**: Rate limiting is the only thing that stops a "Brute Force" bot from trying 100,000 passwords on a user's account in 1 minute.
- **Cost Protection**: If your cloud app auto-scales to handle a massive wave of garbage traffic, you will wake up to a $50,000 server bill.

## 2. What Problem Does This Solve?
- **Resource Exhaustion**: Preventing an attacker from "Drowning" your CPU, RAM, or Database connections.
- **Inventory/Scraping Bots**: Protecting your prices and inventory from being "Scraped" every second by competitors.
- **Naive Approach**: Thinking "I'll just ignore the extra requests." (Ignoring them still costa CPU/Bandwidth).

## 3. Core Idea (Intuition First)
- **Rate Limiting**: Like **A Security Guard at a Subway Station Entrance**. He only lets 1 person through the turnstile every 2 seconds. It doesn't matter how many people are in the street; the platform stays safe and un-crowded.
- **DoS (Denial of Service) Protection**: Like **Building a Giant Wall with a Small, Fast Gate**. You deflect the 1 million people shouting in the street (Deflection) and only even "talk" to the few who look like real passengers.

## 4. How It Actually Works
### 1. Types of Limits
- **IP-based**: "Only 10 requests per minute from IP 1.2.3.4."
- **User-based**: "Logged-in User 'Bob' can only make 500 API calls per hour."
- **Endpoint-based**: "The /login route can only handle 10 attempts per minute total (to prevent brute force)."

### 2. Implementation Layers
- **L3/L4 (Network)**: Deflecting massive "Volumetric" attacks (like UDP floods) before they even reach your code. (Done by Cloudflare/AWS).
- **L7 (Application)**: Logic-based limiting (like "Only 5 password resets per day"). (Done in Nginx or your code).

### 3. Response Strategy
- **HTTP 429**: "Too Many Requests." Tells the client they are blocked and how long to wait.
- **Tarpitting**: Deliberately slowing down the response (e.g., "Wait 10 seconds to log in") to make it expensive for bots to continue.

## 5. Real-World Example
**The "Credential Stuffing" Bot**:
1. A hacker buys a list of 1 million leaked passwords from another site.
2. They run a bot to try those 1 million passwords on *your* site.
3. **Without Rate Limiting**: The bot tries 1,000 passwords per second. It hits a "Match" in 10 minutes. 
4. **With Rate Limiting**: After 5 failed attempts, your server blocks that IP for 10 minutes. The bot would now take **19 years** to check the whole list. The hacker gives up and attacks someone else.

## 6. Code Example
Implementing a simple rate limiter using Redis in Node.js:
```javascript
const redis = require('redis');
const client = redis.createClient();

async function rateLimiter(req, res, next) {
  const ip = req.ip;
  const count = await client.incr(ip); // Atomic increment

  if (count === 1) {
    await client.expire(ip, 60); // Reset count every 60 seconds
  }

  if (count > 20) {
    return res.status(429).send("Slow down! You are limited to 20 req/min.");
  }
  next();
}
```

## 7. Design Variations & Alternatives
- **WAF (Web Application Firewall)**: A smart cloud filter that uses AI to detect "Bot-like" behavior.
- **Captchas**: Asking the user to "Prove they are human" when they hit a limit.
- **Load Shedding**: Dropping *all* non-critical traffic during a massive attack to save the database.

## 8. Pros and Cons
### Pros
- **Stability**: Keeps the site online during a surge.
- **Security**: Effectively stops automated brute-force attacks.
- **Predictability**: You can cap your server costs.

### Cons
- **False Positives**: A library of 100 students all sharing one school IP might be accidentally "Blocked" because they look like one big bot.
- **State management**: At high scale, you need a very fast (Redis) DB just to track the limiters.
- **User Friction**: Real users hate seeing "Too many requests" if they just clicked refresh a few times.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Shared IP" Problem**: You block a whole office building (one public IP) because one employee was running a bad script.
- **Redis Crash**: Your rate-limiting DB dies. Your application either:
    - **Fails Closed**: Nobody can log in (bad).
    - **Fails Open**: The hacker has zero limits (dangerous).
- **Distributed Attack (DDoS)**: The hacker uses 10,000 different IPs. Each IP only makes 1 request per minute. Your IP-based limiter never trips, but your server still crashes from the total volume. (Solution: Use a Cloud-scale protection service like Cloudflare).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Drastically increases "Uptime" during attacks.
- **Cost**: Saves money on server scaling, but adds a small cost for Redis/WAF.

## 11. When NOT to Use This
- **Internal Only Networks**: If your app is inside a locked-down company VPN with no guests, you might skip strict rate limiting for performance.
- **Health Checks**: Never rate limit your own Load Balancer's health check, or it might accidentally think your server is dead.

## 12. Interview Perspective
- **What interviewers expect**: The difference between **Rate Limiting** (Application level) and **DoS Protection** (Network level). Knowledge of **429 status code**. Understanding of **Redis-based counters**.
- **Common question**: "How do you stop a user from brute-forcing passwords?" (Answer: Rate limit on **both** keys — per IP *and* per username — with **exponential backoff and step-up verification** rather than a hard lockout. A hard lockout triggered by unauthenticated input lets an attacker with a credential dump lock every customer out of their own account: a self-inflicted DoS. See [Bot Detection & CAPTCHA Challenges](./Bot_Detection_CAPTCHA_Challenges.md).)

## 13. Connections to Other Topics
- **Load Balancing**: LBs often handle the first layer of limiting.
- **Authentication**: Protecting the login flow.
- **Caching**: Rate limiting metadata must be stored in a fast cache (Redis).
- [Rate Limiting](../02_API_Interface_Design/Rate_Limiting.md) — **the algorithms and the client contract live there**: token bucket vs sliding window, distributed counters, and the `429`/`Retry-After` behaviour clients must implement. This file stays on the security angle (brute force, volumetric vs L7 abuse).
- [Bot Detection & CAPTCHA Challenges](./Bot_Detection_CAPTCHA_Challenges.md) — why per-IP limits alone fail against residential proxies, and the risk-scoring funnel that sits on top of these counters.
- [MFA, Passwordless & WebAuthn](./MFA_Passwordless_WebAuthn.md) — rate limiting keeps a 6-digit TOTP code from being brute-forced and push prompts from being spammed.
- [Session Management & Cookie Security](./Session_Management_Cookie_Security.md) — the idle/absolute timeouts that bound abuse *after* a successful login.

## 14. Visual Aids
- **The Turnstile**: One person at a time vs a crowd.
- **The Layer Diagram**: [Cloudflare] -> [API Gateway/Nginx] -> [Application Code] -> [Redis].

## 15. Summary (TL;DR)
**Rate limiting and DoS protection are critical availability barriers that prevent resource exhaustion and automated security attacks by controlling the volume and frequency of incoming requests at multiple infrastructure layers.**

## 16. Further Reading & Resources
- [Cloudflare: What is a DDoS attack?](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)
- [OWASP: Denial of Service](https://owasp.org/www-community/attacks/Denial_of_Service)
- [Stripe: Rate Limiting Strategies](https://stripe.com/blog/rate-limiters)
- [Nginx: Rate Limiting Configuration Guide](https://www.nginx.com/blog/rate-limiting-nginx/)
