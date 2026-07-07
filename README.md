# DataCo Global Supply Chain Performance Dashboard

An interactive Power BI dashboard analyzing 180,000+ supply chain transactions across five global markets. Built to demonstrate enterprise-level business intelligence skills including data modeling, DAX measure development, Power Query transformations, and multi-page dashboard design.

> **Tools:** Power BI Desktop · Power Query (M Code) · DAX · Microsoft Azure Maps  
> **Dataset:** [DataCo Smart Supply Chain Dataset — Mendeley Data](https://data.mendeley.com/datasets/8gx2fvg2k6/5)  
> **Records:** 180,519 order line items · 41 fields · 5 global markets · 2015–2018

---

## Dashboard Pages

### Page 1 — Executive Summary
![Executive Summary](page%201_Executive_summary.png)

High-level KPI overview for senior leadership. Six headline cards surface the most critical business metrics at a glance, supported by a year-over-year sales trend line and market distribution donut chart.

**Key metrics displayed:**
- Total Sales: **$36.78M**
- Total Profit: **$3.97M**
- Total Orders: **66K**
- Profit Margin: **10.78%**
- On-Time Delivery Rate: **45.17%**
- Average Days to Ship: **3.50**

**Key insight:** Only 45% of orders are delivered on time — a systemic logistics problem visible across all markets and shipping modes.

---

### Page 2 — Regional & Market Performance
![Regional Performance](page%202_regional_performance.png)

Geographic and market-level breakdown of sales and profitability. Identifies where revenue is concentrated and where margin is strongest relative to volume.

**Key insights:**
- Europe leads in total sales ($10.87M, 29.56%) followed closely by LATAM ($10.28M, 27.94%)
- Fan Shop and Apparel are the dominant departments by revenue
- Southern Africa and Canada show the highest profit margins by region despite lower total sales volume
- The Sales vs. Profit bar chart reveals a consistent margin gap across all markets — high revenue does not translate proportionally into profit

---

### Page 3 — Product & Category Analysis
![Product Analysis](page%203_product_analysis.png)

Category and product-level profitability analysis. Identifies high-volume low-margin categories and products where discounting may be eroding profitability.

**Key insights:**
- Fishing is the top revenue category at $6.7M+ but warrants margin scrutiny
- "As Seen on TV" receives the highest average discount rate (~10%) — a red flag for margin compression
- The scatter chart reveals most categories cluster between $0–2M in sales with 0–10% margin, with only a few outliers driving significant volume
- At least one product (K7 Jolt Slope Rangefinder) generates negative profit — a direct flag for pricing or procurement review

---

### Page 4 — Delivery & Logistics Intelligence
![Logistics Intelligence](page%204_logistics_intelligence.png)
Operational deep-dive into delivery performance across shipping modes and global regions. Designed to support logistics optimization decisions.

**Key insights:**
- **54.83% of all orders are delivered late** — more than half of all shipments fail to meet scheduled delivery
- First Class shipping has the highest late delivery rate (~95%) — counterintuitive and operationally critical
- Standard Class performs best on delivery reliability despite being the lowest-cost option
- Central Africa and West Africa have the longest average days late, signaling regional logistics bottlenecks
- Average days late consistently exceeds average days to ship across all regions — systemic scheduling underestimation

---

## Data Model

```
DataCoSupplyChainDataset (Fact Table)
        │
        │  Many-to-One on [Order Date] → [Date]
        │
DateTable (Dimension Table)
    - Date, Year, Month Number, Month Name
    - Month Short, Quarter, Year-Month
    - Week Number, Weekday
```

---

## Power Query Transformations (M Code)

All data cleaning was scripted in Power Query Advanced Editor for reproducibility:

```powerquery
let
    Source = Csv.Document(File.Contents("...DataCoSupplyChainDataset.csv"),
        [Delimiter=",", Columns=53, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    RemovedColumns = Table.RemoveColumns(PromotedHeaders, {
        "Customer Email", "Customer Password", "Customer Street",
        "Customer Zipcode", "Product Description", "Product Image",
        "Product Status", "Order Item Cardprod Id", "Product Card Id",
        "Product Category Id", "Order Customer Id", "Order Zipcode"
    }),
    RenamedColumns = Table.RenameColumns(RemovedColumns, {
        {"order date (DateOrders)", "Order Date"},
        {"shipping date (DateOrders)", "Ship Date"},
        {"Days for shipping (real)", "Days Shipping Real"},
        {"Late_delivery_risk", "Late Delivery Risk"},
        {"Order Item Profit Ratio", "Profit Ratio"},
        {"Order Profit Per Order", "Profit Per Order"}
    }),
    FixedTypes = Table.TransformColumnTypes(RenamedColumns, {
        {"Order Date", type datetime}, {"Ship Date", type datetime},
        {"Sales", type number}, {"Profit Per Order", type number},
        {"Late Delivery Risk", Int64.Type}
    })
in
    FixedTypes
```

**Transformations applied:**
- Removed 12 irrelevant columns (PII, duplicates, image URLs)
- Renamed 14 columns for readability and professional labeling
- Corrected data types for 21 columns including datetime, decimal, and integer fields

---

## DAX Measures

All measures stored in a dedicated `_Measures` table following best practice data modeling conventions:

```dax
-- Core financial metrics
Total Sales = SUM('DataCoSupplyChainDataset'[Sales])
Total Profit = SUM('DataCoSupplyChainDataset'[Profit Per Order])
Total Orders = DISTINCTCOUNT('DataCoSupplyChainDataset'[Order Id])
Profit Margin % = DIVIDE([Total Profit], [Total Sales], 0) * 100
Avg Order Value = DIVIDE([Total Sales], [Total Orders], 0)

-- Logistics performance
Late Delivery Rate % = 
    DIVIDE(
        COUNTROWS(FILTER('DataCoSupplyChainDataset',
            'DataCoSupplyChainDataset'[Late Delivery Risk] = 1)),
        COUNTROWS('DataCoSupplyChainDataset'), 0) * 100

On Time Delivery Rate % = 100 - [Late Delivery Rate %]
Avg Days to Ship = AVERAGE('DataCoSupplyChainDataset'[Days Shipping Real])
Avg Days Late = 
    CALCULATE(AVERAGE('DataCoSupplyChainDataset'[Days Shipping Real]),
        'DataCoSupplyChainDataset'[Late Delivery Risk] = 1)

-- Time intelligence
Total Sales YTD = TOTALYTD([Total Sales], 'DateTable'[Date])
Total Sales Previous Year = 
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('DateTable'[Date]))
Sales YoY % = 
    VAR PriorSales = [Total Sales Previous Year]
    RETURN IF(ISBLANK(PriorSales) || PriorSales = 0, BLANK(),
        DIVIDE([Total Sales] - PriorSales, PriorSales, 0) * 100)
```

---

## Project Structure

```
dataco-supply-chain-powerbi/
│
├── DataCo_SupplyChain_Dashboard.pbix    # Power BI report file
├── README.md                            # This file
│
├── page1_executive_summary.png          # Dashboard screenshot
├── page2_regional_performance.png       # Dashboard screenshot
├── page3_product_analysis.png           # Dashboard screenshot
└── page4_logistics_intelligence.png     # Dashboard screenshot
```

---

## Key Business Questions Answered

| Question | Answer |
|----------|--------|
| Which market generates the most revenue? | Europe ($10.87M, 29.56%) |
| What is the overall profit margin? | 10.78% across all markets |
| Which shipping mode has the worst on-time performance? | First Class (~95% late delivery rate) |
| Which product category receives the most discounting? | As Seen on TV (~10% avg discount rate) |
| What percentage of orders arrive late? | 54.83% — more than half of all shipments |
| Which region has the longest delivery delays? | Central Africa and West Africa |
| Which department drives the most revenue? | Fan Shop followed by Apparel |

---

## About

Built by **Fahad Usman** as part of a business intelligence portfolio demonstrating applied data analysis skills in Power BI. The project mirrors real-world enterprise BI workflows including scripted data transformation, dimensional data modeling, DAX time intelligence, and multi-stakeholder dashboard design.

**Connect:** [LinkedIn](https://www.linkedin.com/in/fahad-usman) · [GitHub](https://github.com/cy-fahad)
