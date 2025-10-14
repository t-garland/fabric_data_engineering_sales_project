# DAX Measures

This document contains all DAX measures used in the sales semantic model. Measures are stored in a separate _measures table outside the star schema for easy maintainability and access. These measures are available for downstream analytics consumption.

## Revenue Metrics

### Total Revenue
Sum of all sales revenue across transactions.

```dax
Total Revenue = 
// Sum of all sales revenue
SUM(fact_sales_gold[line_total])
```

### Total Quantity Sold
Total units sold across all transactions.

```dax
Total Quantity Sold = 
// Total units sold across all transactions
SUM(fact_sales_gold[quantity])
```

### Average Order Value
Average revenue per order. Uses distinct count to avoid double-counting orders with multiple line items.

```dax
Average Order Value = 
// Average revenue per order
DIVIDE(
    [Total Revenue],
    DISTINCTCOUNT(fact_sales_gold[order_id]),
    0
)
```

### Total Revenue PY
Prior year revenue for year-over-year comparisons.

```dax
Total Revenue PY = 
// Prior year revenue for YoY analysis
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR(dim_date_gold[date])
)
```

### Revenue vs Prior Year
Revenue difference compared to same period last year.

```dax
Revenue vs Prior Year = 
// Revenue variance compared to prior year
[Total Revenue] - [Total Revenue PY]
```

### YTD Revenue
Year-to-date revenue based on order date.

```dax
YTD Revenue = 
// Year-to-date revenue accumulation
TOTALYTD(
    [Total Revenue],
    dim_date_gold[date]
)
```

## Cost Metrics

### Total Cost
Cost of goods sold, calculated by multiplying quantity sold by unit cost from the product dimension.

```dax
Total Cost = 
// Calculate total cost based on quantity sold and product unit price
SUMX(
    fact_sales_gold,
    fact_sales_gold[quantity] * RELATED(dim_product_gold[unit_price])
)
```

## Profitability Metrics

### Total Profit
Net profit calculated as revenue minus cost of goods sold. Uses SUMX to compute profit at the line item level before aggregation.

```dax
Total Profit = 
// Calculate total profit (revenue minus cost of goods sold)
SUMX(
    fact_sales_gold,
    fact_sales_gold[line_total] - (fact_sales_gold[quantity] * RELATED(dim_product_gold[unit_price]))
)
```

### Profit Margin %
Profit as a percentage of revenue. Includes zero-handling to prevent division errors.

```dax
Profit Margin % = 
// Profit margin as a percentage of revenue
DIVIDE(
    [Total Profit],
    [Total Revenue],
    0
) * 100
```

### Total Profit PY
Prior year profit for year-over-year comparisons.

```dax
Total Profit PY = 
// Prior year profit for YoY analysis
CALCULATE(
    [Total Profit],
    SAMEPERIODLASTYEAR(dim_date_gold[date])
)
```

### YoY Profit
Year-over-year profit change in absolute dollars.

```dax
YoY Profit = 
// Year-over-year profit variance
[Total Profit] - [Total Profit PY]
```

### YoY Profit %
Year-over-year profit change as a percentage.

```dax
YoY Profit % = 
// Year-over-year profit growth percentage
DIVIDE(
    [YoY Profit],
    [Total Profit PY],
    0
) * 100
```

## Customer Metrics

### Top 10 Customers by Revenue
Returns rank of customer by revenue, filtered to top 10. Returns BLANK for customers outside top 10.

```dax
Top 10 Customers by Revenue = 
// Rank customers by revenue, show only top 10
VAR CustomerRank = 
    RANKX(
        ALL(dim_customer_gold[customer_name]),
        [Total Revenue],
        ,
        DESC,
        DENSE
    )
RETURN
    IF(CustomerRank <= 10, CustomerRank, BLANK())
```

## Product Metrics

### Top 10 Products by Profit
Returns rank of product by profit, filtered to top 10. Returns BLANK for products outside top 10.

```dax
Top 10 Products by Profit = 
// Rank products by profit, show only top 10
VAR ProductRank = 
    RANKX(
        ALL(dim_product_gold[product_name]),
        [Total Profit],
        ,
        DESC,
        DENSE
    )
RETURN
    IF(ProductRank <= 10, ProductRank, BLANK())
```

## Notes

- All measures use the fact_sales_gold fact table with related dimension tables
- Time intelligence measures use dim_date_gold[date] as the date key
- DIVIDE function includes 0 as alternate result to handle division by zero scenarios
- SUMX measures iterate row-by-row for line-level calculations before aggregation
- Cost calculations assume dim_product_gold[unit_price] represents cost price, not selling price
- RANKX measures use DENSE ranking to handle ties appropriately
- Top N measures return BLANK() for items outside the top 10 to enable clean filtering in visuals
