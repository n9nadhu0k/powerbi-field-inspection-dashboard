# Data Dictionary

Source file: `data/Field_Operations_Practice_Data.xlsx`. All data is **synthetic**.

## Fact tables

### Inspections (one row per pole inspection)

| Column | Type | Description |
|---|---|---|
| Inspection_ID | Text | Unique inspection key (INS-000001). 45 duplicates exist in the raw file |
| Inspection_Date | Date | Date of the inspection |
| Pole_ID | Text | Pole identifier |
| Crew_ID | Text | Links to Dim_Crew |
| Region_ID | Text | Links to Dim_Region (the state where the pole is) |
| Client_ID | Text | Links to Dim_Client |
| Pole_Type | Text | Wood, Steel or Concrete |
| Pole_Age_Years | Whole number | Age of the pole in years |
| Inspection_Status | Text | Pass, Fail, Needs Repair, Pending (raw file has mixed case and blanks) |
| Defect_Type | Text | Rot, Leaning, Cracked, Corrosion, Damaged Crossarm, Woodpecker Damage (blank when no defect) |
| Inspection_Hours | Decimal | Hours spent on the inspection (raw file has outliers) |
| Defect_Clean | Text | Derived in Power Query: Defect_Type with blanks labeled Not Recorded or No Defect |
| Hours_Clean | Decimal | Derived in Power Query: Inspection_Hours with impossible values set to blank |
| Age Band | Text | Derived in DAX: 0-10, 11-25, 26-40, 41+ years |

### Invoices (one row per weekly invoice per client)

| Column | Type | Description |
|---|---|---|
| Invoice_ID | Text | Unique invoice key |
| Invoice_Date | Date | Invoice date |
| Client_ID | Text | Links to Dim_Client |
| Poles_Invoiced | Whole number | Number of poles billed |
| Rate_Per_Pole | Decimal | Price per pole (USD) |
| Due_Date | Date | Invoice date plus 30 days |
| Invoice_Status | Text | Paid, Pending or Overdue |
| Paid_Date | Date | Payment date (blank if not paid) |

The invoice **amount** is not stored. It is calculated in DAX as Poles_Invoiced multiplied by Rate_Per_Pole.

### Targets (one row per region per month)

| Column | Type | Description |
|---|---|---|
| Region_ID | Text | Links to Dim_Region |
| Month | Date | First day of the month |
| Target_Poles | Whole number | Planned number of inspections |

## Dimension tables

| Table | Columns | Description |
|---|---|---|
| Dim_Crew | Crew_ID, Crew_Name, Supervisor_ID, Region_ID | 24 inspection crews |
| Dim_Supervisor | Supervisor_ID, Supervisor_Name, Region_ID | 8 supervisors |
| Dim_Region | Region_ID, State, Region | 8 states grouped into 5 regions |
| Dim_Client | Client_ID, Client_Name, Client_Type | 3 clients (2 telecom, 1 utility) |
| Dim_Date | Date, Year, Quarter, Month Number, Month, Year-Month | Calendar table built in DAX, 1 Jan 2025 to 31 Dec 2026, marked as the date table |

## Relationships

| From (many) | To (one) | Column |
|---|---|---|
| Inspections | Dim_Crew | Crew_ID |
| Dim_Crew | Dim_Supervisor | Supervisor_ID |
| Inspections | Dim_Region | Region_ID |
| Inspections | Dim_Client | Client_ID |
| Invoices | Dim_Client | Client_ID |
| Targets | Dim_Region | Region_ID |
| Inspections | Dim_Date | Inspection_Date to Date |
| Invoices | Dim_Date | Invoice_Date to Date |
| Targets | Dim_Date | Month to Date |

All relationships are many-to-one with single-direction filtering. The relationship from Dim_Crew to Dim_Region was deleted on purpose, so Inspections has only one active path to Dim_Region.

## Data quality issues in the raw file

| Issue | Count |
|---|---|
| Duplicate Inspection_ID rows | 45 |
| Blank Inspection_Status | 20 |
| Fail rows with a blank Defect_Type | about 25 |
| Impossible Inspection_Hours (0, or 36 to 60) | 8 |
| Inspection_Status with inconsistent case or spaces | 60 |
