# Fruit Store API Documentation

A developer documentation portfolio project for a REST API built with Python, Flask, PostgreSQL, SQLAlchemy, JWT authentication, and Redis caching.

This documentation set demonstrates how a backend application can be documented for developers, from initial setup and authentication to endpoint reference, caching behavior, and troubleshooting.

> **Portfolio project:** The Fruit Store API was developed as part of my software development training. I created this documentation set to demonstrate my technical writing and developer documentation skills.

## Documentation

### [Getting Started](docs/getting-started.md)

Set up the API, configure the required environment variables, start the development server, register a user, and make your first authenticated request.

### [API Reference](docs/api-reference.md)

Reference documentation for all available endpoints, including authentication requirements, parameters, request bodies, response examples, status codes, and data models.

### [Authentication and Authorization](docs/authentication.md)

Learn how JWT authentication, Bearer tokens, user roles, protected endpoints, and role-based authorization work in the API.

### [Caching with Redis](docs/caching.md)

Learn how the API uses Redis to cache product data, handle cache hits and misses, and invalidate stale data after write operations.

### [Troubleshooting](docs/troubleshooting.md)

Resolve common authentication, authorization, product, purchase, and request-validation errors.

## API Overview

The Fruit Store API supports:

- User registration and login
- JWT-based authentication
- Role-based authorization for users and administrators
- Product CRUD operations
- Product purchases
- Invoice generation and retrieval
- PostgreSQL data persistence
- Redis caching and targeted cache invalidation

## Technology Stack

| Technology | Purpose |
| --- | --- |
| Python | Application programming language |
| Flask | REST API framework |
| PostgreSQL | Relational database |
| SQLAlchemy | Object-relational mapping (ORM) |
| PyJWT | JWT authentication |
| Redis | Product-data caching |

## API at a Glance

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/liveness` | Public | Check API status |
| `POST` | `/register` | Public | Register a user |
| `POST` | `/login` | Public | Authenticate a user |
| `GET` | `/me` | Authenticated | Get the current user |
| `GET` | `/products` | Admin | List products |
| `GET` | `/products/{product_id}` | Admin | Get a product |
| `POST` | `/products` | Admin | Create a product |
| `PUT` | `/products/{product_id}` | Admin | Update a product |
| `DELETE` | `/products/{product_id}` | Admin | Delete a product |
| `POST` | `/purchase` | Authenticated | Purchase a product |
| `GET` | `/invoices` | Authenticated | List invoices |

For complete request and response details, see the [API Reference](docs/api-reference.md).

## Example

Authenticate an existing user:

```http
POST /login
Content-Type: application/json
```

```json
{
  "username": "example_user",
  "password": "example_password"
}
```

Use the JWT returned by the API to access protected endpoints:

```http
GET /me
Authorization: Bearer <JWT_TOKEN>
```

## Caching Strategy

Product GET requests use Redis as a caching layer.

```text
Request
   ↓
Check Redis
   ↓
Cache hit? ── Yes ──→ Return cached data
   │
   No
   ↓
Query PostgreSQL
   ↓
Store result in Redis
   ↓
Return data
```

The API uses targeted cache invalidation when product data changes through create, update, delete, or purchase operations.

See [Caching with Redis](docs/caching.md) for details.

## What This Project Demonstrates

This documentation portfolio demonstrates experience with:

- Developer-focused technical writing
- REST API documentation
- Quickstart and onboarding documentation
- Reference documentation
- Request and response examples
- HTTP methods and status codes
- Authentication and authorization documentation
- Technical concept documentation
- Error messages and troubleshooting content
- Relational data models
- Caching concepts and invalidation strategies
- Reading application code to document implemented behavior

## About the Project

The underlying Fruit Store API was developed during my software development training to apply backend development concepts in a practical project.

The application combines REST APIs, HTTP, Flask, relational databases, ORM-based data access, authentication and authorization, CRUD operations, purchasing and invoicing, and Redis caching.

This repository focuses specifically on **developer documentation for that implementation**.

## About the Author

**Kristal Pastor**

Technical Writer · Content Designer · Software Development Student

I combine extensive professional writing and content experience with hands-on software development training to create clear, accurate, and user-centered technical content.

My current technical experience includes Python, Flask, REST APIs, HTTP, SQL, PostgreSQL, SQLAlchemy, JWT authentication, role-based authorization, and Redis caching.
