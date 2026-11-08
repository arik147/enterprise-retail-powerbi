# DAX Measures Library

**Project:** Enterprise Omnichannel Retail: Commercial Performance & Margin Leakage  
**Target Table:** `_Measures`  

> **Implementation Note:**  
> In Power BI, create a disconnected table named `_Measures` via **Home > Enter Data**. Place all measures here to keep calculations cleanly separated from fact tables.

---

## 1. Core Commercial Metrics

### 1.1 Total Orders Placed
```dax
Total Orders = 
DISTINCTCOUNT(Fact_Sales[SalesOrderID])
```
*Format: `#,##0`*

### 1.2 Delivered Orders
```dax
Delivered Orders = 
CALCULATE(
    [Total Orders],
    Fact_Sales[OrderStatus] = "Delivered"
)
```
*Format: `#,##0`*

### 1.3 Return Rate (%)
```dax
Return Rate % = 
VAR ReturnedOrders = 
    CALCULATE([Total Orders], Fact_Sales[OrderStatus] = "Returned")
RETURN
DIVIDE(ReturnedOrders, [Total Orders], 0)
```
*Format: `0.0%`*

### 1.4 Cancellation Rate (%)
```dax
Cancellation Rate % = 
VAR CancelledOrders = 
    CALCULATE([Total Orders], Fact_Sales[OrderStatus] = "Cancelled")
RETURN
DIVIDE(CancelledOrders, [Total Orders], 0)
```
*Format: `0.0%`*

### 1.5 Gross Sales (Before Discounts)
```dax
Gross Sales = 
CALCULATE(
    SUM(Fact_Sales[GrossSales]),
    Fact_Sales[OrderStatus] <> "Cancelled"
)
```
*Format: `$#,##0`*

### 1.6 Total Discounts Given
```dax
Total Discounts = 
CALCULATE(
    SUM(Fact_Sales[DiscountAmount]),
    Fact_Sales[OrderStatus] = "Delivered"
)
```
*Format: `$#,##0`*

### 1.7 Realized Net Sales (Core Revenue)
```dax
Total Net Sales = 
CALCULATE(
    SUM(Fact_Sales[NetSales]),
    Fact_Sales[OrderStatus] = "Delivered"
)
```
*Format: `$#,##0`*

### 1.8 Total Cost of Goods Sold (COGS)
```dax
Total COGS = 
CALCULATE(
    SUM(Fact_Sales[TotalCost]),
    Fact_Sales[OrderStatus] = "Delivered"
)
```
*Format: `$#,##0`*

### 1.9 Gross Profit
```dax
Total Gross Profit = 
[Total Net Sales] - [Total COGS]
```
*Format: `$#,##0`*

### 1.10 Gross Profit Margin (%)
```dax
Gross Profit Margin % = 
DIVIDE([Total Gross Profit], [Total Net Sales], 0)
```
*Format: `0.0%`*

### 1.11 Average Order Value (AOV)
```dax
Average Order Value = 
DIVIDE([Total Net Sales], [Delivered Orders], 0)
```
*Format: `$#,##0.00`*

---

## 2. Time Intelligence (YoY & YTD)

### 2.1 Net Sales Prior Year (SPLY)
```dax
Net Sales SPLY = 
CALCULATE(
    [Total Net Sales],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```
*Format: `$#,##0`*

### 2.2 Sales YoY Absolute Change ($)
```dax
Sales YoY Growth $ = 
VAR CurrentSales = [Total Net Sales]
VAR PriorSales = [Net Sales SPLY]
RETURN
IF(
    ISBLANK(PriorSales), 
    BLANK(), 
    CurrentSales - PriorSales
)
```
*Format: `+$#,##0;-$#,##0;$0`*

### 2.3 Sales YoY Growth Rate (%)
```dax
Sales YoY Growth % = 
VAR CurrentSales = [Total Net Sales]
VAR PriorSales = [Net Sales SPLY]
RETURN
DIVIDE(CurrentSales - PriorSales, PriorSales, 0)
```
*Format: `+0.0%;-0.0%;0.0%`*

### 2.4 Net Sales YTD
```dax
Net Sales YTD = 
TOTALYTD(
    [Total Net Sales],
    Dim_Date[Date]
)
```
*Format: `$#,##0`*

### 2.5 Gross Profit YoY Growth Rate (%)
```dax
Gross Profit SPLY = 
CALCULATE(
    [Total Gross Profit],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```
```dax
Gross Profit YoY Growth % = 
DIVIDE([Total Gross Profit] - [Gross Profit SPLY], [Gross Profit SPLY], 0)
```
*Format: `+0.0%;-0.0%;0.0%`*

---

## 3. Budget & Variance Analysis

### 3.1 Budget Revenue Target
```dax
Budget Revenue = 
SUM(Fact_Budget[TargetRevenue])
```
*Format: `$#,##0`*

### 3.2 Budget Gross Profit Target
```dax
Budget Gross Profit = 
SUM(Fact_Budget[TargetGrossProfit])
```
*Format: `$#,##0`*

### 3.3 Revenue Variance ($)
```dax
Revenue Variance $ = 
[Total Net Sales] - [Budget Revenue]
```
*Format: `+$#,##0;-$#,##0;$0`*

### 3.4 Revenue Target Achievement (%)
```dax
Revenue Target Achievement % = 
DIVIDE([Total Net Sales], [Budget Revenue], 0)
```
*Format: `0.0%`*

### 3.5 Gross Profit Variance ($)
```dax
Gross Profit Variance $ = 
[Total Gross Profit] - [Budget Gross Profit]
```
*Format: `+$#,##0;-$#,##0;$0`*

### 3.6 Profit Target Achievement (%)
```dax
Profit Target Achievement % = 
DIVIDE([Total Gross Profit], [Budget Gross Profit], 0)
```
*Format: `0.0%`*

### 3.7 KPI Target Status Color (Hex Code)
```dax
KPI Status Revenue Color = 
VAR Ratio = [Revenue Target Achievement %]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(Ratio), "#94A3B8",        -- Gray
    Ratio >= 1.00, "#10B981",          -- Green (Met / Exceeded)
    Ratio >= 0.90, "#F59E0B",          -- Amber (Close to Target)
    "#EF4444"                          -- Red (Deficit)
)
```

---

## 4. Margin Leakage & Pricing Diagnostics

### 4.1 Average Realized Discount Rate (%)
```dax
Average Discount % = 
DIVIDE([Total Discounts], [Gross Sales], 0)
```
*Format: `0.0%`*

### 4.2 Gross Margin YoY Change (Basis Points)
```dax
Margin YoY Delta (bps) = 
VAR CurrentMargin = [Gross Profit Margin %]
VAR PriorMargin = CALCULATE([Gross Profit Margin %], SAMEPERIODLASTYEAR(Dim_Date[Date]))
RETURN
IF(
    ISBLANK(PriorMargin),
    BLANK(),
    (CurrentMargin - PriorMargin) * 10000
)
```
*Format: `+0 bps;-0 bps;0 bps`*

---

## 5. Pareto Concentration Analysis

### 5.1 SKU Sales Rank
```dax
Product Sales Rank = 
RANKX(
    ALLSELECTED(Dim_Product[ProductName]),
    [Total Net Sales],
    ,
    DESC,
    DENSE
)
```

### 5.2 Cumulative Sales Contribution (%)
```dax
Cumulative Sales % = 
VAR CurrentSales = [Total Net Sales]
VAR TotalSalesAll = CALCULATE([Total Net Sales], ALLSELECTED(Dim_Product))
VAR CurrentRank = [Product Sales Rank]
VAR CumulativeSales = 
    CALCULATE(
        [Total Net Sales],
        FILTER(
            ALLSELECTED(Dim_Product[ProductName]),
            [Product Sales Rank] <= CurrentRank
        )
    )
RETURN
DIVIDE(CumulativeSales, TotalSalesAll, 0)
```
*Format: `0.0%`*
