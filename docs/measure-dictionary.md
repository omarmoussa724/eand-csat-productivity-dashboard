# Measure dictionary

The 25 DAX measures live in one empty table, `_Measures`, on top of the star schema
(`Fact_CSAT`, `Fact_Productivity` -> `Dim_Date`, `Dim_Roster`).

| Measure | DAX | Format | What it answers |
|---|---|---|---|
| Surveys | `COUNTROWS ( Fact_CSAT )` | Whole | Number of CFM survey responses |
| Avg CSAT | `AVERAGE ( Fact_CSAT[CSAT Score] )` | Decimal | Average CSAT score on the 1-5 scale |
| CSAT % | `DIVIDE ( CALCULATE ( [Surveys], Fact_CSAT[CSAT Score] >= 4 ), [Surveys] )` | Percent | Top-2-box: share of surveys scoring 4 or 5 |
| DSAT % | `DIVIDE ( CALCULATE ( [Surveys], Fact_CSAT[CSAT Score] <= 2 ), [Surveys] )` | Percent | Bottom-2-box: share of surveys scoring 1 or 2 |
| Avg CSAT (min 20) | `IF ( [Surveys] >= 20, [Avg CSAT] )` | Decimal | Avg CSAT only where the sample has at least 20 surveys (used for Team Leader ranking) |
| Avg Courtesy | `AVERAGE ( Fact_CSAT[Courtesy Score] )` | Decimal | Average Courtesy & Politeness sub-score (only surveys where it was asked) |
| Avg Knowledge | `AVERAGE ( Fact_CSAT[Knowledge Score] )` | Decimal | Average Knowledge sub-score (only surveys where it was asked) |
| Avg CSAT PM | `CALCULATE ( [Avg CSAT], DATEADD ( Dim_Date[Date], -1, MONTH ) )` | Decimal | Average CSAT for the previous month |
| CSAT MoM | `IF ( HASONEVALUE ( Dim_Date[Month Start] ) && NOT ISBLANK ( [Avg CSAT PM] ), [Avg CSAT] - [Avg CSAT PM] )` | Decimal | Change in Avg CSAT vs previous month (only when one month is selected) |
| CSAT vs Account | `IF ( NOT ISBLANK ( [Avg CSAT] ), [Avg CSAT] - CALCULATE ( [Avg CSAT], ALLSELECTED ( Dim_Roster ) ) )` | Decimal | Gap between this row's Avg CSAT and the account average for the current filters |
| Answered Calls | `SUM ( Fact_Productivity[Answered Calls] )` | Whole | Total ACD calls answered |
| Handle Time (s) | `SUM ( Fact_Productivity[Handle Time (s)] )` | Whole | Total handle time in seconds (talk + hold + ACW) |
| AHT (s) | `DIVIDE ( [Handle Time (s)], [Answered Calls] )` | Decimal | Average Handle Time = Handle Time / Answered Calls (seconds) |
| Avg Talk (s) | `DIVIDE ( SUM ( Fact_Productivity[Talk Time (s)] ), [Answered Calls] )` | Decimal | Average talk time per answered call (seconds) |
| Avg Hold (s) | `DIVIDE ( SUM ( Fact_Productivity[Hold Time (s)] ), [Answered Calls] )` | Decimal | Average hold time per answered call (seconds) |
| Avg ACW (s) | `DIVIDE ( SUM ( Fact_Productivity[ACW Time (s)] ), [Answered Calls] )` | Decimal | Average after-call work per answered call (seconds) |
| Occupancy % | `CALCULATE ( DIVIDE ( SUM ( Fact_Productivity[Handle Time (s)] ), SUM ( Fact_Productivity[Handle Time (s)] ) + SUM ( Fact_Productivity[Avail Time (s)] ) ), Fact_Productivity[Time Outlier] = FALSE () )` | Percent | Handle time / (handle + available time); rows with >16h staffed/avail excluded |
| Active Agents | `CALCULATE ( DISTINCTCOUNT ( Fact_Productivity[Agent ID] ), Fact_Productivity[Answered Calls] > 0 )` | Whole | Agents who answered at least one call |
| Calls per Agent Day | `DIVIDE ( [Answered Calls], CALCULATE ( COUNTROWS ( SUMMARIZE ( Fact_Productivity, Fact_Productivity[Agent ID], Fact_Productivity[Date] ) ), Fact_Productivity[Answered Calls] > 0 ) )` | Decimal | Answered calls per agent working day |
| % Calls Outside Home LOB | `DIVIDE ( CALCULATE ( [Answered Calls], FILTER ( Fact_Productivity, Fact_Productivity[Queue LOB] <> RELATED ( Dim_Roster[LOB] ) ) ), [Answered Calls] )` | Percent | Share of calls handled on the other LOB's queue (cross-skilling) |
| AHT PM | `CALCULATE ( [AHT (s)], DATEADD ( Dim_Date[Date], -1, MONTH ) )` | Decimal | AHT for the previous month |
| AHT MoM | `IF ( HASONEVALUE ( Dim_Date[Month Start] ) && NOT ISBLANK ( [AHT PM] ), [AHT (s)] - [AHT PM] )` | Decimal | Change in AHT vs previous month in seconds (only when one month is selected) |
| AHT vs Account | `IF ( NOT ISBLANK ( [AHT (s)] ), [AHT (s)] - CALCULATE ( [AHT (s)], ALLSELECTED ( Dim_Roster ) ) )` | Decimal | Gap between this row's AHT and the account AHT for the current filters (seconds) |
| Scatter CSAT | `IF ( [Surveys] >= 10 && [Answered Calls] > 0, [Avg CSAT] )` | Decimal | Avg CSAT for agents with >= 10 surveys and calls (for the CSAT vs AHT quadrant) |
| Scatter AHT | `IF ( [Surveys] >= 10 && [Answered Calls] > 0, [AHT (s)] )` | Decimal | AHT for agents with >= 10 surveys and calls (for the CSAT vs AHT quadrant) |
