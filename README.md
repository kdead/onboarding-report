# Onboarding Report: Where Does Onboarding Stall, and Which Workstream Feels It Most?

I rebuilt the onboarding process I managed at Accenture as a Power BI report.

This is a one page Power BI report built from a fictional 60 person onboarding dataset modeled on that real process. It answers one question: where does onboarding stall, and which workstream feels it most?

## Key insights

- 60 people in total, 49 onboarded and 11 in progress. That is 81.67% onboarded.
- Average time to onboard is 15.8 days, for onboarded people only, from RequestDate to StartDate. The range is 9 to 21 days.
- Finance Transformation takes the longest at 16.8 days. HR Operations is the shortest at 14.8 days. Supply Chain is 15.4 days and Technology Delivery is 16.0 days.
- Supply Chain has the most people still in progress, 4 of 15. Finance Transformation has the fewest, 1 of 15. HR Operations and Technology Delivery each have 3 in progress.
- The stall point is Role Created, with 5 of the 11 in progress people waiting there. Next is WBS Access Granted with 3, then Distribution Lists Added with 2, then Forecasting Done with 1.
- Starts by month peak in February and March, 11 each, and are lowest in September, 2.
- What if stage delays were cut? At a 3 day reduction, the scenario average falls to 12.8 days. At a 5 day reduction, it falls to 10.8 days. The scenario is a reduction to the overall average, not a reduction per stage.

## Dataset

File: `onboarding_dataset.csv`

Fictional data, 60 rows, 15 per workstream: Finance Transformation, Supply Chain, HR Operations, Technology Delivery.

Columns include PersonID, FictionalName, Workstream, TowerLead, RequestDate, DetailsConfirmedDate, RoleCreatedDate, ForecastingDoneDate, WBSAccessDate, DistListsAddedDate, StartDate, RollOffDate, CurrentStage, Status, and Location.

Stages follow the real process: details confirmed, role created, forecasting done, WBS access granted, distribution lists added, onboarded.

For the 11 in progress rows, StartDate is the planned start, RequestDate plus 28 days, and later stage dates are blank. That is intentional. It is what makes the stage bottleneck honest, and it is why the average only uses onboarded rows.

Validation tie outs, checked against the CSV:

- Total people: 60
- Onboarded: 49, In Progress: 11
- In progress by CurrentStage: Role Created 5, WBS Access Granted 3, Distribution Lists Added 2, Forecasting Done 1
- Avg Days to Onboard: 15.8 days, min 9, max 21, onboarded rows only
- No StartDate before its RequestDate. No RollOffDate on or before its StartDate.

## Data model

- `FactOnboarding`: the dataset above.
- `DimDate`: created with `CALENDARAUTO()`, with Date, Day, Month, Year, and Month Name columns. Month Name is `FORMAT(DimDate[Date], "mmmm")`, sorted by Month so it stays in calendar order. DimDate is marked as a date table.
- Active relationship: DimDate[Date] to FactOnboarding[StartDate]. StartDate is the main business date for onboarding timing, averages, and date slicers.
- Inactive relationship: DimDate[Date] to FactOnboarding[RollOffDate]. RollOffDate is secondary and is used only for the Roll-offs measure, with USERELATIONSHIP. Only one relationship can be active at a time.

## DAX measures

All measures live in FactOnboarding and were tied out against the CSV.

| Measure | Formula | Result, no filters |
| --- | --- | --- |
| Total People | `COUNTROWS(FactOnboarding)` | 60 |
| Onboarded Count | `CALCULATE([Total People], FactOnboarding[Status] = "Onboarded")` | 49 |
| In Progress Count | `CALCULATE([Total People], FactOnboarding[Status] = "In Progress")` | 11 |
| % Onboarded | `DIVIDE([Onboarded Count], [Total People])` | 81.67% |
| Avg Days to Onboard | `AVERAGEX(FILTER(FactOnboarding, FactOnboarding[Status] = "Onboarded"), DATEDIFF(FactOnboarding[RequestDate], FactOnboarding[StartDate], DAY))` | 15.8 |
| Roll-offs | `CALCULATE([Total People], USERELATIONSHIP(DimDate[Date], FactOnboarding[RollOffDate]))` | 60, with no date filter |
| Avg Days, Scenario | `[Avg Days to Onboard] - SELECTEDVALUE('Stage delay reduction'[Stage delay reduction])` | 15.8 at 0, 12.8 at 3, 10.8 at 5 |

Why AVERAGEX, not AVERAGE: AVERAGEX calculates DATEDIFF row by row first, then averages the result. AVERAGE would need a column that already holds the number of days. Averaging all 60 rows would inflate the result, because the 11 in progress rows only have a planned StartDate.

Why USERELATIONSHIP: Roll-offs counts by RollOffDate for that one measure, rather than counting only people who rolled off. With no date filter it is 60, because all 60 rows have a RollOffDate.

## Report page

One page, in this order:

- Title: Onboarding Report, with the decision question as the subtitle.
- KPI cards: Total People 60, Onboarded Count 49, In Progress Count 11, Avg Days to Onboard 15.80, % Onboarded 81.67%.
- Workstream slicer at the top.
- Onboarded vs In Progress by Workstream, stacked bar chart. Each bar totals 15. Finance Transformation 14 plus 1, HR Operations 12 plus 3, Technology Delivery 12 plus 3, Supply Chain 11 plus 4.
- In Progress Count by CurrentStage, bar chart. Role Created 5, WBS Access Granted 3, Distribution Lists Added 2, Forecasting Done 1.
- Total People by Month Name, column chart. January 4, February 11, March 11, April 5, May 6, June 9, July 6, August 6, September 2.
- Avg Days to Onboard by Workstream, bar chart. Finance Transformation 16.8, Technology Delivery 16.0, Supply Chain 15.4, HR Operations 14.8.
- What if section: Avg Days, Scenario card, plus the Stage delay reduction slicer, 0 to 5 in steps of 1, default 0.

## Row level security

Four roles, one per workstream, created in Modeling, Manage roles. Each role filters FactOnboarding[Workstream] to that workstream, for example `FactOnboarding[Workstream] = "Finance Transformation"`.

Tested with View as. Each role sees exactly 15 people.

Finance Transformation as tested: Total People 15, Onboarded Count 14, In Progress Count 1, % Onboarded 93.33%, Avg Days to Onboard 16.79, Avg Days, Scenario 16.79 at slicer 0. Supply Chain, HR Operations, and Technology Delivery each tested at 15 as well.

## Bookmarks

Two bookmarks, in the Bookmarks pane:

- Overview: no workstream filter, what if slicer at 0. This is the default view.
- Finance Transformation: Workstream slicer set to Finance Transformation only, the workstream with the longest average time to onboard.

Save the file on Overview so the report opens on the full view.

## What if parameter

Parameter name: Stage delay reduction. Numeric range, minimum 0, maximum 5, increment 1, default 0.

The scenario measure subtracts the selected value from the overall average. It does not shave that many days off every stage. At 0 the scenario is 15.8, at 1 it is 14.8, at 2 it is 13.8, at 3 it is 12.8, at 4 it is 11.8, and at 5 it is 10.8.

## Files

- `Onboarding_Report_Project3.pbix`: the report.
- `onboarding_dataset.csv`: the fictional dataset.
- `README.md`: this file.

## Limitations

- The dataset is fictional, so the findings describe this sample, not a real workforce.
- StartDate for in progress people is planned, not actual, so those 11 people are excluded from Avg Days to Onboard until their status changes to Onboarded.
- Roll-offs counts by RollOffDate for that measure only. It is not a count of people who left early.
- There is no Power BI Service tenant for this project, so publishing, workspaces, apps, refresh, and gateway behavior is conceptual only. Row level security was tested in Power BI Desktop with View as.
