# Power BI DAX Measures Reference
## Enterprise Sales, Inventory & Demand Forecasting Analytics

---

## Data Model — Star Schema

```
                    ┌─────────────┐
                    │  DimDate    │
                    │  (1,096 rows)│
                    └──────┬──────┘
                           │ Date [1]
                           │
┌─────────────┐     ┌──────▼──────┐     ┌─────────────┐
│ DimCustomer │─────│  FactSales  │─────│DimInventory │
│ (2,000 rows)│     │ (50,000 rows)│    │  (500 SKUs) │
└─────────────┘     └─────────────┘     └─────────────┘
  CustomerID [1]    CustomerID [*]       SKUID [1]
                    SKUID [*]
```

---

## 20 DAX Measures

### Revenue Measures

```dax
Total Revenue = SUM(FactSales[Revenue])

Total COGS = SUM(FactSales[COGS])

Gross Profit = [Total Revenue] - [Total COGS]

Gross Margin % = DIVIDE([Gross Profit], [Total Revenue], 0) * 100

Discount Impact =
    SUMX(
        FactSales,
        FactSales[Quantity] * FactSales[UnitPrice] * FactSales[Discount]
    )
```

### Time Intelligence

```dax
Revenue YTD = TOTALYTD([Total Revenue], DimDate[Date])

Revenue PYTD =
    CALCULATE(
        [Total Revenue],
        SAMEPERIODLASTYEAR(DimDate[Date])
    )

YoY Growth % =
    DIVIDE(
        [Revenue YTD] - [Revenue PYTD],
        [Revenue PYTD], 0
    ) * 100

Revenue MoM % =
    VAR cur  = [Total Revenue]
    VAR prev = CALCULATE([Total Revenue], DATEADD(DimDate[Date], -1, MONTH))
    RETURN DIVIDE(cur - prev, prev, 0) * 100

Revenue Rolling 3M =
    CALCULATE(
        [Total Revenue],
        DATESINPERIOD(DimDate[Date], LASTDATE(DimDate[Date]), -3, MONTH)
    )
```

### Orders & Customers

```dax
Total Orders = DISTINCTCOUNT(FactSales[OrderID])

Total Customers = DISTINCTCOUNT(FactSales[CustomerID])

Avg Order Value = DIVIDE([Total Revenue], [Total Orders], 0)

Revenue per Customer = DIVIDE([Total Revenue], [Total Customers], 0)

Customer Retention Rate =
    VAR prev_customers =
        CALCULATE(
            DISTINCTCOUNT(FactSales[CustomerID]),
            DATEADD(DimDate[Date], -12, MONTH)
        )
    VAR cur_customers = [Total Customers]
    RETURN DIVIDE(cur_customers, prev_customers, 0) * 100
```

### Inventory

```dax
Stock Value =
    SUMX(
        DimInventory,
        DimInventory[StockOnHand] * DimInventory[UnitCost]
    )

SKUs Below Reorder =
    COUNTROWS(
        FILTER(
            DimInventory,
            DimInventory[StockOnHand] < DimInventory[ReorderPoint]
        )
    )

Avg Lead Time = AVERAGE(DimInventory[LeadTimeDays])

Top SKU Revenue = MAXX(VALUES(FactSales[SKUID]), [Total Revenue])

Fulfillment Rate % =
    DIVIDE(
        COUNTROWS(FILTER(FactSales, FactSales[Quantity] > 0)),
        [Total Orders], 0
    ) * 100
```

---

## How to Load into Power BI Desktop

1. Open **Power BI Desktop** → Get Data → Excel
2. Select `PowerBI_DataModel.xlsx`
3. Load all 4 sheets: `FactSales`, `DimInventory`, `DimCustomer`, `DimDate`
4. Go to **Model View** and create relationships:
   - `FactSales[CustomerID]` → `DimCustomer[CustomerID]` (Many-to-One)
   - `FactSales[SKUID]` → `DimInventory[SKUID]` (Many-to-One)
   - `FactSales[OrderDate]` → `DimDate[Date]` (Many-to-One)
5. In **Report View**, create a new Measure for each DAX formula above
6. Mark `DimDate` as a **Date Table** (Table tools → Mark as date table → Date)

---

## Recommended Dashboards

| Dashboard | Key Visuals |
|-----------|-------------|
| Sales Overview | Revenue KPI, YoY trend line, Revenue by Region map, Channel mix donut |
| Customer Analysis | Customer count, AOV, Retention %, Top customers table |
| Product Performance | Revenue by Category bar, Top 10 SKUs, Margin scatter |
| Inventory Management | Stock Value KPI, SKUs below reorder, Lead time by supplier |
| Demand Forecasting | 30/60/90-day forecast line, SKU-level table, Confidence bands |
| Supply Chain | Fulfillment rate, Discount impact, Regional heatmap |
| Executive Summary | All KPIs, YoY Growth, Quarterly trend |
| Financial P&L | Revenue, COGS, Gross Profit waterfall, Margin % trend |
