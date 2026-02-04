# WebSockets (Full Duplex)

## 1. Why Should I Care About This?
- **Real-Time Speed**: Standard HTTP is "Ask-and-Receive." WebSockets are "Just Talk." This is the only way to build lightning-fast chat apps, stock tickers, or multiplayer games.
- **Low Overhead**: Once the connection is open, there are no heavy HTTP headers (Cookie, User-Agent, etc.) sent with every message. You save massive amounts of bandwidth.
- **Push Capability**: The server can send data to the client *at any time* without the client asking.

## 2. What Problem Does This Solve?
- **Polling Latency**: Eliminating the need for the browser to ask "Any new messages?" every 1 second (which is slow and wastes battery).
- **Efficiency**: Reducing server load by maintaining one long-lived connection instead of opening and closing 1,000s of HTTP connections.
- **Naive Approach**: Using `setInterval(() => fetch('/api'), 1000)`. This destroys your server and your user's data plan.

## 3. Core Idea (Intuition First)
WebSockets are like **An Open Phone Call**.
- **HTTP**: Like **Sending Letters**. You write a letter, send it, and wait 2 days for an answer. Every time you want to say something, you have to buy a new stamp and envelope.
- **WebSocket**: You call someone on the phone. You both keep the line open. You can talk, they can listen. They can talk, you can listen. You don't have to dial the number again until you are permanently done.

## 4. How It Actually Works
### 1. The Handshake (Upgrade)
- It starts as a normal HTTP request.
- The client sends a header: `Upgrade: websocket`.
- If the server agrees, it sends back: `101 Switching Protocols`.
- From that moment on, the TCP connection stays open and switches to the "WebSocket Protocol."

### 2. Framing
- Data is sent in "Frames."
- A frame can be **Text** or **Binary**.
- It uses a minimal 2-14 byte header, compared to the 500-1000 byte headers of HTTP.

### 3. Statefulness
- Unlike HTTP, WebSockets are **Stateful**. The server knows exactly who is on the other end of the socket without re-authenticating every message.

## 5. Real-World Example
**The "Live Trading" App**:
- A user is watching the price of Bitcoin. The price changes 10 times a second.
- **HTTP**: The browser asks 10 times a second. The server answers 10 times. 90% of the time, the price hasn't even changed, but the server still has to process the request.
- **WebSocket**: The browser opens one connection. The server only sends a small binary message *only when the price actually moves*. The user sees the price update instantly with zero delay.

## 6. Code Example
Implementing a simple WebSocket server and client in JavaScript:
```javascript
// SERVER (Node.js + 'ws' library)
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  console.log('Client connected');
  ws.on('message', (message) => {
    console.log(`Received: ${message}`);
    ws.send(`Echo: ${message}`);
  });
  ws.send('Welcome to the Live Stream!');
});

// CLIENT (Browser API)
const socket = new WebSocket('ws://localhost:8080');
socket.onopen = () => socket.send('Hello Server!');
socket.onmessage = (event) => console.log(`Data from server: ${event.data}`);
```

## 7. Design Variations & Alternatives
- **Server-Sent Events (SSE)**: Better for 1-way streaming (Server -> Client only). (See separate file).
- **Long Polling**: A "Hack" for old browsers that don't support WebSockets.
- **Socket.io**: A library that starts with WebSockets but can "Fallback" to polling if the connection is blocked by a corporate firewall.

## 8. Pros and Cons
### Pros
- **Fastest real-time performance**.
- **Truly Full-Duplex** (Both sides send at the same time).
- **Reduces server overhead** for high-frequency updates.

### Cons
- **Hard to Scale**: Requires **Sticky Sessions** (or a Shared Pub/Sub like Redis) because the user's "State" lives on ONE specific server.
- **Consumes Resources**: Each open socket uses some RAM. 1 million users = 1 million open connections.
- **Proxy/Load Balancer Issues**: Some proxies automatically close "Idle" connections after 60 seconds (Need to implement "Pings/Pongs").

## 9. Failure Scenarios & What Can Go Wrong
- **Connection Zombie**: The client loses internet (e.g., Tunnel), but the server doesn't realize it and keeps a "Dead" socket open, wasting RAM. (Solution: Heartbeats).
- **Load Balancer Disconnect**: A load balancer reloads its config and kills all 100,000 open WebSockets instantly, causing a "Thundering Herd" of 100,000 clients trying to reconnect at the same time.
- **Security**: Because it's stateful, "Cross-Site WebSocket Hijacking" is a risk if you don't validate the `Origin` header.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: **Harder** than HTTP. You cannot just add a new server; you need a way for Server A to talk to Server B if User 1 (on A) wants to message User 2 (on B). (Usually solved with Redis Pub/Sub).
- **Cost**: Saves bandwidth but increases RAM costs since you can't just "Close" the request.

## 11. When NOT to Use This
- **Standard CRUD**: If you are just fetching a user's profile once, WebSockets are overkill and add unnecessary complexity.
- **SEO/Public Content**: Search engines don't "Index" WebSocket data.

## 12. Interview Perspective
- **What interviewers expect**: The ability to describe the **Handshake**. Understanding of **Sticky Sessions** and how to scale with **Redis Pub/Sub**. Knowledge of **Ping/Pong** frames. 
- **Common question**: "How would you design a chat system for 10 million users?" (Answer: WebSocket servers + Redis/Kafka backbone).

## 13. Connections to Other Topics
- **Communication Protocols**: The 2nd most popular model after HTTP.
- **Message Queues**: Used to orchestrate messages between different WebSocket servers.
- **Load Balancing**: Requires special configuration to handle long-lived connections.

## 14. Visual Aids
- **The Sequence Diagram**: HTTP Handshake -> 101 Switch -> Bidirectional Frames -> Close.
- **Scaling Architecture**: [Client] -> [LB] -> [WS Server] <-> [Redis Bus] <-> [WS Server] <- [Client].

## 15. Summary (TL;DR)
**WebSockets provide a permanent, full-duplex communication channel over a single TCP connection, making them the gold standard for high-performance, real-time applications.**

## 16. Further Reading & Resources
- [MDN: The WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [RFC 6455: The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455)
- [Heroku: WebSockets and the Real-time Web](https://devcenter.heroku.com/articles/websockets)
- [Socket.io Documentation](https://socket.io/docs/v4/)
