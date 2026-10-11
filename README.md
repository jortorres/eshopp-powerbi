# eShopp Sales & Web Analytics Dashboard (Power BI)

A 9-page Power BI report built on a synthetic e-commerce dataset. It combines sales, customer, product, store, and website-session data into one star-style model, and answers questions like *what drives revenue, which channels and devices convert, and where are customers located?*

![CEO Dashboard](CEO Dashboard 2026-10-11 at 6.18.04 PM.png)

## What this project demonstrates

- **Data prep in Power Query:** typing columns, removing duplicate sales records, splitting a combined `browser|os` field into two columns, building a derived `os_on_device` column, and replacing missing quality scores.
- **Data modeling:** a sales fact table related to customers, products, stores, and a date table, plus a second fact table (website sessions) linked through customers and stores.
- **DAX:** measures and calculated columns for revenue, units, discounts, order-size buckets, and a session-quality gauge (see below).
- **Report design:** 9 pages covering KPIs, trends, rankings, geography, and an executive summary page with slicers and drill-down.

## Data

Seven CSV files in [`/data`](data/): `sales`, `customers`, `products`, `stores`, `website_sessions`, `dates`, `geographies`.

- The data is synthetic (fictional customers and store names). It contains no real customer or company data.
- **Source:** _[add where the dataset came from, e.g., course or tutorial name, and its license/terms]_
- Raw `sales.csv` has 805 rows with 5 duplicate `sale_id` values; the model keeps 800 unique sales.

## Data model

| Table | Role | Joins to |
|---|---|---|
| `sales` | Fact | `customers`, `products`, `stores`, `dates` |
| `website_sessions` | Fact | `customers`, `stores` |
| `customers`, `products`, `stores`, `dates` | Dimensions | |
| `geographies` | Standalone lookup (population by country/state/city) | not related |

## Key DAX

**Measures**

```dax
total_revenue = SUM(sales[discounted_price])
total_units_sold = SUM(sales[sale_quantity])
customer_avg_age = AVERAGE(customers[customer_age])
session_quality_score_avg_scaled = INT(AVERAGEX(website_sessions, website_sessions[session_quality_score] * 100))
```

**Calculated columns**

```dax
price            = sales[sale_quantity] * RELATED(products[product_list_price])
discounted_price = sales[price] * (1 - sales[sale_discount])
bulk_order       = IF(sales[sale_quantity] > 3, TRUE(), FALSE())
order_size       = SWITCH(TRUE, sales[discounted_price] < 100, "Tiny",
                                sales[discounted_price] < 200, "Mid", "Massive")
```

## Report pages

| # | Page | What it shows |
|---|---|---|
| 1 | Customers | Customers by payment type |
| 2 | Sessions | Pages visited by device and over time |
| 3 | Cards | Headline KPIs (revenue, units, average age, oldest store, quality score) |
| 4 | KPIs | Session-quality gauge vs. target, sales by store type, revenue by category |
| 5 | Trends | Session duration and pages over time; sales vs. average discount by week |
| 6 | Analysis | Duration vs. pages visited (scatter); revenue by category (waterfall) |
| 7 | Rankings | Category revenue rankings by weekday (ribbon chart) |
| 8 | Geo | Population by geography |
| 9 | CEO Dashboard | Executive view: revenue and units cards, revenue trend by category, sales by brand, session duration by store, region and date slicers, population drill-down |

Screenshots of every page are in [`/screenshots`](screenshots/).

## Example findings

_(Computed from the deduplicated data; confirm they match what your visuals show.)_

- Total revenue is about **$167K** on **1,793 units** across 800 sales, with an average discount of roughly 11%.
- **Appliances** drive about 60% of revenue; the top brand, **Smartly**, accounts for about half.
- Retail stores sold more units (1,072) than the online store (721).
- Mobile sessions convert best (about 24% made a purchase), ahead of desktop (about 19%) and tablet (about 10%).

## How to open it

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Open `eshopp.pbip` (a Power BI Project, stored as readable text files so changes show clearly in Git).
3. The tables load from CSVs using a local path. In Power Query, update each source path to point to the `/data` folder in this repo, then refresh.

## Tools

Power BI Desktop (PBIP project format), Power Query (M), DAX, CSV data.

## What I'd do next

- Replace hard-coded dates in measures with a dynamic date selection.
- Add a proper `geographies` relationship (for example, via customer region or store location) to enable map visuals.
- Move the source path into a Power Query parameter so the project opens on any machine.
