## What is GraphQL API in Microsoft Fabric?

GraphQL in Microsoft Fabric is a way to expose your tables and views (usually stored in a Lakehouse) as an API endpoint. You can use it to fetch data using GraphQL queries instead of SQL. The main use case is building dashboards, prototypes, or lightweight apps that need flexible data access without writing complex backends.

---

## Why Use It?

* You can query only what you need — no overfetching.
* It’s good for building React or Power Apps frontends.
* You don’t need to write a REST API — it’s auto-generated.

---

## What You Need First

1. A Lakehouse with data — tables like `amazon_uk_products`.
2. Access to the **GraphQL Explorer** inside Fabric (still in preview as of 2024).
3. A working dataset — clean column names, no null keys, etc.

---

## What It Looks Like

Example query:

```graphql
query {
  amazon_uk_products(first: 5) {
    items {
      asin
      title
      stars
    }
  }
}
```

It fetches 5 items with the fields you ask for.

You can filter:

```graphql
query {
  amazon_uk_products(
    filter: {stars: {gte: 4.5}}
  ) {
    items {
      title
      stars
    }
  }
}
```

You can sort, paginate, and even use `OR`, `AND`, and null checks.

---

## When You Shouldn’t Use It

* If your logic is very complex (joins, custom aggregations).
* If your data isn’t cleaned or modeled properly.
* If you need full control over the API (GraphQL in Fabric is still opinionated).

---
