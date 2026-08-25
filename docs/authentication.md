# Authentication and Authorization

The Fruit Store API uses JSON Web Tokens (JWTs) to authenticate users and role-based access control to protect administrator-only endpoints.

This guide explains how to obtain a token, authenticate requests, understand user roles, and troubleshoot common authentication and authorization errors.

## Authentication vs. Authorization

Authentication and authorization serve different purposes in the API.

- **Authentication** verifies the identity of the user making a request.
- **Authorization** determines whether an authenticated user has permission to access a specific resource or perform an action.

For example, a valid JWT can authenticate a user. However, that user still cannot access product-management endpoints unless their account has the `admin` role.

## User Roles

The API supports two user roles:

| Role | Access |
| --- | --- |
| `user` | View account information, purchase products, and view their own invoices |
| `admin` | Access user functionality, manage products, and view all invoices |

New accounts created through `/register` are assigned the `user` role.

## Obtain a JWT

A JWT is returned when you register a new user or log in to an existing account.

### Register a new user

Send a request to:

```http
POST /register
Content-Type: application/json
```

Example request body:

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

A successful registration returns a token:

```json
{
  "message": "User created successfully.",
  "token": "<JWT_TOKEN>",
  "user": {
    "id": 1,
    "username": "example_user",
    "role": "user"
  }
}
```

**Status:** `201 Created`

The token can be used immediately to access endpoints available to authenticated users.

### Log in to an existing account

Send the username and password to:

```http
POST /login
Content-Type: application/json
```

Example request body:

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

A successful login returns:

```json
{
  "message": "Login successful.",
  "token": "<JWT_TOKEN>",
  "user": {
    "id": 1,
    "username": "example_user",
    "role": "user"
  }
}
```

**Status:** `200 OK`

## Authenticate a Request

Protected endpoints expect the JWT in the HTTP `Authorization` header.

Use the Bearer authentication scheme:

```http
Authorization: Bearer <JWT_TOKEN>
```

For example:

```http
GET /me
Authorization: Bearer <JWT_TOKEN>
```

The API extracts the token from the header and validates it before allowing access to the endpoint.

A successful request to `/me` returns information about the authenticated user:

```json
{
  "id": 1,
  "username": "example_user",
  "role": "user"
}
```

**Status:** `200 OK`

## Authentication Flow

A typical authentication flow looks like this:

```text
Register or log in
        ↓
API returns JWT
        ↓
Client stores token
        ↓
Client sends protected request
        ↓
Authorization: Bearer <JWT_TOKEN>
        ↓
API validates token
        ↓
API identifies the user
        ↓
Protected endpoint executes
```

When validating a request, the API uses the token to identify the user associated with it.

If the token is valid and the user still exists, the application makes the authenticated user's ID, username, and role available to the protected endpoint.

## Protected Endpoints

The following endpoints require authentication:

| Method | Endpoint | Required access |
| --- | --- | --- |
| `GET` | `/me` | Authenticated user |
| `GET` | `/products` | Admin |
| `GET` | `/products/{product_id}` | Admin |
| `POST` | `/products` | Admin |
| `PUT` | `/products/{product_id}` | Admin |
| `DELETE` | `/products/{product_id}` | Admin |
| `POST` | `/purchase` | Authenticated user |
| `GET` | `/invoices` | Authenticated user |

The `/register`, `/login`, and `/liveness` endpoints do not require authentication.

## Administrator Authorization

Product-management endpoints require the authenticated user to have the `admin` role.

For example:

```http
GET /products
Authorization: Bearer <JWT_TOKEN>
```

The API first authenticates the request by validating the JWT.

It then checks the authenticated user's role.

```text
Request
   ↓
Is there a valid JWT?
   ↓
  Yes
   ↓
Identify authenticated user
   ↓
Is role = admin?
   ↓
Yes ─────────────→ Allow request
   ↓
 No
   ↓
403 Forbidden
```

If the user is authenticated but does not have administrator privileges, the API returns:

```json
{
  "error": "Administrator access is required."
}
```

**Status:** `403 Forbidden`

## Invoice Authorization

The `/invoices` endpoint demonstrates role-based authorization without restricting the endpoint entirely to administrators.

Both regular users and administrators can access:

```http
GET /invoices
```

However, the response depends on the authenticated user's role:

- A regular user receives only invoices associated with their own account.
- An administrator receives all invoices.

This allows the same endpoint to provide different levels of data access based on the user's permissions.

## Authentication Errors

### Missing Token

If a protected request does not include an authentication token:

```json
{
  "error": "Authentication token is required."
}
```

**Status:** `401 Unauthorized`

Make sure the request includes:

```http
Authorization: Bearer <JWT_TOKEN>
```

### Invalid Authorization Header

The API expects the authorization header to contain exactly two parts:

```text
Bearer <JWT_TOKEN>
```

A missing or incorrectly formatted Bearer token is treated as a missing authentication token and returns:

```json
{
  "error": "Authentication token is required."
}
```

**Status:** `401 Unauthorized`

### Expired Token

If the JWT has expired:

```json
{
  "error": "The authentication token has expired."
}
```

**Status:** `401 Unauthorized`

Log in again to obtain a new token.

### Invalid Token

If the token cannot be validated:

```json
{
  "error": "The authentication token is invalid."
}
```

**Status:** `401 Unauthorized`

Obtain a valid token through `/register` or `/login` and retry the request.

### User No Longer Exists

A JWT can be valid while the user associated with it no longer exists in the database.

In this case, the API returns:

```json
{
  "error": "The authenticated user no longer exists."
}
```

**Status:** `401 Unauthorized`

### Insufficient Permissions

If a valid authenticated user attempts to access an administrator-only endpoint:

```json
{
  "error": "Administrator access is required."
}
```

**Status:** `403 Forbidden`

This is an authorization failure rather than an authentication failure.

## 401 vs. 403

The API uses `401 Unauthorized` and `403 Forbidden` for different situations.

| Status | Meaning | Example |
| --- | --- | --- |
| `401 Unauthorized` | The request cannot be authenticated | Missing, expired, or invalid JWT |
| `403 Forbidden` | The user is authenticated but does not have permission | A `user` attempts to access an admin-only endpoint |

A simple way to think about the difference is:

```text
401 → The API cannot authenticate this request.
403 → The API authenticated the user, but the user cannot perform this action.
```

## Security Considerations

Treat JWTs as sensitive credentials.

- Do not include real JWTs in documentation, screenshots, or public repositories.
- Use placeholders such as `<JWT_TOKEN>` in examples.
- Do not commit passwords, tokens, or other credentials to version control.
- Obtain a new token when an existing token expires.

## Related Documentation

- [Getting Started](getting-started.md)
- [API Reference](api-reference.md)
