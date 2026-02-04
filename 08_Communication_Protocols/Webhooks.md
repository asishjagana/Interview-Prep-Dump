# Webhooks

## 1. Why Should I Care About This?
- **Inversion of Control**: Instead of YOUR app asking "Is it done yet?" 1,000 times, the OTHER app tells you the instant it's done.
- **Efficiency**: Zero wasted CPU cycles or bandwidth on empty polling requests.
- **Standard for Integration**: Every major SaaS (Stripe, GitHub, Slack, Shopify) uses Webhooks as their primary way to talk to your code.

## 2. What Problem Does This Solve?
- **Delayed Notifications**: Getting notified about a payment or a code push in milliseconds instead of minutes.
- **Third-Party Integration**: Letting external systems trigger logic inside your private backend.
- **Naive Approach**: Running a `while(true)` loop to check an external API for changes.

## 3. Core Idea (Intuition First)
Webhooks are like **Leaving Your Phone Number at a Restaurant**.
- **Polling**: You walk up to the hostess every 2 minutes and ask, "Is my table ready?" It's annoying for her and tiring for you.
- **Webhook**: You give the hostess your phone number and walk away to go shopping. When the table is ready, she sends you a **Text Message**. You only return to the restaurant when you are actually needed.

## 4. How It Actually Works
### 1. Registration
- You (The Client) provide a URL to the external system (The Provider). 
- Example: "Hey Stripe, send updates to `https://my-app.com/webhooks/stripe`."

### 2. The Trigger
- An event occurs in the provider's system (e.g., a customer pays a bill).

### 3. The Callback
- The provider makes an **HTTP POST** request to your registered URL.
- The body contains JSON data about the event.

### 4. Acknowledgment
- Your server must respond quickly with a `200 OK`. If you don't, the provider might think your server is dead and try sending it again later.

## 5. Real-World Example
**The "E-commerce Shipping" Flow**:
1. A customer buys a shirt. 
2. Your app sends the order to the "Shipping Company" API.
3. 2 days later, the postman scans the package as "Delivered."
4. The **Shipping Company** sends a **Webhook** to your app.
5. Your app receives the POST, updates the DB to "Delivered," and automatically sends a "How was your shirt?" email to the customer.

## 6. Code Example
Implementing a secure Webhook receiver in Node.js (with signature verification):
```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

const WEBHOOK_SECRET = 'your_super_secret_key';

app.post('/webhook', express.json(), (req, res) => {
  const signature = req.headers['x-hub-signature'];
  
  // SECURITY: Verify the message actually came from the trusted source
  const hmac = crypto.createHmac('sha256', WEBHOOK_SECRET);
  const digest = 'sha256=' + hmac.update(JSON.stringify(req.body)).digest('hex');

  if (signature !== digest) {
    return res.status(401).send('Invalid Signature');
  }

  console.log('Valid Event Received:', req.body.event_type);
  
  // Important: Respond fast!
  res.status(200).send('Received');
});
```

## 7. Design Variations & Alternatives
- **Pub/Sub (Message Queues)**: Used for internal events. Webhooks are for crossing company boundaries (Public Internet).
- **Websockets**: If the notification needs to go to a browser. Webhooks are usually **Server-to-Server**.

## 8. Pros and Cons
### Pros
- **Extremely efficient** for both parties.
- **Real-time** notification.
- **Easy to set up** (Standard HTTP).

### Cons
- **Unreliable Delivery**: If your server is down, you miss the event. (Solution: Providers should use "Retries with Backoff").
- **Security Risks**: Anyone can POST to your public URL. (Solution: Signature verification).
- **Debug Difficulty**: Hard to test locally because the provider can't "see" your `localhost`. (Solution: Tools like `ngrok`).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Webhook Storm"**: Your server was down for 1 hour. It comes back up. The provider instantly sends you 50,000 missed events, crashing your server all over again. (Solution: Use a Queue to ingest the events, respond `200` instantly, and process them slowly).
- **Duplicate Events**: The provider sent the same event twice because of a network glitch. (Solution: Your logic must be **Idempotent** - processing the same event twice shouldn't double-charge a user).
- **Out-of-Order Events**: You receive "Order Delivered" *before* "Order Shipped."

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves by decoupling systems.
- **Cost**: Reduces API costs since you don't have to poll 24/7.

## 11. When NOT to Use This
- **High Sensitivity Internal Communication**: If the message is between two services you control, use a Message Queue (Kafka/RabbitMQ) which has better durability guarantees.
- **Batch Processing**: If you only care about updates once a day, just do a single batch fetch.

## 12. Interview Perspective
- **What interviewers expect**: Knowledge of **Signatures/Security**. Understanding of **Idempotency**. Ability to explain why a **Queue** should sit behind a webhook receiver.
- **Common question**: "How do you ensure your webhook endpoint is secure?"

## 13. Connections to Other Topics
- **Idempotency**: The #1 requirement for webhook receivers.
- **Message Queues**: The best architectural pattern for handling webhook traffic.
- **Authentication**: Using signing secrets.

## 14. Visual Aids
- **The Loop vs The Push Diagram**: Polling (Circle) vs Webhook (Direct line).
- **Security Flow**: Provider (Sign) -> Request -> Receiver (Verify).

## 15. Summary (TL;DR)
**Webhooks are server-to-server HTTP callbacks that push real-time event notifications from a provider to a client, enabling efficient, event-driven architectures across system boundaries.**

## 16. Further Reading & Resources
- [Stripe: Webhooks Documentation](https://stripe.com/docs/webhooks) - The gold standard of webhook design.
- [GitHub: Creating Webhooks](https://docs.github.com/en/developers/webhooks-and-events/webhooks/creating-webhooks)
- [Webhooks.fyi](https://webhooks.fyi/) - A directory of webhook best practices.
- [Ngrok: Local Testing for Webhooks](https://ngrok.com/use-cases/webhooks)
