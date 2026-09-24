# API Users - Object-Level Authorization

## Endpoint

GET /api/Users/1

## Authentication

Authenticated as:

- User ID: 25
- Role: customer
- Account: vapt.test@example.com

## Expected Behavior

A customer should only be able to access resources they are authorized to view.

## Observed Behavior

The customer account successfully requested another user's record:

HTTP/1.1 200 OK

The requested user was:

- User ID: 1
- Role: admin
- Email: admin@juice-sh.op

## Evidence

Raw response:

enumeration/api-user-1-authenticated.txt

## Impact

An authenticated low-privileged user can access another user's account information by changing the user ID in the API endpoint.

The collection endpoint also exposed multiple user records and roles.

## Reproduction

1. Authenticate as a customer.
2. Obtain the legitimate JWT.
3. Send:

   GET /api/Users/1

   with:

   Authorization: Bearer <customer JWT>

4. Observe HTTP 200 OK and the administrator's user record.

## Remediation

Enforce server-side object-level authorization for every user-resource request.

Verify that the authenticated user's identity and privileges permit access to the requested user ID before returning the resource.
