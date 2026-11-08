# Data Dictionary & Modeling Schema

**Project:** Enterprise Omnichannel Retail: Commercial Performance & Margin Leakage  
**Platform:** Microsoft Power BI Desktop  
**Data Model:** Multi-Fact Star Schema  
**Granularity:**  
- `Fact_Sales`: Order line item (transaction level)
- `Fact_Budget`: Monthly commercial target (MonthKey + Category + Channel)

---

## 1. Schema Diagram (Star Schema)

```text
       +-----------------------+              +-----------------------+
       |       Dim_Date        |              |      Dim_Product      |
       +-----------------------+              +-----------------------+
       | PK: DateKey           |              | PK: ProductID         |
       | Date                  |              | ProductName           |
       | Year, Month, etc.     |              | Category, SubCategory |
       +-----------+-----------+              | UnitCost, Price       |
                   |                          +-----------+-----------+
         1:N (DateKey)                                    |
                   |                                      | 1:N (ProductID)
                   v                                      v
       +--------------------------------------------------------------+
       |                          Fact_Sales                          |
       +--------------------------------------------------------------+
       | PK: SalesOrderID                                             |
       | FK: DateKey -> Dim_Date[DateKey]                             |
       | FK: ProductID -> Dim_Product[ProductID]                       |
       | FK: StoreID -> Dim_Store_Region[StoreID]                     |
       | FK: CustomerID -> Dim_Customer[CustomerID]                   |
       | OrderStatus (Delivered, Returned, Cancelled)                 |
       | Quantity, UnitPrice, DiscountPct, DiscountAmount             |
       | GrossSales, NetSales, TotalCost, GrossProfit, MarginPct      |
       +--------------------------------------------------------------+
                   ^                                      ^
         1:N (StoreID)                        1:N (CustomerID)
                   |                                      |
       +-----------+-----------+              +-----------+-----------+
       |   Dim_Store_Region    |              |     Dim_Customer      |
       +-----------------------+              +-----------------------+
       | PK: StoreID           |              | PK: CustomerID        |
       | StoreName, Channel    |              | CustomerName, Segment |
       | City, Region, Manager |              | City, JoinDate        |
       +-----------------------+              +-----------------------+

                                  +
                                  |
            1:N (DateKey)         v
       +--------------------------------------------------------------+
       |                         Fact_Budget                          |
       +--------------------------------------------------------------+
       | Composite Key: MonthKey + Category + Channel                 |
       | TargetRevenue, TargetGrossProfit                             |
       +--------------------------------------------------------------+
```

---

## 2. Table Specifications

### A. Fact_Sales (Transactional Orders)
* **File:** `data/processed/Fact_Sales.csv` (43,170 records)
* **Description:** Records individual customer purchases across retail stores, online DTC, marketplaces, and B2B channels.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `SalesOrderID` | Text | Primary Key | Unique sales order identifier (e.g., `SO-2024-000001`). |
| `OrderDate` | Date | Attribute | Date when the order was placed (`YYYY-MM-DD`). |
| `DateKey` | Whole Number | Foreign Key | Integer date key linking to `Dim_Date[DateKey]` (`YYYYMMDD`). |
| `ProductID` | Text | Foreign Key | Product ID linking to `Dim_Product[ProductID]`. |
| `CustomerID` | Text | Foreign Key | Customer ID linking to `Dim_Customer[CustomerID]`. |
| `StoreID` | Text | Foreign Key | Store/channel ID linking to `Dim_Store_Region[StoreID]`. |
| `OrderStatus` | Text | Dimension | Order lifecycle status: `Delivered`, `Returned`, or `Cancelled`. |
| `PaymentMethod` | Text | Dimension | Payment channel (Credit Card, QRIS, Bank Transfer, Invoice). |
| `Quantity` | Whole Number | Metric | Number of units purchased. |
| `UnitPrice` | Decimal Number | Price | Standard retail unit price at sale time (USD). |
| `DiscountPct` | Decimal Number | Parameter | Applied promotion percentage (0.00 – 0.40). |
| `DiscountAmount` | Decimal Number | Metric | Nominal discount (`Quantity * UnitPrice * DiscountPct`). |
| `GrossSales` | Decimal Number | Metric | Revenue before discount (`Quantity * UnitPrice`). |
| `NetSales` | Decimal Number | Metric | Net transaction value (`GrossSales - DiscountAmount`). |
| `TotalCost` | Decimal Number | Metric | Total procurement cost / COGS (`Quantity * UnitCost`). |
| `GrossProfit` | Decimal Number | Metric | Profit contribution (`NetSales - TotalCost`). |
| `ProfitMarginPct` | Decimal Number | Metric | Margin percentage (`GrossProfit / NetSales * 100`). |

---

### B. Fact_Budget (Commercial Planning Targets)
* **File:** `data/processed/Fact_Budget.csv` (384 records)
* **Description:** Monthly commercial targets established for each product category and sales channel.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `MonthKey` | Date | Attribute | First calendar day of the budget month (`YYYY-MM-01`). |
| `DateKey` | Whole Number | Foreign Key | Integer date key linking to `Dim_Date[DateKey]`. |
| `Year` | Whole Number | Attribute | Budget Year (2024, 2025). |
| `MonthNum` | Whole Number | Attribute | Budget Month Number (1 – 12). |
| `Category` | Text | Dimension | Product Category (Electronics, Home & Living, Apparel, Beauty). |
| `Channel` | Text | Dimension | Channel name (Offline Retail, Online DTC, Marketplace, B2B). |
| `TargetRevenue` | Decimal Number | Metric | Monthly net sales target (USD). |
| `TargetGrossProfit` | Decimal Number | Metric | Monthly gross profit target (USD). |

---

### C. Dim_Product (Product Catalog)
* **File:** `data/processed/Dim_Product.csv` (31 SKUs)
* **Description:** Master product catalog containing hierarchies, procurement unit cost, and standard retail price.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `ProductID` | Text | Primary Key | Unique product SKU code (e.g., `PRD-E01`). |
| `ProductName` | Text | Dimension | Full commercial name of the product. |
| `Category` | Text | Hierarchy Level 1 | Major line (Electronics, Home & Living, Apparel, Beauty). |
| `SubCategory` | Text | Hierarchy Level 2 | Granular grouping (e.g., Laptops & Computers, Skincare). |
| `UnitCost` | Decimal Number | Baseline Cost | Cost of goods sold per unit. |
| `BaseUnitPrice` | Decimal Number | Baseline Price | Standard list selling price per unit. |
| `StandardMarginPct` | Decimal Number | Benchmark | Expected benchmark margin percentage. |

---

### D. Dim_Store_Region (Sales Channels & Geography)
* **File:** `data/processed/Dim_Store_Region.csv` (12 physical & digital outlets)
* **Description:** Outlets detailing physical retail stores, online webstores, marketplaces, and B2B hubs.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `StoreID` | Text | Primary Key | Unique outlet identifier (e.g., `STR-01`). |
| `StoreName` | Text | Dimension | Full outlet name. |
| `Channel` | Text | Hierarchy Level 1 | Omnichannel division (Offline Retail, Online DTC, Marketplace, B2B). |
| `City` | Text | Geography | Operating city. |
| `Province` | Text | Geography | Operating province. |
| `Region` | Text | Geographic Hub | Consolidated territory (Greater Jakarta, West Java, East Java, Sumatra). |
| `StoreManager` | Text | Operational | Accountable manager. |

---

### E. Dim_Customer (Customer Profiles)
* **File:** `data/processed/Dim_Customer.csv` (2,500 accounts)
* **Description:** Master customer registry including B2C consumers and B2B corporate entities.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `CustomerID` | Text | Primary Key | Unique customer ID (e.g., `CUST-00001`). |
| `CustomerName` | Text | Dimension | Customer individual name or corporate entity title. |
| `Segment` | Text | Classification | Customer classification (`Consumer`, `Small Business`, `Corporate`). |
| `City` | Text | Geography | Customer registered location. |
| `CustomerSinceDate`| Date | Cohort | Account creation date. |

---

### F. Dim_Date (Master Calendar)
* **File:** `data/processed/Dim_Date.csv` (730 calendar days: 2024–2025)
* **Description:** Standardized enterprise date table required for Time Intelligence DAX functions.

| Column Name | Data Type | Key / Role | Description |
| :--- | :--- | :--- | :--- |
| `DateKey` | Whole Number | Primary Key | Integer date key (`YYYYMMDD`). |
| `Date` | Date | Attribute | True Date format (`YYYY-MM-DD`). |
| `Year` | Whole Number | Hierarchy | Calendar Year (`2024`, `2025`). |
| `Quarter` | Text | Hierarchy | Quarter code (`Q1`, `Q2`, `Q3`, `Q4`). |
| `QuarterYear` | Text | Display | Consolidated label (e.g., `Q1 2024`). |
| `MonthNum` | Whole Number | Sort Order | Integer month (1–12). |
| `MonthName` | Text | Dimension | Abbreviated month (`Jan`, `Feb`, ...). |
| `MonthYear` | Text | Display | Display label (e.g., `Jan 2024`). |
| `YearMonth` | Whole Number | Sort Order | Integer sort column (`YYYYMM`, e.g., `202401`). |
| `DayOfMonth` | Whole Number | Attribute | Day number (1–31). |
| `DayOfWeek` | Text | Attribute | Day name (`Monday`, ...). |
| `DayOfWeekNum` | Whole Number | Sort Order | Day of week integer (1=Sunday, 2=Monday, etc.). |
| `IsWeekend` | Whole Number | Flag | `1` for Saturday/Sunday, else `0`. |
