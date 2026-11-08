# Dashboard Wireframe & Visual Specification Guide

**Project:** Enterprise Omnichannel Retail: Commercial Performance & Margin Leakage  
**Resolution:** 16:9 Standard (`1280 x 720` px or `1920 x 1080` px)  
**Theme:** Corporate Slate (Clean Light Mode)

---

## 1. Design System & Color Palette

| Color Role | HEX Code | Purpose |
| :--- | :--- | :--- |
| **Primary Navy** | `#0F172A` | Report titles, primary table headers |
| **Canvas Background** | `#F8FAFC` | Page canvas background |
| **Card Container** | `#FFFFFF` | Visual card background (8px border radius, 1px border) |
| **Card Border** | `#E2E8F0` | Container outlines |
| **Primary Accent** | `#2563EB` | Sales bars, trend lines, primary metrics |
| **Target Met / Positive** | `#10B981` | Positive variance, YoY revenue growth |
| **Warning / Pacing** | `#F59E0B` | Near target threshold (90-99%) |
| **Negative / Margin Alert**| `#EF4444` | Deficit variance, margin erosion |
| **Muted Text / Axis** | `#64748B` | Subtitles, data labels, axis labels |

---

## 2. Page Architecture: 3-Tier Hierarchy

The dashboard is structured into three dedicated views based on stakeholder needs:

1. **Executive Overview:** High-level pacing, variance to budget, top-line revenue, and profit.
2. **Margin Leakage Diagnostic:** Root-cause analysis of discount depth and profitability erosion.
3. **Regional Operations:** Tactical view for territory and store managers with drill-through capability.

---

## 3. Wireframe Specifications

### Page 1: Executive Commercial Performance
**Target Audience:** C-Suite (CEO, CFO, Commercial Director)

```text
+-------------------------------------------------------------------------------------------------------------+
| COMMERCIAL PERFORMANCE OVERVIEW                                        Slicers: [Year] [Channel] [Category] |
+-------------------------------------------------------------------------------------------------------------+
| [Net Realized Sales]   | [Gross Profit]         | [Gross Margin %]      | [Budget Variance $]   | [% Target]|
| $24,812,400            | $9,215,300             | 37.1%                 | +$1,050,400           | 104.4%    |
| (+11.5% YoY)           | (+9.4% YoY)            | (-70 bps YoY)         | (Status: Green)       |           |
+------------------------+------------------------+-----------------------+-----------------------+-----------+
| Visual 1: Monthly Net Sales vs Budget (Clustered Column + Line)   | Visual 2: Category Target Attainment    |
| - Columns: Net Sales (Realized)                                   | - Horizontal Bar: Budget vs Actual      |
| - Dotted Line: Budget Revenue Quota                               | - Conditional Color: Met / Deficit      |
| - Solid Line: Prior Year Net Sales (SPLY)                         |                                         |
+-------------------------------------------------------------------+-----------------------------------------+
| Visual 3: Channel Share of Revenue (Donut / Treemap)              | Visual 4: Performance Matrix Summary    |
| - Offline Retail (38%)                                            | - Hierarchy: Category > SubCategory     |
| - Marketplace (31%)                                               | - Columns: Net Sales, Gross Profit,     |
| - Online DTC (21%)                                                |   Margin %, YoY Change %, Target Var    |
| - B2B Wholesale (10%)                                             |                                         |
+-------------------------------------------------------------------------------------------------------------+
```

---

### Page 2: Margin Leakage & Pricing Diagnostic
**Target Audience:** Head of Commercial, Pricing Analysts, Category Managers

```text
+-------------------------------------------------------------------------------------------------------------+
| MARGIN LEAKAGE & DISCOUNT IMPACT                                       Slicers: [Category] [Channel]        |
+-------------------------------------------------------------------------------------------------------------+
| [Gross Value]          | [Total Discounts]      | [Realized Margin]     | [Foregone Gross Profit]           |
| $29,280,000            | $4,467,600 (15.3%)     | 37.1% (vs 37.8% PY)   | -$4,467,600                       |
+-------------------------------------------------------------------------------------------------------------+
| Visual 1: Revenue to Profit Bridge (Waterfall Chart)              | Visual 2: Discount Elasticity Scatter   |
| - Starts: Gross Sales                                             | - X-axis: Average Discount Rate (%)     |
| - Less: Promotional Discounts Given                               | - Y-axis: Realized Gross Margin (%)     |
| - Subtotal: Net Realized Sales                                    | - Bubble Size: Net Sales Volume         |
| - Less: Total Procurement Cost (COGS)                             | - Highlight: Marketplace Electronics    |
| - Ends: Net Gross Profit                                          |                                         |
+-------------------------------------------------------------------+-----------------------------------------+
| Visual 3: Channel Margin Erosion Heatmap (Matrix)                 | Visual 4: 80/20 SKU Profitability Curve |
| - Rows: Channel                                                   | - Ranked Bar: Top 30 SKUs               |
| - Columns: 2024 Margin % | 2025 Margin % | Delta (bps)            | - Line: Cumulative Profit %             |
+-------------------------------------------------------------------------------------------------------------+
```

---

### Page 3: Regional & Store Operations
**Target Audience:** Regional Sales Directors, Store Managers

```text
+-------------------------------------------------------------------------------------------------------------+
| REGIONAL STORE PERFORMANCE                                             Slicers: [Region] [Province]         |
+-------------------------------------------------------------------------------------------------------------+
| Visual 1: Territory Net Sales Map (Bubble Map)                    | Visual 2: Store Ranking Leaderboard     |
| - Bubble Size: Net Sales Volume                                   | - Ranked Bar: Store Sales vs Quota      |
| - Color: Achievement %                                            | - Conditional formatting by quota hit   |
+-------------------------------------------------------------------+-----------------------------------------+
| Visual 3: Store Performance Matrix (Drill-Through Target)                                                   |
| - Store Name | Channel | Orders | Net Sales | Gross Margin % | Target Var $ | % Quota Met                  |
| - User Action: Right click any store -> Drill-through to 'Store SKU Detail'                                 |
+-------------------------------------------------------------------------------------------------------------+
```

---

## 4. Power BI Canvas Configuration

1. **Page Size:** 16:9 (`1280 x 720` or `1920 x 1080` px).
2. **Canvas Background:** `#F8FAFC`, Transparency: `0%`.
3. **Card Container Formatting:**
   - Background: `#FFFFFF` (100% opaque).
   - Visual Border: Enabled, Color `#E2E8F0`, Corner radius `8px`.
   - Shadow: Subtle bottom-right shadow, Opacity `5%`.
4. **Typography:**
   - Primary Headers: Segoe UI Semibold (14pt - 18pt).
   - Card Callout Values: Segoe UI Bold (20pt - 24pt).
   - Table / Matrix Body: Segoe UI (9pt - 10pt).
