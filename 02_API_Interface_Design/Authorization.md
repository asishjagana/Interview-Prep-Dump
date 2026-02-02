# Authorization

## 1. Why Should I Care About This?
- **Internal Threats**: Just because a user is authenticated (they work at the bank) doesn't mean they should be authorized to delete the bank's database or view the CEO's personal account.
- **The "Broken Access Control" Vulnerability**: The #1 vulnerability on the OWASP Top 10. It accounts for more data leaks than almost any other flaw.
- **Product Tiers**: Authorization is how you enforce "Free" vs "Premium" features.

## 2. What Problem Does This Solve?
- **Permission Management**: Controlling *what* a verified specific identity is allowed to do.
- **Naive Approach**: Hardcoding roles in your code (`if user.role == "admin"`). This fails as soon as you need "Moderators who can delete posts but not users" or "Admins who only work on Weekends."

## 3. Core Idea (Intuition First)
Authorization is like the **Keys in a Hotel**. 
- Authentication gave you the key to the Front Door.
- Authorization decides if that key opens **Room 402**, the **Gym**, or the **Restricted Server Room**. If you try to open the Server Room with a Room Key, the lock stays shut.

## 4. How It Actually Works
Three Main Models:
1.  **RBAC (Role-Based)**: Permissions are assigned to Roles (Admin, Editor, Viewer). Users get Roles. Simple and effective for 90% of apps.
2.  **ABAC (Attribute-Based)**: Decisions based on attributes (e.g., "User is in HR" AND "Document is marked Confidential" AND "Request is from the Office HQ"). Very powerful but complex.
3.  **ACL (Access Control List)**: Specifically listing permissions for every object (e.g., "User Bob can Read File 5").

## 5. Real-World Example
**Google Docs Sharing**:
- This is a masterclass in authorization. You can be an "Owner," "Editor," "Commenter," or "Viewer." These permissions are specific to **one document** (ACL), but different users have different global permissions in the Google Workspace (RBAC).

## 6. Code Example
Implementing a Middleware for permission check:
```python
# Authorization Decorator
def require_permission(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            user_id = current_user.id
            # Querying the Permissions Service or checking JWT claims
            if not permissions_service.has_perm(user_id, permission):
                return {"error": "Forbidden"}, 403 # Important: 403, NOT 401
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/admin/delete-item')
@require_permission('DELETE_ITEMS')
def delete_item():
    return "Item Deleted"
```

## 7. Design Variations & Alternatives
- **Scopes (Social Logins)**: When you log into an app with Google, it asks for "Scopes" like `profile` or `email`. This is a form of delegated authorization.
- **Centralized vs Decentralized**: Does every microservice check permissions itself, or do they all call a central "Policy Engine"?

## 8. Pros and Cons
### Pros
- **Security**: Prevents horizontal and vertical privilege escalation.
- **Granularity**: Allows fine-tuning of user experiences.

### Cons
- **Maintenance**: Managing thousands of permission rules across hundreds of services can become a "Permission Hell."
- **Latency**: If every API call needs to check a database for permissions, it slows down the system.

## 9. Failure Scenarios & What Can Go Wrong
- **IDOR (Insecure Direct Object Reference)**: A user changes the URL from `/orders/123` to `/orders/124` and they can see someone else's order because the code checks if they are *logged in* but not if they *own* the order.
- **Privilege Escalation**: A user finding a way to grant themselves the "Admin" role.
- **403 vs 404 leakage**: Sometimes, you return a `404 Not Found` for unauthorized items so that hackers don't even know the item exists (Security through obscurity).

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High-scale systems often "Bake" permissions into the **JWT** (e.g., `groups: ["admins"]`) so that services don't need to call a DB to check basic roles.
- **Reliability**: If your permission database is down, nobody can do anything in the entire system.

## 11. When NOT to Use This
- **Content only for "Logged In" users**: If the only rule is "Everyone who is logged in can see everything," you don't need complex RBAC; just an Authentication check is enough.

## 12. Interview Perspective
- **What interviewers expect**: Clear understanding of **RBAC vs ABAC**, the difference between **AuthN and AuthZ**, and how to prevent **IDOR**.
- **Common question**: "How would you design a permission system for a large company with 50,000 employees?"

## 13. Connections to Other Topics
- **JWT**: Often used to carry authorization claims.
- **Policy as Code (OPA - Open Policy Agent)**: The modern way to handle distributed authorization.

## 14. Visual Aids
- **The AuthZ Flow**: User -> Token (with Scopes) -> Policy Engine -> Yes/No.
- **RBAC Hierarchy Chart**: SuperAdmin > Admin > Moderator > User.

## 15. Summary (TL;DR)
Authorization is the process of **enforcing rules** on what an identity can do. Never assume "Authenticated" means "Authorized."

## 16. Further Reading & Resources
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [Open Policy Agent (OPA) Documentation](https://www.openpolicyagent.org/)
- [Zanzibar: Google’s Consistent, Global Authorization System (Paper)](https://research.google/pubs/pub48190/)
- [RBAC vs ABAC Comparison](https://www.okta.com/blog/2020/09/rbac-vs-abac/)
