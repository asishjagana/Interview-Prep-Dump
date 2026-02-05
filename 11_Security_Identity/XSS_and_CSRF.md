# Cross-Site Scripting (XSS) & CSRF

## 1. Why Should I Care About This?
- **User Compromise**: XSS allows a hacker to run *their* code in *your* user's browser. They can steal cookies, record keystrokes, and act as the user.
- **Financial Fraud**: CSRF (Cross-Site Request Forgery) allows a hacker to trick a user into clicking a link that silently performs an action (like "Send $1,000 to Hacker") on a site where the user is already logged in.
- **Web Security 101**: These are two of the most famous and dangerous web vulnerabilities in history. You cannot build a safe website without understanding them.

## 2. What Problem Does This Solve?
- **Malicious Script Injections**: Stopping hackers from "infecting" your website with their scripts.
- **Action Forgery**: Ensuring that every action (Post, Delete, Buy) was actually intended by the user.
- **Naive Approach**: Thinking that "If I use a framework, I'm safe." While modern frameworks help, they don't solve 100% of the problem.

## 3. Core Idea (Intuition First)
- **XSS (Cross-Site Scripting)**: Like **A Hacker writing a fake message on your Fridge**. They write a note that says "Please give the keys to the man in the van." When you (the User) read the fridge, you follow the instruction because you trust the fridge.
- **CSRF (Request Forgery)**: Like **A Hacker hiding a signature in a "Free Coupons" flyer**. You click the flyer, and it has a "Invisible Stamp" that tells your bank: "Send money now." Because the bank sees your real hand (your Cookie/Session), it does what the stamp says.

## 4. How It Actually Works
### 1. XSS Types
- **Stored XSS**: The hacker saves a script in your DB (e.g., in a "Comment"). Every user who visits the page "downloads" the hacker's script.
- **Reflected XSS**: The script is in the URL: `myapp.com/search?q=<script>alert('hacked')</script>`. The server "reflects" this script back into the page.
- **DOM-based XSS**: The script happens entirely in the browser's JavaScript without touching the server.

### 2. CSRF Mechanics
- The hacker hosts a site `evil.com`. It has an invisible form that auto-submits to `mybank.com/transfer?to=hacker`.
- If you are logged into `mybank.com`, your browser automatically attaches your "Session Cookie."
- The bank thinks you clicked the button!

## 5. Real-World Example
**The "Samy Worm" (MySpace)**:
- In 2005, a user named Samy added a bit of **Stored XSS** to his profile.
- Anyone who viewed his profile would automatically "friend" him AND copy the script to their own profile.
- Within 20 hours, Samy had 1 million friend requests, and MySpace had to be shut down. This was XSS in its purest form.

## 6. Code Example
**Prevention 1: Escaping (XSS)**
```javascript
// BAD - vulnerable to XSS
document.getElementById('welcome').innerHTML = "Hello " + userName;

// GOOD - escapes HTML tags
document.getElementById('welcome').innerText = "Hello " + userName;
```
**Prevention 2: Anti-CSRF Tokens**
```html
<!-- The server gives you a unique, random 'token' -->
<form action="/transfer" method="POST">
  <input type="hidden" name="_csrf" value="abc_123_unique_token">
  <input type="number" name="amount">
  <button type="submit">Send</button>
</form>
<!-- The server ONLY processes the form if the token matches the session -->
```

## 7. Design Variations & Alternatives
- **Content Security Policy (CSP)**: A browser header that says: "Only run scripts from `my-trusted-site.com`." This blocks almost all XSS, even if your code is buggy.
- **SameSite Cookies**: A cookie setting (`SameSite=Lax` or `Strict`) that tells the browser: "Don't send this cookie if the request started on a different website." This is the modern "Magic Weapon" against CSRF.

## 8. Pros and Cons
### Escaping/Sanitization (XSS)
- **Pros**: Simple, fast, and supported by all languages.
- **Cons**: You have to remember to do it *everywhere*. If you miss one spot, you're hacked.

### CSRF Tokens
- **Pros**: Extremely reliable protection.
- **Cons**: Makes "Stateless" APIs (like purely RESTful ones) harder to build because the server needs to remember the token.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Broken CSP"**: You set a CSP, but you include `'unsafe-inline'` because it's "easier." Now your CSP is useless.
- **The "JSON CSRF"**: You protect your forms with tokens, but you forget to protect your AJAX/JSON endpoints.
- **Sanitization Bypass**: A hacker uses weird encoding (e.g., `%3Cscript%3E`) to trick your sanitizer.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Vital for user account safety.
- **Cost**: Minor performance hit for templating engines to escape strings.

## 11. When NOT to Use This
- **Never**: You should always protect against XSS and CSRF.
- **Exception**: Public, read-only "Static Site Generators" (like Jekyll) that have no forms and no cookies are naturally immune to CSRF.

## 12. Interview Perspective
- **What interviewers expect**: The difference between **Stored vs Reflected XSS**. Knowledge of **Anti-CSRF Tokens**. Understanding of **SameSite Cookies** and **Content Security Policy (CSP)**.
- **Common question**: "How do you prevent a user from being tricked into a bank transfer by a malicious link?" (Answer: CSRF tokens and SameSite cookies).

## 13. Connections to Other Topics
- **Authentication**: JWTs (vulnerable to XSS theft).
- **Security & Identity**: The category this belongs in.
- **Rate Limiting**: Can stop a Samy-style worm from spreading too fast.

## 14. Visual Aids
- **The Cookie Heist (XSS)**: Script sends `document.cookie` to `hacker.com/log?c=...`.
- **The Invisible Stamp (CSRF)**: [Evil Site] --POST Request (with real cookie)--> [Good Site].

## 15. Summary (TL;DR)
**XSS is an attack where a hacker runs malicious code in a user's browser, prevented by input sanitization and CSP; CSRF is an attack where a hacker tricks a logged-in user into performing an unwanted action, prevented by Anti-CSRF tokens and SameSite cookies.**

## 16. Further Reading & Resources
- [OWASP Top 10: XSS](https://owasp.org/www-community/attacks/xss/)
- [OWASP Top 10: CSRF](https://owasp.org/www-community/attacks/csrf)
- [MDN: Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [PortSwigger Academy (Excellent Labs)](https://portswigger.net/web-security/cross-site-scripting)
