# Executive Financial Performance Dashboard — Power BI

## Project Overview

This project is a three-page Power BI dashboard built to evaluate 2016 financial performance using actual sales, budget data, profitability metrics, product hierarchy, customer segments, territory performance, and prior-year sales.

The goal was to simulate an FP&A / business analyst workflow: transform raw Excel data, build a clean star-schema model, create reusable DAX measures, validate budget logic, and present findings in a management-facing dashboard.

**Tools used:** Power BI Desktop, Power Query, DAX  
**Dataset:** Kaggle — *Budget & Sales Dataset* by michaeldsouza16

---

## Business Objective

The dashboard was designed to answer four core questions:

1. How did 2016 actual sales perform against budget?
2. Which categories and subcategories drove the annual budget variance?
3. Which territories, products, and customer segments drove sales and gross profit?
4. How did 2016 sales compare with the prior year?

The final dashboard contains three pages:

- **Page 1 — Executive Financial Performance**
- **Page 2 — Budget Drivers & Subcategory Analysis**
- **Page 3 — Sales & Profitability Drivers**

---

## Dataset Structure

The source workbook contains six tables:

| Table | Purpose |
|---|---|
| Budget | Monthly 2016 budget by product subcategory |
| Calendar | Date dimension covering 2014–2017 |
| Customers | Customer demographics and attributes |
| Products | Product, subcategory, category, cost, and price information |
| Sales | Sales transaction-level records |
| Territory | Sales territory and geographic attributes |

The sales data covers multiple years, while the budget data covers **2016 only**.

---

## Data Preparation in Power Query

### FactBudget
- Unpivoted monthly budget columns into long format
- Created `ProductKey`, `BudgetMonth`, and `BudgetAmount`
- Converted `BudgetMonth` to Date
- Removed descriptive product fields from the fact table

### DimDate
- Validated one row per date
- Added Month Number, Start of Month, Year-Month, Month-Year, and YearMonth Sort
- Marked as the official date table
- Sorted Month-Year chronologically

### DimCustomer
- Validated unique `CustomerKey`
- Standardized text and numeric types
- Added Customer Name and House Owner fields

### DimProduct
- Validated unique `ProductKey`
- Standardized product attributes and numeric fields
- Cleaned inconsistent categorical values

### FactSales
- Standardized key, date, quantity, sales, cost, and tax fields
- Preserved transaction / line-level grain

### DimTerritory
- Validated unique `SalesTerritoryKey`
- Cleaned geographic fields
- Renamed `Continents` to `Continent`

---

## Data Model

The project uses a two-fact star schema.

### Fact tables
- `FactSales`
- `FactBudget`

### Dimension tables
- `DimDate`
- `DimProduct`
- `DimCustomer`
- `DimTerritory`

### Relationships
- `DimDate[Date]` → `FactSales[OrderDate]`
- `DimDate[Date]` → `FactBudget[BudgetMonth]`
- `DimProduct[ProductKey]` → `FactSales[ProductKey]`
- `DimProduct[ProductKey]` → `FactBudget[ProductKey]`
- `DimCustomer[CustomerKey]` → `FactSales[CustomerKey]`
- `DimTerritory[SalesTerritoryKey]` → `FactSales[SalesTerritoryKey]`

Relationships use one-to-many cardinality with single-direction filtering from dimension to fact.

---

## Important Modeling Issue: Budget Grain Validation

One of the most important parts of the project was validating the grain of the budget data.

An early version of the dashboard compared total budget against actual sales filtered to the **17 exact ProductKeys listed in the budget table**. This produced an implausible result of roughly **2% budget attainment**.

Further inspection showed that the budget table contained one row for each **product subcategory**, while the ProductKey behaved more like a representative product identifier than a complete SKU-level budget definition. Actual 2016 sales included many more ProductKeys within those same subcategories.

As a result, the initial ProductKey-based comparison excluded most relevant actual sales and materially understated performance.

### Correction

The ProductKey-restricted actual-sales measure was removed. The final dashboard compares total 2016 actual sales against total 2016 budget sales.

Validated annual results:

- **Actual Sales:** $16.47M
- **Budget Sales:** $16.87M
- **Sales Variance:** -$395.96K
- **Sales Variance %:** -2.35%
- **Budget Attainment:** 97.65%

This validation prevented a technically valid but business-misleading result from being presented.

---

## Key DAX Measures

```DAX
Actual Sales =
SUM(FactSales[SalesAmount])
```

```DAX
Budget Sales =
SUM(FactBudget[BudgetAmount])
```

```DAX
Sales Variance =
[Actual Sales] - [Budget Sales]
```

```DAX
Sales Variance % =
DIVIDE(
    [Sales Variance],
    [Budget Sales]
)
```

```DAX
Budget Attainment % =
DIVIDE(
    [Actual Sales],
    [Budget Sales]
)
```

```DAX
Total Product Cost =
SUM(FactSales[TotalProductCost])
```

```DAX
Gross Profit =
[Actual Sales] - [Total Product Cost]
```

```DAX
Gross Margin % =
DIVIDE(
    [Gross Profit],
    [Actual Sales]
)
```

```DAX
Actual Sales YTD =
TOTALYTD(
    [Actual Sales],
    DimDate[Date]
)
```

```DAX
Budget Sales YTD =
TOTALYTD(
    [Budget Sales],
    DimDate[Date]
)
```

```DAX
Sales Variance YTD =
[Actual Sales YTD] - [Budget Sales YTD]
```

```DAX
Sales Variance YTD % =
DIVIDE(
    [Sales Variance YTD],
    [Budget Sales YTD]
)
```

```DAX
Prior Year Sales =
CALCULATE(
    [Actual Sales],
    SAMEPERIODLASTYEAR(DimDate[Date])
)
```

```DAX
YoY Sales Growth % =
DIVIDE(
    [Actual Sales] - [Prior Year Sales],
    [Prior Year Sales]
)
```

---

## Dashboard Pages

## 1. Executive Financial Performance

This page summarizes overall 2016 financial performance.

### Main visuals
- Actual Sales KPI
- Budget Sales KPI
- Sales Variance % KPI
- Gross Profit KPI
- Gross Margin % KPI
- Actual vs. Budget monthly trend
- Monthly Budget Attainment %
- Budget Variance by Category
- Executive key findings

### Key findings
- 2016 sales totaled **$16.47M**, finishing **2.35% below the $16.87M budget**
- All three major categories finished below budget
- Clothing had the largest percentage shortfall at the category level
- Monthly performance remained relatively close to target overall
- August showed the largest monthly underperformance
- Several months exceeded budget

---

## 2. Budget Drivers & Subcategory Analysis

This page explains where the annual budget shortfall came from.

### Main visuals
- Sales Variance ($) by Subcategory
- Budget Size vs. Sales Variance % scatterplot
- Month × Subcategory variance heatmap
- Subcategory-level detail table

### Key findings
- The **$395.96K annual shortfall was highly concentrated**
- **Touring Bikes:** approximately **-$211K**
- **Mountain Bikes:** approximately **-$137K**
- Together, Touring Bikes and Mountain Bikes accounted for roughly **88% of the annual budget miss**
- Road Bikes finished approximately on plan at about **-0.30% variance**
- Only a small number of subcategories exceeded annual budget
- Smaller subcategories sometimes showed large percentage swings but relatively small dollar impacts

### Interpretation
The annual miss was concentrated in a few large-dollar subcategories rather than spread evenly across the portfolio.

---

## 3. Sales & Profitability Drivers

This page evaluates the underlying drivers of sales, profit, customer demand, and year-over-year growth.

### Main visuals
- Gross Profit by Region
- Actual Sales vs. Gross Margin % by Subcategory
- Actual Sales by Region and Category
- Actual Sales by Occupation
- Monthly Actual Sales vs. Prior Year Sales

### Key findings
- **Australia** was the strongest geographic market by sales and gross profit
- The **Southwest** was the next-largest geographic contributor
- Bikes drove the majority of sales across major regions
- Professional customers generated the highest sales among occupation segments
- High sales volume did not always correspond to the highest gross-margin percentage
- Several smaller subcategories produced stronger margin percentages than the highest-volume subcategories
- 2016 monthly sales exceeded prior-year sales throughout the year

---

## Business Implications & Recommendations

### 1. Investigate Touring Bikes and Mountain Bikes
These two subcategories generated most of the annual budget shortfall. Pricing, demand assumptions, product mix, and volume expectations should be reviewed before broader budget changes are considered.

### 2. Prioritize dollar impact alongside percentage variance
Large percentage swings in small subcategories can look important while contributing little to total financial performance. Dollar variance and variance percentage should be evaluated together.

### 3. Protect high-performing geographic markets
Australia and the Southwest are major contributors to sales and gross profit. Performance in these markets has an outsized effect on overall results.

### 4. Evaluate product-mix and margin opportunities
Some smaller subcategories generate stronger gross-margin percentages than the highest-volume categories. These products may offer opportunities to improve profitability if additional volume can be captured.

### 5. Monitor year-over-year momentum
2016 materially outperformed the prior year across the monthly sales trend. Future planning should evaluate whether this growth is sustainable and whether budget assumptions reflect the company’s growth trajectory.

---

## Skills Demonstrated

- Power BI dashboard development
- Power Query data cleaning and transformation
- Star-schema dimensional modeling
- Multiple fact-table modeling
- Relationship design and filter context
- DAX measures
- `SUM`
- `DIVIDE`
- `CALCULATE`
- `TOTALYTD`
- `SAMEPERIODLASTYEAR`
- Actual vs. budget analysis
- Variance analysis
- Gross profit and margin analysis
- Time intelligence
- Conditional formatting
- Matrix / heatmap analysis
- Scatterplot analysis
- Data-grain validation and troubleshooting
- Executive dashboard design
- Business interpretation and recommendations

---

## Project Takeaway

The most important lesson from this project was that technically correct calculations can still produce misleading business results when the underlying data grain is misunderstood.

The initial budget comparison produced an extreme and implausible result. Rather than accepting the output, the source data and model were investigated, the grain mismatch was identified, and the comparison logic was corrected.

The final dashboard shows 2016 sales finishing modestly below budget while revealing that the annual shortfall was concentrated primarily in Touring Bikes and Mountain Bikes. At the same time, the business demonstrated strong year-over-year growth and significant geographic concentration in its highest-performing markets.

This project demonstrates an end-to-end analytics workflow:

**data preparation → modeling → DAX → validation → visualization → business interpretation**
