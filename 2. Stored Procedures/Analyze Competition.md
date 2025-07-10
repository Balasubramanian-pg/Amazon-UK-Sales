### Stored Procedure 2: `AnalyzeCompetition`

This procedure is a powerful business intelligence tool designed to analyze the competitive landscape for a product or an entire category. It operates in two distinct modes, making it highly versatile for market research.

---

### 1. Purpose

The main goal is to provide a **competitive analysis** from two different perspectives:
1.  **Product-Specific:** "Given my product (by its ASIN), who are my direct competitors, and how do we compare on price and quality?"
2.  **Category-General:** "What does the overall market look like in a specific category? Who are the top players?"

---

### 2. Parameters (The Controls)

*   `@target_asin VARCHAR(20) = NULL`:
    *   **What it does:** The Amazon Standard Identification Number (ASIN) of the product you want to analyze.
    *   **Default (`NULL`):** This parameter is optional. If you provide an ASIN, the procedure runs in "Product-Specific" mode. If you leave it `NULL`, it runs in "Category-General" mode.

*   `@target_category VARCHAR(100) = NULL`:
    *   **What it does:** The name of the category you want to analyze.
    *   **Default (`NULL`):** This is used when `@target_asin` is `NULL`.

*   `@price_tolerance DECIMAL(10,2) = 10.0`:
    *   **What it does:** Defines a price range (e.g., ±£10) to classify competitors. This is only used in "Product-Specific" mode.
    *   **Default (`10.0`):** Products priced within £10 of the target product are considered "Direct Competitors."

---

### 3. Core Logic Breakdown (How it Works)

The procedure is structured with a clear setup phase followed by a single, powerful analysis query.

#### a. Setup Phase: Determining the Mode

The first part of the procedure uses an `IF/ELSE` block to figure out which mode to operate in.

```sql
DECLARE @target_price DECIMAL(10,2);
DECLARE @target_stars DECIMAL(3,2);
DECLARE @category VARCHAR(100);

IF @target_asin IS NOT NULL
BEGIN
    -- Product-Specific Mode
    SELECT @target_price = price, @target_stars = stars, @category = categoryName
    FROM amazon_uk_products
    WHERE asin = @target_asin;
END
ELSE
BEGIN
    -- Category-General Mode
    SET @category = @target_category;
END
```

*   **If `@target_asin` is provided:** The procedure looks up the `price`, `stars`, and `categoryName` of that specific product and stores them in local variables (`@target_price`, `@target_stars`, `@category`). These values become the **benchmark** for comparison.
*   **If `@target_asin` is `NULL`:** The procedure simply uses the `@target_category` provided by the user. The `@target_price` and `@target_stars` variables remain `NULL`.

#### b. Analysis Phase: The Main `SELECT` Query

This query performs the actual competitive analysis. Let's break down its key components.

*   **Filtering (`WHERE` clause):**
    ```sql
    WHERE categoryName = @category
      AND (@target_asin IS NULL OR asin != @target_asin)
    ```
    1.  `categoryName = @category`: This is crucial. It ensures the analysis is always confined to the correct category (either the one from the target ASIN or the one specified directly).
    2.  `(@target_asin IS NULL OR asin != @target_asin)`: This condition smartly excludes the target product itself from the list of its own competitors. If you are in category mode (`@target_asin` is `NULL`), this part of the condition is always true and doesn't filter anything extra.

*   **Calculated Columns (The Insights):**

    1.  **`price_difference`**
        ```sql
        ABS(price - COALESCE(@target_price, price)) as price_difference
        ```
        This calculates the price gap between a competitor and the target. The `COALESCE` function makes it work in both modes:
        *   **Product Mode:** `COALESCE` returns `@target_price`, so it calculates `ABS(competitor_price - target_price)`.
        *   **Category Mode:** `@target_price` is `NULL`, so `COALESCE` returns the competitor's own `price`. The calculation becomes `ABS(price - price)`, which is always `0`. This effectively neutralizes the column in this mode.

    2.  **`competitive_position`**
        ```sql
        CASE
            WHEN @target_asin IS NOT NULL THEN
                CASE
                    WHEN price < @target_price - @price_tolerance THEN 'Cheaper Alternative'
                    WHEN price > @target_price + @price_tolerance THEN 'Premium Alternative'
                    ELSE 'Direct Competitor'
                END
            ELSE 'Market Player'
        END as competitive_position
        ```
        This logic is mode-aware:
        *   **Product Mode:** It uses the `@price_tolerance` to classify each competitor relative to the target product's price.
        *   **Category Mode:** It simply labels every product as a 'Market Player', since there's no single product to compare against.

    3.  **`market_rank`**
        ```sql
        RANK() OVER (ORDER BY stars DESC, reviews DESC) as market_rank
        ```
        This uses a powerful window function. It calculates the rank of each product *within its category*. The ranking is based first on star rating (higher is better) and then on the number of reviews as a tie-breaker. This shows a product's standing in the market hierarchy.

*   **Sorting (`ORDER BY` clause):**
    ```sql
    ORDER BY
        CASE WHEN @target_asin IS NOT NULL THEN ABS(price - @target_price) ELSE stars END,
        stars DESC;
    ```
    The sorting logic is also mode-aware:
    *   **Product Mode:** It sorts primarily by `ABS(price - @target_price)`. This brings the products with the closest prices to the top, making it easy to see the most direct competitors. The secondary sort is by `stars DESC`.
    *   **Category Mode:** It sorts primarily by `stars` in descending order. This shows the highest-rated products in the category first.

---

### Step-by-Step Execution Flow (Product Mode Example)

Let's say you execute `EXEC AnalyzeCompetition @target_asin = 'B09B96TG33'`.

1.  **Setup:** The procedure finds product `B09B96TG33`. Let's assume it costs £99, has 4.5 stars, and is in the 'Headphones' category. It stores these values in `@target_price`, `@target_stars`, and `@category`.
2.  **Filter:** It queries the `amazon_uk_products` table for all products where `categoryName = 'Headphones'` and `asin != 'B09B96TG33'`.
3.  **Calculate & Rank:** For each of these competing headphones, it calculates:
    *   The `price_difference` (e.g., a £85 headphone has a difference of 14).
    *   The `competitive_position` (that £85 headphone is a 'Cheaper Alternative').
    *   The `market_rank` based on stars and reviews.
4.  **Sort:** It sorts the entire resulting list, putting the headphones with prices closest to £99 at the top.
5.  **Return:** It returns the sorted list of competitors with all the calculated analysis columns.
