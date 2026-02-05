# Hashing & Salting (BCrypt, Argon2)

## 1. Why Should I Care About This?
- **The #1 Security Rule**: **NEVER store plain-text passwords**. If your database is leaked and you stored passwords as "123456", every user's account (and their other accounts with the same password) is compromised.
- **Protection Against "Rainbow Tables"**: Salting ensures that even if two users choose the same password, their "Stored Junk" in the database looks completely different.
- **Slow is Good**: Unlike normal code where we want "Fast," we want password hashing to be "Slow." This makes it impossible for a hacker to guess 1 billion passwords per second.

## 2. What Problem Does This Solve?
- **Data Breach Containment**: Ensuring that a leaked database is mostly useless to a hacker.
- **Brute Force Attacks**: Preventing hackers from using supercomputers to guess passwords.
- **Naive Approach**: Using simple `MD5` or `SHA-1` hashing. These are too fast and have been "broken" for years.

## 3. Core Idea (Intuition First)
- **Hashing**: Like **Turning a Cow into a Hamburger**. You can easily turn a cow into meat, but you can NEVER turn the hamburger back into a cow. It's a one-way process. 
- **Salting**: Like **Adding a Random Secret Spice** to every cow before you grind it. Even if two cows are identical, the final hamburgers will taste (look) different because of the unique spice. 
- **The Result**: You don't store the "Cow" (Password). You store the "Unique Hamburger" and the "Recipe for the Spice." When the user logs in, you take their cow, add the spice, grind it, and see if it match the hamburger you have in the safe.

## 4. How It Actually Works
### 1. The Salt
- A random string generated for every single user (e.g., `s3cr3t_77`).
- Stored in the database alongside the hash: `username | salt | hash`.

### 2. The Iterations (Cost Factor)
- Modern algorithms like **BCrypt** hash the password 1,000s of times. 
- A server handles 1 hash in 300ms (unnoticeable to one user).
- A hacker trying 1,000,000,000 combinations would take **9 years** instead of 1 second.

### 3. The Algorithms
- **Argon2**: The current "World Champion" (Winner of the Password Hashing Competition). It is resistant to GPU-based cracking.
- **BCrypt**: The industry standard for 20 years. Very safe and easy to use.
- **SHA-256**: Great for "Data Integrity," but **BAD for passwords** because it's too fast.

## 5. Real-World Example
**The "LinkedIn" Leak (2012)**:
- LinkedIn stored 6 million passwords using simple `SHA-1` with no salt. 
- Hackers used "Rainbow Tables" (pre-computed lists of common hashes) to reveal millions of passwords in minutes. 
- If LinkedIn had used **BCrypt + Salt**, the hackers would still be trying to crack those passwords today.

## 6. Code Example
Hashing a password using BCrypt in Node.js:
```javascript
const bcrypt = require('bcrypt');

const myPassword = "CorrectHorseBatteryStaple";
const saltRounds = 12; // The "Cost" factor

// 1. SIGNUP: Hash and store
const hash = await bcrypt.hash(myPassword, saltRounds);
// Store 'hash' in DB. It includes the salt automatically! 
// e.g. "$2b$12$R9h/..."

// 2. LOGIN: Compare
const match = await bcrypt.compare(providedPassword, storedHash);
if(match) {
  console.log("Welcome!");
}
```

## 7. Design Variations & Alternatives
- **Pepper**: Like a salt, but stored in the **Application Code** (or a Secret Manager) rather than the database. Even if the DB is stolen, the hacker doesn't have the "Pepper."
- **Argon2id**: The specific version of Argon2 recommended for most use cases (balances side-channel resistance).

## 8. Pros and Cons
### Pros
- **User Safety**: If your DB leaks, your user's "Identity" is still safe.
- **Low Effort**: Modern libraries handle the salt and iterations for you automatically.

### Cons
- **CPU Usage**: Doing 10,000 hashes per login is intensive. A massive DDoS attack on your "Login" page can take down your server by exhausting the CPU. (Solution: Rate limiting).
- **Complexity**: Choosing the right "Cost Factor." Too high and your server is slow; too low and it's insecure.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Re-hashing" Mistake**: You hash a password that was already hashed. Now the user can never log in.
- **Weak Salts**: Using the `username` as the salt. If the username is "Bob," it's not random enough. (Solution: Use a cryptographically secure random generator).
- **The "MD5" Legacy**: You have an old app from 2005 still using MD5. A hacker steals the DB and cracks every password in 5 seconds. (Solution: **Migrate on Login** - when a user logs in, check if their hash is old, and upgrade it to BCrypt).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves system trust.
- **Cost**: Minor CPU increase on your auth servers.

## 11. When NOT to Use This
- **Non-Passwords**: Don't use BCrypt for "Searching for a User." It's too slow. Use a standard indexing hash for that.
- **High-speed Data**: Don't use password-strength hashing for check-summing a file.

## 12. Interview Perspective
- **What interviewers expect**: The **Cow vs Hamburger** one-way logic. Knowledge of **Salting** (Rainbow tables). Understanding of **Cost Factors** (Slow hashing).
- **Common question**: "Why is SHA-256 bad for passwords?" (Answer: It's designed to be fast, which allows hackers to brute-force billions of guesses per second).

## 13. Connections to Other Topics
- **Authentication**: This is how you store credentials.
- **OAuth2**: Offloads this entire problem to Google/Microsoft.

## 14. Visual Aids
- **The Hashing Flow**: [Password] + [Salt] -> [Hash Algorithm x 12,000] -> [Final String].
- **Comparison Table**: MD5 (Unsafe) vs SHA-1 (Unsafe) vs Argon2 (Gold Standard).

## 15. Summary (TL;DR)
**Password hashing with unique salts and high cost-factors (like Argon2 or BCrypt) ensures that users' credentials remain secure and computationally difficult to crack even if the backend database is compromised.**

## 16. Further Reading & Resources
- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Auth0: Salt, Hash, and Hashing Algorithms](https://auth0.com/blog/adding-salt-to-hashing-algorithms/)
- [Argon2 Whitepaper](https://github.com/P-H-C/phc-winner-argon2/raw/master/argon2-specs.pdf)
- [Try it yourself: BCrypt Online](https://www.bcrypt-generator.com/)
