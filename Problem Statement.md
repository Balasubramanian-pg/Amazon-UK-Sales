# Amazon Product Performance Analysis System - Scenario Document

## Business Context
The organization lacks comprehensive visibility into product performance metrics across its Amazon UK marketplace. Current systems provide basic sales data but fail to deliver actionable insights about:
- How products truly perform relative to competitors
- Optimal pricing strategies for different market positions
- Customer satisfaction beyond simple star ratings
- Emerging product trends and momentum
- Strategic market positioning opportunities

## Solution Overview
We've developed five analytical SQL functions that transform raw product data into strategic business intelligence. These functions work together to provide a 360-degree view of product performance.

## Function Scenarios

### 1. CalculateProductScore
**Purpose**: Provides a comprehensive performance metric combining multiple factors into a single comparable score.

**Business Use Case**:
- Quickly compare products across different categories
- Identify top performers for inventory prioritization
- Evaluate new product launch performance

**Input Parameters**:
- Star rating (1-5 scale)
- Review count
- Product price
- Best seller status
- Category benchmarks (optional)

**Output**: Standardized score (0-100 scale)

**Example Application**:
```sql
SELECT product_name, dbo.CalculateProductScore(stars, reviews, price, isBestSeller) AS performance_score
FROM amazon_products
ORDER BY performance_score DESC;
```

### 2. GetMarketPosition
**Purpose**: Classifies products into strategic market positions based on price and quality relative to competitors.

**Business Use Case**:
- Identify premium vs. value products
- Detect overpriced products needing adjustment
- Position new products strategically

**Input Parameters**:
- Product ASIN
- Category
- Price
- Star rating

**Output**: Market position classification (e.g., "Value Leader", "Premium Positioned")

**Example Application**:
```sql
SELECT product_name, price, stars, 
       dbo.GetMarketPosition(asin, category, price, stars) AS market_position
FROM amazon_products;
```

### 3. CalculateSatisfactionIndex
**Purpose**: Measures true customer satisfaction by considering rating distribution and review patterns.

**Business Use Case**:
- Identify products with genuine customer satisfaction vs. inflated ratings
- Detect potential quality issues before they impact sales
- Benchmark satisfaction against category norms

**Input Parameters**:
- Star rating
- Review count
- Category benchmarks (optional)

**Output**: Satisfaction index (40-110 scale)

**Example Application**:
```sql
SELECT product_name, 
       dbo.CalculateSatisfactionIndex(stars, reviews) AS satisfaction_index
FROM amazon_products
WHERE satisfaction_index < 70;  -- Identify low satisfaction products
```

### 4. GetOptimalPriceRange
**Purpose**: Recommends price ranges based on market position strategy and target rating.

**Business Use Case**:
- Price new product launches
- Adjust prices for seasonal promotions
- Optimize pricing for different market segments

**Input Parameters**:
- Category
- Target rating
- Positioning strategy ('BUDGET', 'COMPETITIVE', 'PREMIUM')

**Output**: Recommended price range string

**Example Application**:
```sql
SELECT product_name, price,
       dbo.GetOptimalPriceRange(category, stars, 'COMPETITIVE') AS recommended_range
FROM amazon_products;
```

### 5. CalculateTrendMomentum
**Purpose**: Identifies products gaining or losing market momentum.

**Business Use Case**:
- Identify emerging bestsellers before they peak
- Detect declining products for potential discounts
- Allocate marketing resources to high-momentum products

**Input Parameters**:
- Product ASIN
- Star rating
- Review count
- Price
- Best seller status
- Category

**Output**: Momentum score (20-150 scale)

**Example Application**:
```sql
SELECT product_name,
       dbo.CalculateTrendMomentum(asin, stars, reviews, price, isBestSeller, category) AS momentum
FROM amazon_products
ORDER BY momentum DESC
LIMIT 20;  -- Top trending products
```

## Implementation Benefits
1. **Standardized Metrics**: Consistent evaluation across all products
2. **Actionable Insights**: Clear recommendations for pricing and positioning
3. **Proactive Management**: Early identification of trends and issues
4. **Data-Driven Decisions**: Reduced reliance on intuition for product strategy
5. **Competitive Advantage**: Better understanding of market position relative to competitors

## Integration Recommendations
- Incorporate these functions into daily performance dashboards
- Use in automated alerting systems for significant changes
- Apply to new product launch planning processes
- Integrate with inventory management systems for stock optimization
