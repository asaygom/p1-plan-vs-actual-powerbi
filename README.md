# P1 — Plan vs Actual Dashboard (Power BI)

**Goal.** Weekly view of *planned vs actual* work hours with variance and a simple **pace index** for engineering/construction projects.  
**Stack.** Power BI (DAX, modeling) · Power Query · CSV demo data.  
**Deliverables.** `.pbix` + README + screenshots/PDF.

---

## 1) Repository structure
```
p1-plan-vs-actual-powerbi/
├─ README.md
├─ asaygom_p1.pbix                # (add when ready)
├─ /data
│  ├─ activities.csv              # sample or real data
│  └─ date.csv                    # weekly calendar (Sunday start)
└─ /assets
   ├─ screenshot-01.png
   ├─ screenshot-02.png
   └─ report.pdf                  # optional, exported from PBIX
```
> Start with the demo CSVs in `/data` and replace later with real exports.

---

## 2) Data schema

### 2.1 `activities.csv` (fact)
Required columns:
- `ActivityID` (key), `WBS`, `Discipline`, `Area/Room`  
- `BaselineStart`, `BaselineFinish`, `ActualStart`, `ActualFinish` (ISO `YYYY-MM-DD`)  
- `PlannedHours`, `ActualHours`, `PercentComplete` (numbers)

### 2.2 `date.csv` (date dimension, weekly)
Columns:
- `Date` (Sunday for each week), `FridayLabel` (Date + 5), `Week`, `Year`

> Use `FridayLabel` on the X-axis to display “week ending Friday”.

---

## 3) Data model (star)
- **FactActivities** ← `activities.csv`  
- **DimDate** ← `date.csv`  
- **DimDiscipline** (optional, from `Discipline`)  
- **DimArea** (optional, from `Area/Room`)

Relationships:
- `FactActivities[BaselineStart]` (or a derived week key) → `DimDate[Date]` (many-to-one)  
- Alternatively, create a **WeekKey** in both tables for a stable weekly join.

---

## 4) DAX measures (base)
```DAX
-- Hours
Total Planned HH = SUM(FactActivities[PlannedHours])
Total Actual HH  = SUM(FactActivities[ActualHours])
HH Variance      = [Total Actual HH] - [Total Planned HH]

-- Cumulative curves (S-curve)
Cum Planned HH =
CALCULATE(
    [Total Planned HH],
    FILTER(ALL('DimDate'[Date]), 'DimDate'[Date] <= MAX('DimDate'[Date]))
)

Cum Actual HH =
CALCULATE(
    [Total Actual HH],
    FILTER(ALL('DimDate'[Date]), 'DimDate'[Date] <= MAX('DimDate'[Date]))
)

-- Pace index (proxy of schedule performance based on hours)
Pace Index (SPI proxy) = DIVIDE([Cum Actual HH], [Cum Planned HH])

-- Optional extras
HH Variance % = DIVIDE([HH Variance], [Total Planned HH])
Planned HH (Selected) = [Total Planned HH]
Actual HH (Selected)  = [Total Actual HH]
```

If you build `DimDate` inside Power BI:
```DAX
FridayLabel = 'DimDate'[Date] + 5
```

---

## 5) Visuals (suggested)
1. **KPIs**: Planned HH, Actual HH, Variance HH, Pace Index.  
2. **S-curve**: `Cum Planned HH` vs `Cum Actual HH` over `DimDate[FridayLabel]`.  
3. **Variance by discipline**: stacked/clustered weekly bars.  
4. **Top 10 deviations**: table (WBS/Area/ActivityID).  
5. **Heatmap**: Week × Discipline by `HH Variance`.

Slicers: `Discipline`, `Area/Room`, `WBS`, Week range.

---

## 6) How to use

### Option A — Start from demo data
1. Copy `/data/activities.csv` and `/data/date.csv` into the repo.  
2. Create/open `asaygom_p1.pbix` → **Get Data → Text/CSV** → load both files.  
3. Build relationships (FactActivities → DimDate).  
4. Add the DAX measures and visuals.  
5. Export **PDF** and screenshots to `/assets`.

### Option B — With your own exports
1. Export activity/progress from your system (P6/Excel/DB) to a CSV with the required columns.  
2. Replace `/data/activities.csv`.  
3. Keep `/data/date.csv` (or regenerate it to match your range).  
4. Refresh and validate KPIs.

---

## 7) Optional — Generate `DimDate` with Power Query (M)
```m
let
    StartDate = #date(2025, 7, 6),         // Sunday
    Weeks     = 26,                         // adjust
    Source    = List.Transform({0..Weeks-1}, each Date.AddWeeks(StartDate, _)),
    Tbl       = Table.FromList(Source, Splitter.SplitByNothing(), {"Date"}),
    AddWeek   = Table.AddColumn(Tbl, "Week", each Date.WeekOfYear([Date], Day.Sunday)),
    AddYear   = Table.AddColumn(AddWeek, "Year", each Date.Year([Date])),
    AddFri    = Table.AddColumn(AddYear, "FridayLabel", each Date.AddDays([Date], 5))
in
    AddFri
```

---

## 8) Performance & quality tips
- Keep numeric columns as **Whole/Decimal Number**; parse dates as **Date**.  
- Hide technical columns (keys) from report view.  
- Use **ALLSELECTED** instead of **ALL** in cumulative measures if you need slicer-aware curves.  
- Version your `.pbix` via exported **PBIT** + data folder (optional).

---

## 9) Screenshots / Report
Place exported PNGs in `/assets` and (optional) a `report.pdf`.

---

## 10) License
MIT (or your choice). Add a `LICENSE` file if needed.

---

## 11) Changelog / Roadmap (optional)
- v0.1 — Initial model & measures  
- Next — Add drillthrough to Activity, variance thresholds, area heatmap
