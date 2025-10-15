# DAX Measures

This document contains all DAX measures used in the sales semantic model. Measures are stored in a separate _measures table outside the reporting model for easy maintainability and access. These measures are available for downstream analytics consumption.

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

## Notes

- All measures use the fact_sales_gold fact table with related dimension tables
- Time intelligence measures use dim_date_gold[date] as the date key
- DIVIDE function includes 0 as alternate result to handle division by zero scenarios
- SUMX measures iterate row-by-row for line-level calculations before aggregation
- Cost calculations assume dim_product_gold[unit_price] represents cost price, not selling price
- Top N filtering (customers, products) should be applied at the report layer using visual filters
