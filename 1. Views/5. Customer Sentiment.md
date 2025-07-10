### View 5: `customer_sentiment_insights`

This view is an advanced analytical tool focused on understanding the overall customer happiness and product quality within each category. It moves beyond simple averages to provide a nuanced picture of customer sentiment.

---

### 1. Purpose

The primary purpose of this view is to measure and diagnose **customer sentiment** at a category level. It's designed to help business leaders, category managers, and marketers understand the overall health and customer perception of different markets.

It answers critical questions like:
*   Which categories consistently delight customers?
*   Which categories have a wide variance in product quality, making them "risky" for shoppers?
*   What is the distribution of high-quality vs. low-quality products in a specific market?
*   What percentage of products in a category meet a "high satisfaction" benchmark?

---

### 2. Core Logic Breakdown (How it Works)

The view aggregates all products by their category and calculates a suite of metrics that describe the sentiment landscape.

#### a. The `WHERE` and `GROUP BY` Clauses

*   **`WHERE stars > 0`:** A simple but important filter that ensures the analysis only includes products that have actually been rated by customers.
*   **`GROUP BY categoryName`:** This is the core of the view. Every calculation is performed for each unique category, creating a high-level summary row for each one.

#### b. The Calculated Columns (The Insights)

1.  **`avg_rating` & `avg_review_count`:**
    *   These are the basic health metrics, showing the average quality and popularity for the category.

2.  **`rating_volatility (STDEV(stars))`:**
    *   **What it does:** This is a key advanced metric. `STDEV` calculates the standard deviation of star ratings within the category.
    *   **Why it's useful:** It measures the **consistency** of product quality.
        *   A **low volatility** (e.g., 0.2) means most products in the category have ratings very close to the average. The customer experience is predictable and consistent.
        *   A **high volatility** (e.g., 0.9) means the category is a mix of 5-star hits and 1-star duds. The customer experience is a "gamble," and the average rating can be misleading.

3.  **Rating Distribution (`excellent_products`, `good_products`, etc.):**
    ```sql
    SUM(CASE WHEN stars >= 4.5 THEN 1 ELSE 0 END) AS excellent_products
    ```
    *   **What it does:** Instead of just an average, these columns break down the total product count into performance buckets. It counts how many products fall into the 'Excellent' (4.5+), 'Good' (4.0-4.4), 'Average' (3.0-3.9), and 'Poor' (<3.0) tiers.
    *   **Why it's useful:** This provides a much richer understanding than `avg_rating` alone. A category with an average rating of 3.5 could either have all its products rated around 3.5, or it could be sharply polarized with half at 5 stars and half at 2 stars. This distribution reveals the true picture.

4.  **`satisfaction_rate`:**
    ```sql
    ROUND(100.0 * SUM(CASE WHEN stars >= 4.0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS satisfaction_rate
    ```
    *   **What it does:** It calculates the percentage of products in a category that meet or exceed a 4.0-star rating.
    *   **Why it's useful:** This creates a clear, actionable Key Performance Indicator (KPI). It defines a "satisfied" customer experience and measures how well a category delivers on it. It's often more insightful than a simple average.

5.  **`sentiment_status`:**
    ```sql
    CASE WHEN AVG(stars) >= 4.3 THEN 'Highly Satisfied' ... END AS sentiment_status
    ```
    *   **What it does:** It translates the numerical `avg_rating` into a simple, human-readable label.
    *   **Why it's useful:** It allows for quick sorting and filtering to find the best or worst-performing categories without having to remember specific numerical thresholds.

---

### 3. How It's Used

This view is a one-stop-shop for understanding market health from the customer's perspective.

**Example Query 1: Find the 5 categories with the happiest customers.**
```sql
SELECT
    categoryName,
    sentiment_status,
    satisfaction_rate,
    avg_rating
FROM
    customer_sentiment_insights
ORDER BY
    satisfaction_rate DESC
LIMIT 5;
```

**Example Query 2: Find the 5 "riskiest" categories with the most inconsistent quality.**
```sql
SELECT
    categoryName,
    rating_volatility,
    avg_rating
FROM
    customer_sentiment_insights
ORDER BY
    rating_volatility DESC
LIMIT 5;
```
This query is perfect for identifying markets where a brand could enter and win by providing a consistently high-quality product.

**Example Query 3: Get a full health report for the "Headphones" category.**
```sql
SELECT
    excellent_products,
    good_products,
    average_products,
    poor_products
FROM
    customer_sentiment_insights
WHERE
    categoryName = 'Headphones';
```
This shows a category manager the exact distribution of product quality in their market, helping them understand their competitive positioning.
