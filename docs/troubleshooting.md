# Troubleshooting

This guide covers common issues you may encounter when using the Fruit Store API and provides steps to resolve them.

For endpoint-specific request and response details, see the [API Reference](api-reference.md).

## Authentication Issues

### 401 — Authentication Token Is Required

**Response**

```json
{
  "error": "Authentication token is required."
}
```

**Possible cause**

The request was sent to a protected endpoint without a valid Bearer token in the `Authorization` header.

The API expects the header in this format:

```http
Authorization: Bearer <JWT_TOKEN>
```

**Resolution**

1. Register a new account through `/register` or log in through `/login`.
2. Copy the JWT returned in the response.
3. Add the token to the `Authorization` header.
4. Retry the request.

Example:

```http
GET /me
Authorization: Bearer <JWT_TOKEN>
```

If you already included a token, verify that the header contains exactly two parts: `Bearer` followed by the token.

---

### 401 — Authentication Token Has Expired

**Response**

```json
{
  "error": "The authentication token has expired."
}
```

**Cause**

The JWT was valid but has reached its expiration time.

**Resolution**

Log in again through:

```http
POST /login
```

Use the new JWT returned by the API for subsequent protected requests.

---

### 401 — Authentication Token Is Invalid

**Response**

```json
{
  "error": "The authentication token is invalid."
}
```

**Possible causes**

- The token is malformed.
- The token was modified.
- The value is not a valid JWT for this API.

**Resolution**

1. Log in again through `/login`.
2. Copy the new token exactly as returned.
3. Send it using the Bearer authentication scheme:

```http
Authorization: Bearer <JWT_TOKEN>
```

Do not include quotation marks around the token.

---

### 401 — Authenticated User No Longer Exists

**Response**

```json
{
  "error": "The authenticated user no longer exists."
}
```

**Cause**

The token identifies a user that can no longer be found in the database.

**Resolution**

Register a new account or authenticate using an existing account.

---

## Authorization Issues

### 403 — Administrator Access Is Required

**Response**

```json
{
  "error": "Administrator access is required."
}
```

**Cause**

The request is authenticated, but the current account does not have the `admin` role.

Product-management endpoints require administrator access.

These include:

```text
GET    /products
GET    /products/{product_id}
POST   /products
PUT    /products/{product_id}
DELETE /products/{product_id}
```

**Resolution**

Use credentials for an account with the `admin` role.

A valid JWT from a regular `user` account authenticates the request but does not grant administrator permissions.

For more information, see [Authentication and Authorization](authentication.md).

---

## Registration and Login Issues

### 400 — Username or Password Is Missing

Possible responses include:

```json
{
  "error": "A JSON body is required."
}
```

```json
{
  "error": "Username and password are required."
}
```

**Possible causes**

- The request does not contain a JSON body.
- `username` is missing.
- `password` is missing.

**Resolution**

Send both fields in the request body:

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

Make sure the request uses:

```http
Content-Type: application/json
```

---

### 400 — Invalid Credential Format

Possible responses include:

```json
{
  "error": "Username and password must be strings."
}
```

or:

```json
{
  "error": "Username cannot be empty."
}
```

**Resolution**

Make sure both credentials are strings and that the username contains a non-whitespace value.

Example:

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

---

### 401 — Invalid Username or Password

**Response**

```json
{
  "error": "Invalid username or password."
}
```

**Cause**

The credentials provided to `/login` could not be authenticated.

**Resolution**

Verify the username and password and retry the request.

---

### 409 — Username Already Exists

**Response**

```json
{
  "error": "Username already exists."
}
```

**Cause**

An account already exists with the username provided to `/register`.

**Resolution**

Choose a different username or use `/login` if the account already belongs to you.

---

## Product Issues

### 400 — Missing Product Fields

**Response**

```json
{
  "error": "Missing required fields.",
  "fields": [
    "price",
    "quantity"
  ]
}
```

The exact `fields` array depends on which fields are missing.

**Cause**

Product creation and update requests require all of the following fields:

- `name`
- `price`
- `entry_date`
- `quantity`

**Resolution**

Send a complete product object.

Example:

```json
{
  "name": "Mango",
  "price": 3.5,
  "entry_date": "2026-08-20",
  "quantity": 25
}
```

---

### 400 — Invalid Product Name

**Response**

```json
{
  "error": "Name must be a non-empty string."
}
```

**Resolution**

Provide a product name as a non-empty string.

---

### 400 — Invalid Price

**Response**

```json
{
  "error": "Price must be greater than zero."
}
```

**Cause**

The price cannot be converted to a valid positive value or is less than or equal to zero.

**Resolution**

Provide a numeric price greater than zero.

Example:

```json
{
  "price": 3.5
}
```

---

### 400 — Invalid Quantity

Possible responses include:

```json
{
  "error": "Quantity must be an integer."
}
```

or:

```json
{
  "error": "Quantity cannot be negative."
}
```

**Resolution**

Provide quantity as an integer greater than or equal to zero.

For example:

```json
{
  "quantity": 25
}
```

---

### 400 — Invalid Entry Date

**Response**

```json
{
  "error": "Entry date must use YYYY-MM-DD format."
}
```

**Resolution**

Use an ISO-style calendar date in `YYYY-MM-DD` format.

Example:

```json
{
  "entry_date": "2026-08-20"
}
```

---

### 404 — Product Not Found

**Response**

```json
{
  "error": "Product not found."
}
```

**Cause**

No product exists with the requested ID.

**Resolution**

Verify the `product_id` in the request and retry with an existing product.

---

### 409 — Product Cannot Be Deleted

**Response**

```json
{
  "error": "The product cannot be deleted because it is included in an invoice."
}
```

**Cause**

The product is referenced by existing invoice data and cannot be removed without violating the database relationship.

**Resolution**

Do not delete products that are already included in an invoice.

---

## Purchase Issues

### 400 — Missing Purchase Data

Possible responses include:

```json
{
  "error": "A JSON body is required."
}
```

or:

```json
{
  "error": "product_id and quantity are required."
}
```

**Resolution**

Provide both required fields:

```json
{
  "product_id": 1,
  "quantity": 2
}
```

---

### 400 — Invalid Product ID

Possible responses include:

```json
{
  "error": "product_id must be an integer."
}
```

or:

```json
{
  "error": "product_id must be greater than zero."
}
```

**Resolution**

Provide a positive integer as the product ID.

Example:

```json
{
  "product_id": 1
}
```

---

### 400 — Invalid Purchase Quantity

Possible responses include:

```json
{
  "error": "Quantity must be an integer."
}
```

or:

```json
{
  "error": "Quantity must be greater than zero."
}
```

Unlike product inventory, a purchase quantity cannot be zero.

**Resolution**

Provide a positive integer.

Example:

```json
{
  "quantity": 2
}
```

---

### 404 — Product Does Not Exist

A purchase returns `404 Not Found` when the requested product does not exist.

**Resolution**

Verify the `product_id` and retry the purchase with an existing product.

---

### 409 — Insufficient Stock

A purchase returns `409 Conflict` when there is not enough inventory to fulfill the requested quantity.

**Resolution**

Reduce the requested quantity or choose another product with sufficient inventory.

A successful purchase changes the product quantity and invalidates the affected Redis cache entries.

---

## Unexpected Server Errors

### 500 — Internal Server Error

Several endpoints can return:

```json
{
  "error": "An internal server error occurred."
}
```

**Possible causes**

A `500 Internal Server Error` indicates that the API encountered an unexpected problem while processing the request.

Depending on the operation, the problem may involve an application dependency such as the database.

**Resolution**

1. Confirm that the API is running.
2. Verify that the environment variables are configured correctly.
3. Verify that required services such as PostgreSQL and Redis are available.
4. Review the application logs for the underlying exception.

Do not expose internal exception details or credentials in public bug reports.

---

## Verify API Availability

If requests are not working as expected, first check whether the API is running:

```http
GET /liveness
```

A healthy API returns:

```json
{
  "status": "ok",
  "message": "Fruit Store API is running."
}
```

**Status:** `200 OK`

The local development server runs on:

```text
http://localhost:5001
```

---

## Quick Error Reference

| Status | Error | Common cause |
| --- | --- | --- |
| `400` | Bad Request | Missing or invalid request data |
| `401` | Unauthorized | Missing, expired, or invalid authentication |
| `403` | Forbidden | Authenticated user lacks required permissions |
| `404` | Not Found | Requested product does not exist |
| `409` | Conflict | Duplicate username, insufficient stock, or product referenced by an invoice |
| `500` | Internal Server Error | Unexpected server-side failure |

## Related Documentation

- [Getting Started](getting-started.md)
- [API Reference](api-reference.md)
- [Authentication and Authorization](authentication.md)
- [Caching with Redis](caching.md)
