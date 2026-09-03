# DAX reference

All measures, calculated columns, and the date table used in this dashboard.

## Date table

Create via `Modeling → New Table`, then `Mark as Date Table` on the `Date` column.

```dax
Date =
ADDCOLUMNS(
    CALENDAR(DATE(2014,1,1), DATE(2017,12,31)),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Month Number", MONTH([Date]),
    "Year-Month", FORMAT([Date], "YYYY-MM")
)
```

## Core measures

```dax
Total Sales = SUM(Sales[Sales])

Total Profit = SUM(Sales[Profit])

Profit Margin = DIVIDE([Total Profit], [Total Sales])

Total Orders = DISTINCTCOUNT(Sales[Order ID])

Total Quantity = SUM(Sales[Quantity])

Avg Order Value = DIVIDE([Total Sales], [Total Orders])

Avg Discount = AVERAGE(Sales[Discount])

Total Customers = DISTINCTCOUNT(Sales[Customer ID])
```

## Time intelligence

```dax
Sales YoY % =
VAR PriorYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN DIVIDE([Total Sales] - PriorYear, PriorYear)

Sales YTD = TOTALYTD([Total Sales], 'Date'[Date])

Profit YTD = TOTALYTD([Total Profit], 'Date'[Date])

Sales MoM % =
VAR PriorMonth = CALCULATE([Total Sales], PREVIOUSMONTH('Date'[Date]))
RETURN DIVIDE([Total Sales] - PriorMonth, PriorMonth)
```

## Product page

```dax
Avg Margin = DIVIDE([Total Profit], [Total Sales])

Loss-Making Subs =
COUNTROWS(
    FILTER(
        VALUES(Product[Sub-Category]),
        [Total Profit] < 0
    )
)

-- Helper for the "Top loss-makers" Top N filter (rank by biggest loss)
Loss Amount = -1 * [Total Profit]
```

## Customer page

```dax
Customer Count = DISTINCTCOUNT(Customer[Customer ID])

Sales per Customer = DIVIDE([Total Sales], [Total Customers])

Profit per Customer = DIVIDE([Total Profit], [Total Customers])

Avg Shipping Delay = AVERAGE(Sales[Shipping Days])
```

## Geography page

```dax
Top Region =
CALCULATE(
    SELECTEDVALUE(Customer[Region]),
    TOPN(1, VALUES(Customer[Region]), [Total Sales], DESC)
)

Top State =
CALCULATE(
    SELECTEDVALUE(Customer[State]),
    TOPN(1, VALUES(Customer[State]), [Total Sales], DESC)
)

Top Region Sales = MAXX(VALUES(Customer[Region]), [Total Sales])

States in Loss =
COUNTROWS(
    FILTER(
        VALUES(Customer[State]),
        [Total Profit] < 0
    )
)

City Count = DISTINCTCOUNT(Customer[City])
```

## Calculated columns

On the `Sales` table:

```dax
Shipping Days = DATEDIFF(Sales[Order Date], Sales[Ship Date], DAY)
```

On the `Customer` table:

```dax
Age Band =
SWITCH(
    TRUE(),
    Customer[Age] < 30, "18–29",
    Customer[Age] < 40, "30–39",
    Customer[Age] < 50, "40–49",
    Customer[Age] < 60, "50–59",
    "60+"
)

Age Band Sort =
SWITCH(
    TRUE(),
    Customer[Age] < 30, 1,
    Customer[Age] < 40, 2,
    Customer[Age] < 50, 3,
    Customer[Age] < 60, 4,
    5
)
```

> Set `Age Band` to **Sort by column → Age Band Sort** so the bands display in natural order.

## Notes

- All ratios use `DIVIDE()` to avoid divide-by-zero errors.
- `Profit Margin` / `Avg Margin` is a **weighted** margin (`Total Profit ÷ Total Sales`), not an average of row-level margins.
- Every measure respects slicers, so KPIs recalculate for the current filter selection.
- `Avg Shipping Delay` measures order → ship (fulfillment time), not delivery time.
