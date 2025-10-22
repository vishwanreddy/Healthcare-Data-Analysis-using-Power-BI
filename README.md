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

Schedule:
- For frequently updated data, set up a scheduled refresh in Power BI Service (daily/hourly depending on license and gateway).

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
