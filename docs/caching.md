# Caching with Redis

The Fruit Store API uses Redis to cache product data and reduce unnecessary database queries.

This guide explains how caching works in the API, how cache hits and misses are handled, and when cached data is invalidated.

## Why the API Uses Caching

Without caching, every request for product information would require a database query.

For example:

```http
GET /products/1
```

Without a cache, each request follows this flow:

```text
Client
  ↓
Fruit Store API
  ↓
PostgreSQL
  ↓
Fruit Store API
  ↓
Client
```

If the same product is requested repeatedly, querying the database every time can perform unnecessary work.

Redis provides a temporary storage layer between the application and the database.

With caching, the API can reuse product data that has already been retrieved.

## What the API Caches

The API caches two types of product data:

- The complete product list returned by `GET /products`
- Individual products returned by `GET /products/{product_id}`

Caching is applied to GET operations because these endpoints retrieve data without modifying it.

## Cache Lookup Flow

When a client requests product data, the API checks Redis before querying PostgreSQL.

```text
GET request
     ↓
Check Redis
     ↓
Is the data cached?
   ↙             ↘
 Yes              No
  ↓                ↓
Return cached    Query database
data               ↓
                 Cache result
                    ↓
                 Return data
```

This is commonly known as a **cache-aside** pattern: the application checks the cache first and loads data from the database when the cached value is unavailable.

## Cache Hits

A **cache hit** occurs when the requested data is already available in Redis.

For example:

```http
GET /products/1
Authorization: Bearer <JWT_TOKEN>
```

If product `1` is already cached, the API returns the cached product without querying the database.

Example response:

```json
{
  "product": {
    "id": 1,
    "name": "Apple",
    "price": 2.5,
    "entry_date": "2026-08-01",
    "quantity": 50
  },
  "source": "cache"
}
```

The `source` field indicates that Redis supplied the data.

## Cache Misses

A **cache miss** occurs when the requested data is not available in Redis.

When this happens, the API:

1. Queries the database.
2. Converts the result into response data.
3. Stores the result in Redis.
4. Returns the data to the client.

For example, the first request for a product might return:

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

A later request for the same product may return:

```json
{
  "product": {
    "id": 1,
    "name": "Apple",
    "price": 2.5,
    "entry_date": "2026-08-01",
    "quantity": 50
  },
  "source": "cache"
}
```

The response data is the same, but the source indicates how the API retrieved it.

## Product List Caching

The complete product list is also cached.

When an administrator sends:

```http
GET /products
Authorization: Bearer <JWT_TOKEN>
```

the API first checks Redis for the cached product list.

If the list is available, the API returns:

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
  "source": "cache"
}
```

If the list is not cached, the API retrieves all products from PostgreSQL, stores the result in Redis, and returns:

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

## Why Cache Invalidation Is Necessary

Cached data can become outdated when the underlying database changes.

For example, imagine Redis contains:

```json
{
  "id": 1,
  "name": "Apple",
  "price": 2.5,
  "quantity": 50
}
```

An administrator then changes the quantity to `75`.

If the old cached value remains available, a subsequent GET request could still return:

```json
{
  "quantity": 50
}
```

even though PostgreSQL contains:

```json
{
  "quantity": 75
}
```

This is known as **stale data**.

To prevent this, the Fruit Store API invalidates affected cache entries when product data changes.

## Cache Invalidation Strategy

The API uses targeted cache invalidation based on the operation being performed.

| Operation | Cache invalidated |
| --- | --- |
| Create product | Product list |
| Update product | Individual product + product list |
| Delete product | Individual product + product list |
| Purchase product | Individual product + product list |

This prevents outdated product information from being returned after write operations.

## Creating a Product

When an administrator creates a product:

```http
POST /products
```

the cached product list becomes outdated because it does not contain the newly created product.

The API therefore invalidates the product-list cache.

```text
Create product
      ↓
Write to database
      ↓
Invalidate product-list cache
      ↓
Next GET /products
      ↓
Query database
      ↓
Cache updated product list
```

## Updating a Product

When a product is updated:

```http
PUT /products/{product_id}
```

two cached values may become stale:

- The individual product
- The complete product list

The API invalidates both.

```text
Update product
      ↓
Write new data to database
      ↓
Invalidate product cache
      +
Invalidate product-list cache
      ↓
Future GET requests load fresh data
```

## Deleting a Product

Deleting a product also affects both cache levels.

```http
DELETE /products/{product_id}
```

After the product is removed from the database, the API invalidates:

- The individual product cache
- The product-list cache

This prevents the deleted product from continuing to appear in cached responses.

## Purchases and Cache Invalidation

Purchases also require cache invalidation even though `/purchase` is not a product-management endpoint.

For example:

```http
POST /purchase
```

with:

```json
{
  "product_id": 1,
  "quantity": 2
}
```

changes the available inventory for product `1`.

If the product originally has:

```json
{
  "quantity": 50
}
```

a successful purchase of two units changes the inventory to:

```json
{
  "quantity": 48
}
```

Any cached version containing `50` is now stale.

For this reason, a successful purchase invalidates both the individual product cache and the product-list cache.

This ensures that subsequent GET requests retrieve the updated inventory from the database.

## Cache Invalidation Flow

A simplified write-and-invalidate flow looks like this:

```text
Write operation
      ↓
Update PostgreSQL
      ↓
Identify affected cache entries
      ↓
Delete affected Redis entries
      ↓
Next GET request
      ↓
Cache miss
      ↓
Query PostgreSQL
      ↓
Store fresh data in Redis
```

Instead of trying to update every cached representation directly, the application removes affected cached values and allows subsequent reads to repopulate them with current database data.

## Redis Configuration

The API uses environment variables to configure its Redis connection.

The `.env.example` file defines:

```env
REDIS_HOST=PLACEHOLDER
REDIS_PORT=PLACEHOLDER
REDIS_USERNAME=PLACEHOLDER
REDIS_PASSWORD=PLACEHOLDER
```

Create a `.env` file and replace the placeholders with the appropriate Redis connection values for your environment.

Do not commit Redis credentials to version control.

## Benefits of This Approach

Caching product data can provide several benefits:

- Reduces repeated database queries
- Improves response times for frequently requested data
- Reduces database workload
- Allows frequently accessed product data to be served from Redis

Targeted invalidation also helps maintain consistency between cached product data and the database.

## Trade-offs

Caching introduces additional complexity.

The application must determine:

- What data should be cached
- When cached data is no longer valid
- Which cache entries must be invalidated after a write
- How to prevent clients from receiving stale data

For the Fruit Store API, product data is cached because it can be read repeatedly, while invalidation occurs whenever an operation changes information represented by those cached values.

## Related Documentation

- [Getting Started](getting-started.md)
- [API Reference](api-reference.md)
- [Authentication and Authorization](authentication.md)
