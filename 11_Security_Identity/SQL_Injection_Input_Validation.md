# SQL Injection & Input Validation

## 1. Why Should I Care About This?
- **Data Theft & Destruction**: SQL Injection (SQLi) allows a hacker to steal your entire database, bypass login screens, and even delete all your data with a single line of text.
- **The #1 Security Risk**: For over a decade, SQLi has been at the top of the OWASP Top 10 list. It is the most common way companies get hacked.
- **Input is Poison**: In security, the rule is **"Don't Trust User Input."** Every search box, email field, and login form is a potential entry point for an attacker.

## 2. What Problem Does This Solve?
- **Command Injection**: Preventing users from "tricking" your database into running their commands.
- **Unauthorized Data Access**: Ensuring users can only see the rows they are supposed to see.
- **Naive Approach**: Concatenating strings to build a query: `"SELECT * FROM users WHERE name = '" + userName + "'"` (The most dangerous code in the world).

## 3. Core Idea (Intuition First)
- **SQL Injection**: Like **A Hacker tricking a Bank Teller**. You hand a piece of paper to the teller that says: "Give me the balance for Account 123... **And also give me all the money in the vault.**" Because the teller (the SQL engine) isn't smart enough to distinguish between your "Request" and your "Hidden Instruction," they do both.
- **Parameterized Queries**: Like **A Bank Form with distinct boxes**. The teller says: "Put the Account Number in THIS box and the Amount in THAT box. If you write 'And the vault' in the account number box, I'm just going to try to find an account named 'And the vault'." The data and the command are physically separated.

## 4. How It Actually Works
### 1. The Vulnerability (String Concatenation)
- User enters `' OR '1'='1` in the password box.
- The query becomes: `SELECT * FROM users WHERE pass = '' OR '1'='1'`.
- Since `'1'='1'` is always true, the server logs the hacker in without a password.

### 2. The Solution (Parameterized Queries / Prepared Statements)
- The query is pre-compiled by the database: `SELECT * FROM users WHERE pass = ?`.
- The data (`' OR '1'='1`) is sent separately as a "Literal Value." 
- The DB engine looks for a user whose password is *literally* that string of characters. It doesn't "run" the logic.

### 3. Input Validation
- Whitelisting: "This field must only contain numbers 0-9."
- Sanitization: Stripping out characters like `;`, `--`, and `'`.

## 5. Real-World Example
**The "Little Bobby Tables" (XKCD)**:
- A school's computer system crashes because a mother named her son `Robert'); DROP TABLE Students;--`.
- When the school's software tried to insert the son's name, the SQL command executed the `DROP TABLE` command, deleting the entire student database.
- **The Lesson**: Even something as simple as a "Name" field can be a weapon if not handled correctly.

## 6. Code Example
**BAD (Vulnerable)**
```javascript
// Never do this!
const query = `SELECT * FROM products WHERE id = ${req.query.id}`;
db.execute(query);
```
**GOOD (Secure)**
```javascript
// Use placeholders (?) and pass the data as an array
const query = "SELECT * FROM products WHERE id = ?";
db.execute(query, [req.query.id]);
```

## 7. Design Variations & Alternatives
- **ORM (Object Relational Mapping)**: Tools like Sequelize or Prisma use parameterized queries by default, making it harder to accidentally write vulnerable code.
- **WAF (Web Application Firewall)**: A layer at the edge that detects and blocks `OR 1=1` style attacks before they even hit your server.

## 8. Pros and Cons
### Parameterized Queries
- **Pros**: 100% effective against SQLi. Better performance (DB caches the pre-compiled plan).
- **Cons**: Requires a tiny bit more effort than a simple string template.

### Input Validation
- **Pros**: Catches "Bad Data" early. Improves data quality.
- **Cons**: Can be annoying for users if the rules are too strict (e.g., "Your name cannot contain a hyphen").

## 9. Failure Scenarios & What Can Go Wrong
- **The "Dynamic Order By" Trap**: You use parameters for `WHERE` clauses, but you use string concatenation for the `ORDER BY` column name. This is still vulnerable to SQLi!
- **Sanitization Bypass**: You try to manually strip out `'` by replacing it with `\'`. A clever hacker uses double-encoding (e.g., `%2527`) to trick your filter.
- **Truncation**: You validate that the input is 20 characters, but the database column only holds 10. The rest is cut off, potentially creating a "Malformed" record that causes a bug.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves system stability and prevents catastrophic data loss.
- **Cost**: No performance cost; in fact, prepared statements are often faster.

## 11. When NOT to Use This
- **Never**: You should **always** use parameterized queries. There is no legitimate reason to build SQL queries using string concatenation in a production application.

## 12. Interview Perspective
- **What interviewers expect**: The **Teller vs Form** analogy. Knowledge of **Prepared Statements**. Understanding that **ORMs** aren't a silver bullet (you still have to be careful with "Raw Queries").
- **Common question**: "How do you prevent SQL injection?" (Answer: Use Prepared Statements/Parameterized Queries and validate all incoming data).

## 13. Connections to Other Topics
- **Databases**: Where the attack happens.
- **Encryption**: Doesn't stop SQLi, but makes the stolen data harder to use.
- **Security & Identity**: The category this belongs in.

## 14. Visual Aids
- **The "Broken Query" Diagram**: Showing how `' OR 1=1` changes the logic of the SQL statement.
- **The "Secure Box" Diagram**: Showing data inside a parameter "container" where it can't affect the code.

## 15. Summary (TL;DR)
**SQL Injection is a critical vulnerability that allows attackers to execute arbitrary database commands by manipulating input fields, prevented by strictly using parameterized queries and robust input validation.**

## 16. Further Reading & Resources
- [OWASP: SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [PortSwigger Academy: SQL Injection](https://portswigger.net/web-security/sql-injection)
- [W3Schools: SQL Injection](https://www.w3schools.com/sql/sql_injection.asp)
- [Bobby Tables: A Guide to Preventing SQLi](https://bobby-tables.com/)
