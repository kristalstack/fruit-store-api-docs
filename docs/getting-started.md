# Getting Started

This guide walks you through setting up the Fruit Store API, configuring its dependencies, starting the development server, and making your first authenticated request.

## Prerequisites

Before running the API, make sure you have:

- Python installed
- PostgreSQL database access
- Redis access
- `pip` for installing Python dependencies

## Install the dependencies

Install the project's Python dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
```

The application uses Flask for the API, SQLAlchemy for database operations, PostgreSQL as the relational database, PyJWT for authentication, and Redis for caching.

## Configure environment variables

The project includes an `.env.example` file that defines the environment variables required by the application:

```env
DATABASE_URL=PLACEHOLDER

REDIS_HOST=PLACEHOLDER
REDIS_PORT=PLACEHOLDER
REDIS_USERNAME=PLACEHOLDER
REDIS_PASSWORD=PLACEHOLDER
```

Create a `.env` file and replace each placeholder with the appropriate value for your environment.

Do not commit your `.env` file or credentials to version control.

## Start the API

Run the application:

```bash
python app.py
```

The development server runs on port `5001`.

The examples in this documentation use:

```text
http://localhost:5001
```

as the base URL.

## Verify that the API is running

Send a GET request to the liveness endpoint:

```http
GET /liveness
```

A successful request returns:

```json
{
  "status": "ok",
  "message": "Fruit Store API is running."
}
```

**Status:** `200 OK`

## Register a user

Create a user by sending a POST request to:

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

A successful registration returns a JWT authentication token together with the newly created user.

Example:

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

Save the token returned by the API. You will need it to access protected endpoints.

## Authenticate an existing user

If you already have an account, send your credentials to:

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

A successful login returns a JWT token:

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

## Make your first authenticated request

Protected endpoints expect the JWT in the `Authorization` header using the Bearer authentication scheme:

```http
Authorization: Bearer <JWT_TOKEN>
```

For example, retrieve information about the currently authenticated user:

```http
GET /me
Authorization: Bearer <JWT_TOKEN>
```

A successful response looks like:

```json
{
  "id": 1,
  "username": "example_user",
  "role": "user"
}
```

**Status:** `200 OK`

## Authentication errors

If the request does not include an authentication token, the API returns:

```json
{
  "error": "Authentication token is required."
}
```

**Status:** `401 Unauthorized`

An expired or invalid token also returns `401 Unauthorized`.

Some endpoints, including product management endpoints, require administrator access. An authenticated user without the `admin` role receives:

```json
{
  "error": "Administrator access is required."
}
```

**Status:** `403 Forbidden`

## Next steps

Once the API is running and you are authenticated, you can:

- View your authenticated user information
- Purchase products
- View invoices
- Manage products with administrator access
- Explore cached product responses

See the API Reference for the complete list of available endpoints.
