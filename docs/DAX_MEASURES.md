# DAX Measures

All measures live in a dedicated `_Measures` table. Measures are calculated at query time, using the current filter context (slicers, chart axes and relationships).

## Counts and rates

```
Total Inspections = COUNTROWS(Inspections)
```
Counts the visible rows in Inspections.

```
Pass Inspections = CALCULATE([Total Inspections], Inspections[Inspection_Status] = "Pass")
```
Counts only the rows where the status is Pass. `CALCULATE` adds a filter to the current context.

```
Pass Rate % =
DIVIDE(
    [Pass Inspections],
    CALCULATE([Total Inspections], Inspections[Inspection_Status] IN {"Pass", "Fail", "Needs Repair"})
)
```
Share of **completed** inspections that passed. Pending and Unknown are excluded from the denominator.

```
Fail Rate % =
DIVIDE(
    CALCULATE([Total Inspections], Inspections[Inspection_Status] IN {"Fail", "Needs Repair"}),
    CALCULATE([Total Inspections], Inspections[Inspection_Status] IN {"Pass", "Fail", "Needs Repair"})
)
```
Pass Rate % plus Fail Rate % always equals 100%, because both use the same denominator.

```
Avg Inspection Hours = AVERAGE(Inspections[Hours_Clean])
```
Uses the cleaned column, so the impossible values (set to blank in Power Query) are ignored.

## Targets

```
Total Target Poles = SUM(Targets[Target_Poles])
```

```
Target Achievement % = DIVIDE([Total Inspections], [Total Target Poles])
```
Above 100% means the target was beaten.

## Invoicing

```
Total Invoiced Amount = SUMX(Invoices, Invoices[Poles_Invoiced] * Invoices[Rate_Per_Pole])
```
`SUMX` multiplies row by row first, then adds. The Invoices table has no amount column on purpose.

```
Outstanding Amount = CALCULATE([Total Invoiced Amount], Invoices[Invoice_Status] <> "Paid")
```
Pending plus Overdue invoices.

```
Overdue Amount = CALCULATE([Total Invoiced Amount], Invoices[Invoice_Status] = "Overdue")
```

```
Avg Days to Pay =
AVERAGEX(
    FILTER(Invoices, NOT ISBLANK(Invoices[Paid_Date])),
    DATEDIFF(Invoices[Invoice_Date], Invoices[Paid_Date], DAY)
)
```
Averages the days between invoice and payment, for paid invoices only.

## Time intelligence

These require the `Dim_Date` table to be marked as the date table.

```
Inspections Last Month = CALCULATE([Total Inspections], DATEADD(Dim_Date[Date], -1, MONTH))
```
Shifts the current dates back by one month.

```
Inspections MoM % =
DIVIDE([Total Inspections] - [Inspections Last Month], [Inspections Last Month])
```
(New minus old) divided by old. Returns blank when there is no previous month.

```
Inspections YTD = TOTALYTD([Total Inspections], Dim_Date[Date])
```
Running total from 1 January, restarting each year.

## Ranking

```
Crew Rank = RANKX(ALL(Dim_Crew[Crew_Name]), [Pass Rate %], , DESC)
```
`ALL` removes the crew filter so every crew competes against all 24 crews. Rank 1 is the highest pass rate.

## Calculated tables and columns

```
Dim_Date =
ADDCOLUMNS(
    CALENDAR(DATE(2025,1,1), DATE(2026,12,31)),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month Number", MONTH([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Year-Month", FORMAT([Date], "YYYY-MM")
)
```

```
Age Band =
SWITCH(
    TRUE(),
    Inspections[Pole_Age_Years] <= 10, "0-10 yrs",
    Inspections[Pole_Age_Years] <= 25, "11-25 yrs",
    Inspections[Pole_Age_Years] <= 40, "26-40 yrs",
    "41+ yrs"
)
```
A calculated column (computed once per row at refresh), unlike a measure.

## Power Query columns

| Column | Rule |
|---|---|
| `Defect_Clean` | If Defect_Type is blank and the status is Fail or Needs Repair, then "Not Recorded"; else if blank, "No Defect"; else the original value |
| `Hours_Clean` | If Inspection_Hours is 0 or below, or above 8, then blank; else the original value |
