# Healthcare-Data-Analysis-using-Power-BI
Introduction

Created an interactive dashboard using Power BI to reveal key trends in hospital visitor data for informed decision-making. Utilized DAX for insights into patient demographics and service usage, boosting operational efficiency and patient satisfaction. This dynamic data visualization tool incorporates data modeling principles and various data sources for effective data transformation. By implementing KPIs and utilizing Power Query, healthcare professionals can explore interactive visuals that enhance resource optimization and improve patient care quality through business intelligence.



A Power BI healthcare operations and finance dashboard showing KPIs (Available Beds, Net Patient Revenue, Capital Expenditure, Total Hospitals, Total Revenue), trends (discharges, patient days, revenue), geospatial inpatient revenue, revenue breakdowns by hospital type and state, inpatient expenses and average length of stay.

---

## Table of contents
- Project overview
- What you'll see (visuals & layout)
- Data sources & structure
- Data preparation (Power Query)
- Data model & relationships
- Key DAX measures (examples)
- Report usage & interactive controls
- Performance & refresh recommendations
- Deployment & sharing
- Enhancements & roadmap
- Contact

---

## Project overview
This Power BI report provides a consolidated view of hospital operational and financial metrics across years, counties, and facilities. It is designed to enable quick strategic insights via KPI cards, trend analysis, geo analytics, and detailed breakdowns by hospital type and state.

Primary goals:
- Monitor capacity (Available Beds)
- Track revenue and cost trends (Net Patient Revenue, Total Revenue, Total Inpatient Expenses)
- Visualize patient volume and utilization (Total Discharges, Patient Days, Average Days Stayed)
- Identify geographic concentration of inpatient revenue
- Support data-driven resource and financial planning

---

## What you'll see (visuals & layout)
Top row: title + slicers
- Year slicer (single or multi-select)
- County Name slicer
- Facility Name slicer

KPI cards:
- Available Beds
- Net Patient Revenue
- Total Capital Expenditure
- Total Hospitals
- Total Revenue

Charts / visuals (left-to-right, top-to-bottom):
- Total Discharges (yearly bar chart)
- Patient Days (line chart)
- Net-Patient Revenue (horizontal bar per year)
- Type of Hospital Revenue (pie / donut)
- Inpatient Revenue by County (map with bubbles)
- Revenue Trend (multi-measure line chart)
- Statewise Hospital Revenue (stacked bar or clustered bar)
- Total Inpatient Expenses (bar chart by year)
- Average Days Stayed by Patients (donut chart showing distribution by year)

Notes:
- Colors and layout in the sample screenshot use high-contrast yellow/blue theme.
- Tooltips provide contextual details (measure values, facility name, year).

---

## Data sources & structure
This project can be fed from one or more of:
- CSV / Excel extracts (e.g., annual financials, inpatient records)
- SQL Server, Azure SQL, Redshift or other relational DBs
- Data warehouse views / exported extracts

Typical tables used:
- Calendar (Date, Year, Month, MonthNumber, Quarter)
- Facilities (FacilityID, FacilityName, CountyName, State, Type_Hosp, Beds)
- Inpatient (FacilityID, Date, Year, Discharges, PatientDays, AvgStay, DRG, Payer)
- Financials (FacilityID, Date, Year, NET_PAT_REV_CC, GROS_INPAT_REV_CC, TOTAL_REV, CAPITAL_EXP)
- Expenses (FacilityID, Date, Year, EXP_PIPS)
- (Optional) Lookup tables: County, State, HospitalType

Recommended column types:
- Date columns as Date
- Numeric metrics as Decimal / Currency
- Keys suitably typed (FacilityID consistent across tables)

---

## Data preparation (Power Query)
Common ETL steps:
- Normalize date fields and create a Calendar table (continuous date range, mark Year/Month/Quarter)
- Trim and standardize Facility and County names (remove trailing spaces, unify abbreviations)
- Convert numeric columns (strings with commas/currency symbols) to numeric types
- Aggregate large transaction data into monthly/yearly summarized tables if necessary for performance
- Ensure facility lookup has a stable key to join to fact tables
- Geocode facilities (latitude/longitude) for map bubbles if not already available

Power Query tips:
- Use query folding when connecting to relational sources
- Reduce columns (keep only required fields) before loading to the model
- Remove unnecessary rows (e.g., incomplete records) to speed up visuals

---

## Data model & relationships
- Calendar[Date] 1 - * Inpatient[Date] and Financials[Date] (or use Year-based relationships)
- Facilities[FacilityID] 1 - * Inpatient[FacilityID]
- Facilities[FacilityID] 1 - * Financials[FacilityID]
- Expenses[FacilityID] related to Facilities as well

Model design suggestions:
- Star schema preferred: small dimension tables (Calendar, Facilities, HospitalType) and large fact tables (Inpatient, Financials)
- Avoid bi-directional relationships unless required for specific calculations
- Use surrogate integer keys where possible

---

## Key DAX measures (examples)
Below are sample DAX measures you can adapt to your schema. Replace table/column names to match your model.

Basic measures:
```DAX
Total Beds = SUM( Facilities[AvailableBeds] )

Total Hospitals = DISTINCTCOUNT( Facilities[FacilityID] )

Net Patient Revenue = SUM( Financials[NET_PAT_REV_CC] )

Total Revenue = SUM( Financials[TOTAL_REV] )

Total Capital Expenditure = SUM( Financials[CAPITAL_EXP] )
```

Patient / volume measures:
```DAX
Total Discharges = SUM( Inpatient[Discharges] )

Patient Days = SUM( Inpatient[PatientDays] )

Average Days Stayed = DIVIDE( SUM( Inpatient[PatientDays] ), SUM( Inpatient[Discharges] ), 0 )
```

Expenses:
```DAX
Total Inpatient Expenses = SUM( Expenses[EXP_PIPS] )
```

Trends (YOY and period comparisons):
```DAX
Net Patient Revenue (YOY) = 
VAR Current = [Net Patient Revenue]
VAR Prev = CALCULATE( [Net Patient Revenue], SAMEPERIODLASTYEAR( Calendar[Date] ) )
RETURN
DIVIDE( Current - Prev, Prev, BLANK() )
```

Revenue trend over time (for line charts):
```DAX
Revenue by Date = 
CALCULATE( [Total Revenue], ALLSELECTED( Calendar ) )
```

Top N / rankings:
```DAX
Revenue Rank (Facility) = RANKX( ALLSELECTED( Facilities ), [Net Patient Revenue], , DESC, Dense )
```

Formatting:
- Apply thousands (K), millions (M) or billions (B) display formatting on KPI cards to match screenshot.

---

## Report usage & interactive controls
- Use the Year, County Name, and Facility Name slicers to scope the report.
- Clicking on map bubbles filters the other visuals to that geographic selection.
- Use drill-through to open a facility-level details page (configure a facility drill-through page).
- Use bookmarks for pre-set views (e.g., Financial Overview, Capacity Overview).
- Configure tooltips with small multi-field cards to show contextual values.

---

## Performance & refresh recommendations
- Limit visuals that use entire-row level calculations; prefer pre-aggregated tables for heavy measures.
- Use VertiPaq-friendly data types (integers rather than strings where possible).
- Reduce cardinality of columns used in slicers and relationships.
- If connecting to on-premises DBs, configure an On-premises data gateway in Power BI Service for scheduled refreshes.
- Enable incremental refresh for large fact tables (requires Power BI Pro / Premium and date fields).

---

## Deployment & sharing
1. Save Power BI Desktop (.pbix).
2. Publish to Power BI Service (My Workspace or a shared workspace).
3. If data source is on-premises, configure the Gateway and create scheduled refresh credentials.
4. Share reports to stakeholders or embed in Teams/SharePoint if needed.
5. Manage permissions via workspace roles and optionally Row-Level Security (RLS) for data restrictions by county/facility.

RLS example approach:
- Create a Role (e.g., "CountyViewer") with a DAX filter on Facilities:
```DAX
[CountyName] = LOOKUPVALUE( UserCountyMapping[County], UserCountyMapping[UserPrincipalName], USERPRINCIPALNAME() )
```
- Or use a mapping table (User, County) and filter via USERPRINCIPALNAME().

---

## Enhancements & roadmap
Possible improvements:
- Add facility-level drillthrough pages with monthly trend and KPIs
- Build anomaly detection (outlier alerts) for sudden revenue or discharge changes
- Add forecasting (Azure ML / built-in Power BI forecasting) for revenue, patient days
- Add payer mix and margin analytics by DRG
- Implement certificate-based secure gateway with service principal for automation
- Deploy to Power BI Premium for large-scale refreshes and improved performance

---

## Changelog
- v1.0 — Initial dashboard containing KPIs, trend visuals, map, and breakdowns (discharges, revenue, expenses)
- (Add further updates here)

---

## Contact
Author / maintainer: vishwanreddy (vishwanreddypathuti@gmail.com)

    <img width="1091" height="683" alt="Power Bi Dashboard" src="https://github.com/user-attachments/assets/685e533a-3d7e-46eb-93b0-b38b5ff2337e" />


##Overview
--------
This Excel dashboard provides a high-level health care report for 2016–2021. It visualizes hospital operational and financial KPIs (patient days, patient stays, revenue, discharges, hospital counts) with interactive slicers (Year) to filter the entire dashboard. The dashboard is designed for analysts and stakeholders to quickly review trends by year, hospital type, and state.

##Top-level KPIs & Visuals
------------------------
- Title: Health Care Report 2016-2021
- Year slicer: BEG_DATE (Year) — allows selecting a single year or multiple years (image shows 2020 selected)
- Charts / Panels:
  - Patient Days (bar): total patient days for the selected year(s)
  - Patient Stays (pie): patient stays count (or measure) for the selected year(s)
  - Type of Hospital Revenue (area/line): revenue by hospital type (e.g., Comparable, Hospital-LTC Emphasis, Kaiser Foundation, Psychiatric, Shriners)
  - State-wise No. of Hospitals / Revenue (line/marker): state-level hospital counts and revenue comparison
  - Total Hospitals (pie): distribution of hospital types and counts (values like 720 shown)
  - Net Patient Revenue (pie): total net revenue (number displayed)
  - Revenue Trend (column): revenue trend for the selected year(s)
  - Total Discharge (column): discharges total
  - MTD (column): month-to-date revenue (e.g., Jan)
  - YTD (pie): year-to-date revenue
  - QTD (bar): quarter-to-date revenue
- Notes on labels: some chart labels may display in scientific notation (e.g., 1.56353E+11) when values are very large — see troubleshooting below.

##Data Source & Structure
-----------------------
This dashboard expects a flat table (or tables in the Data Model / Power Query output) containing transaction or aggregated hospital-level records. Recommended fields:

- BEG_DATE (Date or Year): Year or full date used for year slicer (e.g., 2016, 2017, …)
- Year (Number or Text): 2016, 2017, ...
- State (Text): e.g., Alameda, Fresno, Corona, Victorville
- Hospital_Name (Text): name of hospital or identifier
- Hospital_Type (Text): e.g., Comparable, Hospital-LTC Emphasis, Kaiser Foundation Health, Psychiatric Health Facilities, Shriners Hospitals
- Patient_Days (Numeric)
- Patient_Stays (Numeric)
- Discharges (Numeric)
- Net_Patient_Revenue (Numeric) — currency
- Revenue (Numeric) — currency
- Month (Text/Number) and Quarter (Text/Number), if MTD/QTD are computed
- Any grouping or flag fields used in pivot tables (region, comparable flag, etc.)

If you use the Data Model, include relationships or lookup tables (State meta, Hospital types) as necessary.

##Excel Features / Requirements
-----------------------------
- Excel 2016 or later (Office 365 recommended) for best compatibility
- PivotTables and PivotCharts (charts are driven by pivot caches)
- Slicers for interactivity (BEG_DATE Year slicer)
- Power Query (Get & Transform) — if you import/transform raw data
- Optional: Data Model (if using relationships) and DAX measures for advanced KPIs

##How the Dashboard is Built
--------------------------
- Raw data is loaded into a data sheet or into Power Query and then loaded into the workbook/data model.
- PivotTables are built from the table/data model and each pivot is connected to a PivotChart.
- A Year slicer (BEG_DATE) is connected to the pivot caches to filter all charts.
- Chart formatting uses dark gray background with blue series and white titles as shown.

##How to Refresh & Update
-----------------------
1. Replace or update the raw data source sheet (the table used by Power Query or Pivot).
2. If using Power Query:
   - Data → Get Data → Refresh All (or use the Query pane to refresh individual queries).
3. If using simple table + pivots:
   - Ensure table range includes new rows (convert raw data to an Excel Table: Insert → Table).
   - Right click any PivotTable → Refresh.
4. To refresh all: Data → Refresh All (or press Ctrl+Alt+F5).
5. After refresh, verify slicer state (e.g., make sure the desired Year is selected).

##Adding new years or hospitals
----------------------------
- Add rows to the data table with the new year(s), hospital entries, and values.
- Refresh queries / pivots.
- If using a separate Year lookup table, update it to include the new year(s).

##Common Issues & Troubleshooting
-------------------------------
- Scientific notation (e.g., 1.56353E+11) shown on chart labels:
  - Increase label number format precision or change number format to comma-separated currency (Format Data Labels → Number → Custom: #,##0).
- Slicer not filtering all charts:
  - Ensure all pivot tables are using the same pivot cache or connect slicer to each pivot (Slicer → Report Connections).
- Pivot chart mismatch or stale values:
  - Right-click pivot → Refresh. If the range has changed, confirm the pivot source range or re-bind to the named table.
- Slow performance with large datasets:
  - Use Power Query to pre-aggregate or load to the Data Model and use DAX measures instead of many individual pivot tables.
- Missing fields after data update:
  - Confirm new data contains the required columns with consistent header names (case/spacing matters).

##Formatting & Export Options
---------------------------
- Export to PDF: File → Export → Create PDF/XPS (choose portrait/landscape and scaling).
- Print: use Dashboard layout → Fit sheet on one page / adjust scaling.
- Copy chart images: Right-click chart → Copy → Paste into presentations/reports.

##Recommended Improvements 
-----------------------------------
- Add clear metric cards at top for single-number KPIs (Total Revenue, Total Patients, Net Revenue).
- Add tooltip or info icon that explains units (currency, patient counts).
- Use Power Pivot measures (DAX) for consistent measure definitions (e.g., MTD, QTD, YTD).
- Add date slicer supporting range selection (Start/End) if multi-year trend comparisons are needed.
- Include small data dictionary sheet describing each column and units.

##Security & Data Governance
--------------------------
- Keep sensitive data encrypted or in a secured network location.
- Avoid embedding unmasked PHI in distributed copies unless required and compliant with local regulations.
- Maintain a changelog when data transformations or KPI definitions change.

##Contact & Credits
-----------------
- Author / Maintainer: (vishwanreddypathuri@gmail.com)
- Dashboard created for: Health Care Analysis team
- For changes to the workbook structure, contact the workbook owner.

  <img width="1819" height="645" alt="Excel DashBoard" src="https://github.com/user-attachments/assets/555cb584-f067-4e29-825c-8a5cc7cc8aec" />
