# 🧮 Fleet Leasing – DAX Measures Reference

This document outlines key DAX measures used in the Fleet Leasing Power BI report. The calculations support metrics across win/loss analysis, performance ratios, trend evaluation, and account insights.

---

## 🔍 Win/Loss Performance

| **Measure**                        | **Description**                                                           |
|------------------------------------|---------------------------------------------------------------------------|
| `TotalAmtWonHub`                  | Sum of opportunity amounts marked *Closed Won* by Hub                     |
| `TotalAmtLostHub`                 | Sum of opportunity amounts marked *Closed Lost* by Hub                    |
| `TotalWonAmount` / `TotalLostAmount` | Totals for all Won and Lost opportunities                                |
| `CountWonJobs` / `CountLostJobs` / `CountTotalJobsQuoted` | Job counts by outcome: won, lost, quoted                  |
| `CountWinLossRatio` / `WinLossRatio#` / `WinLossRatio$` | Various win/loss ratios with fallback messaging on zero counts          |
| `WinLossRatioSwitch` / `WinLossRatioSwitch-HASONEVALUE` | Dynamic messages adjusting by selection context                         |

---

## 📈 Trend & Time Intelligence

| **Measure**            | **Description**                                                          |
|------------------------|--------------------------------------------------------------------------|
| `CumulativeLostJobs`   | Accumulates *Closed Lost* jobs over time using `ALL('Date')`             |
| `CountLostJobsOverTime` | Filtered count for *Closed Lost* jobs, used in line charts or forecasting |
| `PreviousYearAmtWon`   | Prior year’s total won amount                                            |
| `%ChangeAmtWon`        | Year-over-year change in amount won                                      |

---

## 🎯 Target & Goal Adjustments

| **Measure**             | **Description**                                                                 |
|-------------------------|---------------------------------------------------------------------------------|
| `TargetLostAmt`         | Target reduction value for Lost Amount                                          |
| `AdjustedTargetLostAmt` | Final Lost Amount target after applying user-defined reduction %                |
| `TargetWonAmt`          | Projected goal for increased Won Amount                                         |
| `AdjustedWonAmount`     | Final Won target adjusted for user-specified uplift                            |

---

## 🏢 Account, Facility & Commodity Metrics

| **Measure**                  | **Description**                                                       |
|------------------------------|-----------------------------------------------------------------------|
| `LostAmountByAccount`        | Sum of *Closed Lost* amounts grouped by Account                       |
| `WonAmountByAccount`         | Sum of *Closed Won* amounts grouped by Account                        |
| `LostAmountByFacility`       | Facility-level breakdown of lost opportunity amounts                  |
| `FilteredAccounts`           | Filters to accounts with Lost but no Won opportunities                |
| `RankAccountByLostAmount`    | Rank by amount lost per account                                       |
| `RankAccountByWonAmount`     | Rank by amount won per account                                        |
| `RankCommodityByLostAmount`  | Commodity-level rank by lost amount                                   |
| `Total Commodities` / `Total Select Commodity` | Used for proportional shares in visualizations              |

---

## 🧩 UX & Labeling Measures

| **Measure**                     | **Description**                                                    |
|----------------------------------|--------------------------------------------------------------------|
| `TitleSelectedProduct%Total`    | Custom label: selected product as % of total                       |
| `TitleSelectedProductLost` / `TitleSelectedProductWon` | Visual titles that adjust with user filters         |
| `Account Name` / `Opportunity Owner` / `JobNumber` | Text-based helpers for drillthrough and filtering       |

---

## 🧠 Performance Flags & Helpers

| **Measure**            | **Description**                                                   |
|------------------------|-------------------------------------------------------------------|
| `NoWinJobCount`        | Conditional count where jobs were quoted but never won            |
| `NoWinsCount`          | Aggregated underperformance alert logic                          |
| `CountLostJobsByFacility` | Used in comparative visuals by Hub or location                |
| `SumCommodityTest2`    | Filtered total for *Closed Lost* grouped by product (test logic)  |
