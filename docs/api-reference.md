# Fruit Store API Reference

The Fruit Store API provides endpoints for user authentication, product management, purchases, and invoices.

## Base URL

```text
http://localhost:5001
```

## Authentication

Protected endpoints use JWT Bearer authentication.

Include the token returned by `/register` or `/login` in the `Authorization` header:

```http
Authorization: Bearer <JWT_TOKEN>
```

The API supports two access levels:

* **User:** Can access account information, purchase products, and view their own invoices.
* **Admin:** Can access user functionality, manage products, and view all invoices.

---

# Health

## Check API status

```http
GET /liveness
```

Checks whether the Fruit Store API is running.

**Authentication:** Not required

### Successful response

```json
{
  "status": "ok",
  "message": "Fruit Store API is running."
}
```

**Status:** `200 OK`

---

# Authentication

## Register a user

```http
POST /register
```

Creates a new user account. New accounts are assigned the `user` role.

**Authentication:** Not required

### Request body

| Field      | Type   | Required | Description                  |
| ---------- | ------ | -------- | ---------------------------- |
| `username` | string | Yes      | Username for the new account |
| `password` | string | Yes      | Password for the new account |

### Example request

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

### Successful response

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

### Possible errors

* `400 Bad Request` — The request body is missing or the credentials are invalid.
* `409 Conflict` — The username already exists.
* `500 Internal Server Error` — An unexpected server error occurred.

---

## Log in

```http
POST /login
```

Authenticates an existing user and returns a JWT token.

**Authentication:** Not required

### Request body

| Field      | Type   | Required | Description      |
| ---------- | ------ | -------- | ---------------- |
| `username` | string | Yes      | Account username |
| `password` | string | Yes      | Account password |

### Example request

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

### Successful response

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

### Possible errors

* `400 Bad Request` — The request body is missing or the credentials are invalid.
* `401 Unauthorized` — The username or password is incorrect.
* `500 Internal Server Error` — An unexpected server error occurred.

---

# Users

## Get the current user

```http
GET /me
```

Returns information about the currently authenticated user.

**Authentication:** Required

### Example request

```http
GET /me
Authorization: Bearer <JWT_TOKEN>
```

### Successful response

```json
{
  "id": 1,
  "username": "example_user",
  "role": "user"
}
```

**Status:** `200 OK`

### Possible errors

* `401 Unauthorized` — The authentication token is missing, expired, invalid, or belongs to a user that no longer exists.
* `500 Internal Server Error` — An unexpected server error occurred during authentication.

---

# Products

Product-management endpoints require administrator access.

## List products

```http
GET /products
```

Returns all products in the store.

**Authentication:** Admin required

The endpoint checks Redis before querying the database.

The response includes a `source` field that indicates whether the data came from the cache or the database.

### Example request

```http
GET /products
Authorization: Bearer <JWT_TOKEN>
```

### Successful response

```json
{
  "products": [
    {
      "id": 1,
      "name": "Apple",
      "price": 2.5,
      "entry_date": "2026-08-01",
      "quantity": 50
    }
  ],
  "source": "database"
}
```

The `source` value can be:

* `database`
* `cache`

**Status:** `200 OK`

### Possible errors

* `401 Unauthorized` — Authentication is required or the token is invalid.
* `403 Forbidden` — Administrator access is required.
* `500 Internal Server Error` — An unexpected server error occurred.

---

## Get a product

```http
GET /products/{product_id}
```

Returns a specific product by ID.

**Authentication:** Admin required

The endpoint checks Redis for the requested product before querying the database.

### Path parameters

| Parameter    | Type    | Required | Description              |
| ------------ | ------- | -------- | ------------------------ |
| `product_id` | integer | Yes      | Unique ID of the product |

### Example request

```http
GET /products/1
Authorization: Bearer <JWT_TOKEN>
```

### Successful response

```json
{
  "product": {
    "id": 1,
    "name": "Apple",
    "price": 2.5,
    "entry_date": "2026-08-01",
    "quantity": 50
  },
  "source": "database"
}
```

**Status:** `200 OK`

### Possible errors

* `401 Unauthorized` — Authentication is required or the token is invalid.
* `403 Forbidden` — Administrator access is required.
* `404 Not Found` — The product does not exist.
* `500 Internal Server Error` — An unexpected server error occurred.

---

## Create a product

```http
POST /products
```

Creates a new product.

**Authentication:** Admin required

### Request body

| Field        | Type    | Required | Description                               |
| ------------ | ------- | -------- | ----------------------------------------- |
| `name`       | string  | Yes      | Product name                              |
| `price`      | number  | Yes      | Product price. Must be greater than zero. |
| `entry_date` | string  | Yes      | Product entry date in `YYYY-MM-DD` format |
| `quantity`   | integer | Yes      | Available quantity. Cannot be negative.   |

### Example request

```json
{
  "name": "Mango",
  "price": 3.5,
  "entry_date": "2026-08-20",
  "quantity": 25
}
```

### Successful response

```json
{
  "message": "Product created successfully.",
  "product": {
    "id": 2,
    "name": "Mango",
    "price": 3.5,
    "entry_date": "2026-08-20",
    "quantity": 25
  }
}
```

**Status:** `201 Created`

Creating a product invalidates the cached product list so that future list requests receive current data.

### Possible errors

* `400 Bad Request` — The request body or product data is invalid.
* `401 Unauthorized` — Authentication is required or the token is invalid.
* `403 Forbidden` — Administrator access is required.
* `500 Internal Server Error` — An unexpected server error occurred.

---

## Update a product

```http
PUT /products/{product_id}
```

Updates an existing product.

**Authentication:** Admin required

This endpoint expects all required product fields and performs a complete update rather than a partial update.

### Path parameters

| Parameter    | Type    | Required | Description              |
| ------------ | ------- | -------- | ------------------------ |
| `product_id` | integer | Yes      | Unique ID of the product |

### Request body

| Field        | Type    | Required | Description                               |
| ------------ | ------- | -------- | ----------------------------------------- |
| `name`       | string  | Yes      | Product name                              |
| `price`      | number  | Yes      | Product price. Must be greater than zero. |
| `entry_date` | string  | Yes      | Product entry date in `YYYY-MM-DD` format |
| `quantity`   | integer | Yes      | Available quantity. Cannot be negative.   |

### Example request

```json
{
  "name": "Mango",
  "price": 3.75,
  "entry_date": "2026-08-20",
  "quantity": 30
}
```

### Successful response

```json
{
  "message": "Product updated successfully.",
  "product": {
    "id": 2,
    "name": "Mango",
    "price": 3.75,
    "entry_date": "2026-08-20",
    "quantity": 30
  }
}
```

**Status:** `200 OK`

Updating a product invalidates both the cached product and the cached product list.

### Possible errors

* `400 Bad Request` — The request body or product data is invalid.
* `401 Unauthorized` — Authentication is required or the token is invalid.
* `403 Forbidden` — Administrator access is required.
* `404 Not Found` — The product does not exist.
* `500 Internal Server Error` — An unexpected server error occurred.

---

## Delete a product

```http
DELETE /products/{product_id}
```

Deletes an existing product.

**Authentication:** Admin required

### Path parameters

| Parameter    | Type    | Required | Description              |
| ------------ | ------- | -------- | ------------------------ |
| `product_id` | integer | Yes      | Unique ID of the product |

### Example request

```http
DELETE /products/2
Authorization: Bearer <JWT_TOKEN>
```

### Successful response

```json
{
  "message": "Product deleted successfully."
}
```

**Status:** `200 OK`

Deleting a product invalidates both the cached product and the cached product list.

### Possible errors

* `401 Unauthorized` — Authentication is required or the token is invalid.
* `403 Forbidden` — Administrator access is required.
* `404 Not Found` — The product does not exist.
* `409 Conflict` — The product cannot be deleted because it is included in an invoice.
* `500 Internal Server Error` — An unexpected server error occurred.

---

# Purchases

## Purchase a product

```http
POST /purchase
```

Purchases a specified quantity of a product for the authenticated user.

A successful purchase creates an invoice and updates the product inventory.

**Authentication:** Required

### Request body

| Field        | Type    | Required | Description                   |
| ------------ | ------- | -------- | ----------------------------- |
| `product_id` | integer | Yes      | ID of the product to purchase |
| `quantity`   | integer | Yes      | Number of units to purchase   |

Both values must be integers greater than zero.

### Example request

```json
{
  "product_id": 2,
  "quantity": 2
}
```

### Successful response

```json
{
  "message": "Purchase completed successfully.",
  "invoice": {
    "id": 10,
    "user_id": 1,
    "created_at": "2026-08-24T18:30:00+00:00",
    "total": 7.0,
    "items": [
      {
        "id": 21,
        "product_id": 2,
        "product_name": "Mango",
        "quantity": 2,
        "unit_price": 3.5,
        "subtotal": 7.0
      }
    ]
  }
}
```

**Status:** `201 Created`

Because a purchase changes inventory, the corresponding product cache and the product-list cache are invalidated.

### Possible errors

* `400 Bad Request` — The request body or purchase data is invalid.
* `401 Unauthorized` — Authentication is required or the token is invalid.
* `404 Not Found` — The requested product does not exist.
* `409 Conflict` — There is insufficient stock to complete the purchase.
* `500 Internal Server Error` — An unexpected server error occurred.

---

# Invoices

## List invoices

```http
GET /invoices
```

Returns invoice information based on the authenticated user's role.

**Authentication:** Required

* Regular users receive only their own invoices.
* Administrators receive all invoices.

### Example request

```http
GET /invoices
Authorization: Bearer <JWT_TOKEN>
```

### Successful response

```json
{
  "invoices": [
    {
      "id": 10,
      "user_id": 1,
      "created_at": "2026-08-24T18:30:00+00:00",
      "total": 7.0,
      "items": [
        {
          "id": 21,
          "product_id": 2,
          "product_name": "Mango",
          "quantity": 2,
          "unit_price": 3.5,
          "subtotal": 7.0
        }
      ]
    }
  ]
}
```

**Status:** `200 OK`

### Possible errors

* `401 Unauthorized` — Authentication is required or the token is invalid.
* `500 Internal Server Error` — An unexpected server error occurred.

---

# Authentication and Authorization Errors

Protected endpoints can return the following authentication and authorization errors.

## Missing authentication token

```json
{
  "error": "Authentication token is required."
}
```

**Status:** `401 Unauthorized`

---

## Expired authentication token

```json
{
  "error": "The authentication token has expired."
}
```

**Status:** `401 Unauthorized`

---

## Invalid authentication token

```json
{
  "error": "The authentication token is invalid."
}
```

**Status:** `401 Unauthorized`

---

## Authenticated user no longer exists

```json
{
  "error": "The authenticated user no longer exists."
}
```

**Status:** `401 Unauthorized`

---

## Insufficient permissions

```json
{
  "error": "Administrator access is required."
}
```

**Status:** `403 Forbidden`

---

# Data Models

## User

Represents a registered user account.

| Field      | Type    | Description                          |
| ---------- | ------- | ------------------------------------ |
| `id`       | integer | Unique user identifier               |
| `username` | string  | User account name                    |
| `role`     | string  | User role, such as `user` or `admin` |

### Example

```json
{
  "id": 1,
  "username": "example_user",
  "role": "user"
}
```

---

## Product

Represents a product available in the store.

| Field        | Type    | Description                       |
| ------------ | ------- | --------------------------------- |
| `id`         | integer | Unique product identifier         |
| `name`       | string  | Product name                      |
| `price`      | number  | Product price                     |
| `entry_date` | string  | Entry date in `YYYY-MM-DD` format |
| `quantity`   | integer | Available inventory quantity      |

### Example

```json
{
  "id": 2,
  "name": "Mango",
  "price": 3.5,
  "entry_date": "2026-08-20",
  "quantity": 25
}
```

---

## Invoice

Represents a completed purchase.

| Field        | Type    | Description                                       |
| ------------ | ------- | ------------------------------------------------- |
| `id`         | integer | Unique invoice identifier                         |
| `user_id`    | integer | ID of the user associated with the invoice        |
| `created_at` | string  | Invoice creation date and time in ISO 8601 format |
| `total`      | number  | Total invoice amount                              |
| `items`      | array   | List of items included in the invoice             |

### Example

```json
{
  "id": 10,
  "user_id": 1,
  "created_at": "2026-08-24T18:30:00+00:00",
  "total": 7.0,
  "items": [
    {
      "id": 21,
      "product_id": 2,
      "product_name": "Mango",
      "quantity": 2,
      "unit_price": 3.5,
      "subtotal": 7.0
    }
  ]
}
```

---

## Invoice Item

Represents one product included in an invoice.

| Field          | Type    | Description                            |
| -------------- | ------- | -------------------------------------- |
| `id`           | integer | Unique invoice-item identifier         |
| `product_id`   | integer | ID of the purchased product            |
| `product_name` | string  | Name of the purchased product          |
| `quantity`     | integer | Quantity purchased                     |
| `unit_price`   | number  | Price per unit at the time of purchase |
| `subtotal`     | number  | Total amount for this invoice item     |

### Example

```json
{
  "id": 21,
  "product_id": 2,
  "product_name": "Mango",
  "quantity": 2,
  "unit_price": 3.5,
  "subtotal": 7.0
}
```

---

# Endpoint Summary

| Method   | Endpoint                 | Access        | Description          |
| -------- | ------------------------ | ------------- | -------------------- |
| `GET`    | `/liveness`              | Public        | Check API status     |
| `POST`   | `/register`              | Public        | Register a user      |
| `POST`   | `/login`                 | Public        | Authenticate a user  |
| `GET`    | `/me`                    | Authenticated | Get the current user |
| `GET`    | `/products`              | Admin         | List all products    |
| `GET`    | `/products/{product_id}` | Admin         | Get a product        |
| `POST`   | `/products`              | Admin         | Create a product     |
| `PUT`    | `/products/{product_id}` | Admin         | Update a product     |
| `DELETE` | `/products/{product_id}` | Admin         | Delete a product     |
| `POST`   | `/purchase`              | Authenticated | Purchase a product   |
| `GET`    | `/invoices`              | Authenticated | List invoices        |
