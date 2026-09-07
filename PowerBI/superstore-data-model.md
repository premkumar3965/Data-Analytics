# Superstore Data Model — What Was Built

Data model built directly inside your open Power BI Desktop file (via a
live connection — the file itself is still "Untitled", **save it** as
`Superstore-Sales-Report.pbix` in this folder to keep it).

Source data: `E:\Learnings\Dataset\Sample - Superstore.csv` (9,994 rows).

----------------------------------------
`Orders` table

The fact table — one row per order line item. Imported straight from the
CSV via Power Query (a Get Data > Text/CSV step, done here through an
M expression instead of the UI). Key columns: Order Date, Ship Date,
Region, Category, Sub-Category, Sales, Profit, Quantity, Discount.
----------------------------------------

----------------------------------------
`Date` table — marked as the official date table

A calculated table (`CALENDARAUTO()`) spanning the full range of dates
used anywhere in the model, with Year / Month Number / Month Name /
Quarter / Year-Month columns added. Marking a table as the "date table"
is what unlocks time-intelligence DAX functions like `SAMEPERIODLASTYEAR`.

```
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date],"MMMM"),
    "Quarter", "Q" & FORMAT([Date],"Q"),
    "Year-Month", FORMAT([Date],"MMM YYYY")
)
```

Linked to `Orders[Order Date]` — a standard many-to-one relationship
(many order lines → one date).
----------------------------------------

----------------------------------------
`Total Sales`, `Total Profit`

The two foundation measures — nearly every other measure in a sales
report builds on these.

```
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
```
----------------------------------------

----------------------------------------
`Profit Margin %`

Profit as a percentage of sales. Uses `DIVIDE` instead of `/` — `DIVIDE`
returns blank instead of erroring when the denominator is 0, which matters
once you start slicing by filters that might have zero sales.

```
Profit Margin % = DIVIDE([Total Profit], [Total Sales])
```
----------------------------------------

----------------------------------------
`Total Orders`, `Total Quantity`, `Average Discount`

```
Total Orders = DISTINCTCOUNT(Orders[Order ID])
Total Quantity = SUM(Orders[Quantity])
Average Discount = AVERAGE(Orders[Discount])
```

`Total Orders` uses `DISTINCTCOUNT` on Order ID rather than counting rows,
since one order can span multiple line items (multiple products).
----------------------------------------

----------------------------------------
`Sales PY`, `Sales YoY %` — time intelligence

```
Sales PY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
Sales YoY % = DIVIDE([Total Sales] - [Sales PY], [Sales PY])
```

`SAMEPERIODLASTYEAR` shifts the current filter context back exactly one
year — this only works correctly because `Date` is marked as the model's
date table. Verified: 2015 → -2.8%, 2016 → +29.5%, 2017 → +20.4%.
----------------------------------------

## Known quirk

`CALENDARAUTO()` extends the calendar to cover *every* date column in the
model, including `Ship Date` — so the Date table has a few days rolling
into 2018 with no matching orders (blank Sales/Profit for that sliver).
Harmless for visuals — Power BI hides categories with no data by default —
but don't be surprised if you see it while exploring the Date table
directly.
