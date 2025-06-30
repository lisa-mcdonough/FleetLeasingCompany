# 🧮 Fleet Leasing – DAX Measures Reference

This document outlines key DAX measures used in the Fleet Leasing Power BI report. The calculations support metrics across win/loss analysis, performance ratios, trend evaluation, and account insights.

---

## 🔍 Win/Loss Performance

- **TotalAmtWonHub**  
  Sum of amounts marked *Closed Won* by hub.

- **TotalAmtLostHub**  
  Sum of amounts marked *Closed Lost* by hub.

- **TotalWonAmount / TotalLostAmount**  
  Overall amount values for *Closed Won* and *Closed Lost* stages.

- **CountWonJobs / CountLostJobs / CountTotalJobsQuoted**  
  Job counts by deal outcome, used in ratio calculations and KPIs.

- **CountWinLossRatio / WinLossRatio# / WinLossRatio$**  
  Various win/loss ratios, with logic to display messages like “No Jobs Won” when counts are zero.

- **WinLossRatioSwitch-HASONEVALUE / WinLossRatioSwitch**  
  Dynamic text-based outputs that adapt based on year selection or win/loss state.

---

## 📈 Trend & Time-Based Measures

- **CumulativeLostJobs**  
  Cumulative count of closed lost jobs over time using the `ALL('Date')` pattern.

- **CountLostJobsOverTime**  
  Count of lost jobs using a date range (for line charts or forecasting).

- **PreviousYearAmtWon / %ChangeAmtWon**  
  Amount won for prior year and its % change vs. current.

---

## 🎯 Target & Adjustment Metrics

- **TargetLostAmt / AdjustedTargetLostAmt**  
  Targets based on user-selected reduction percentages.

- **TargetWonAmt / AdjustedWonAmount**  
  Targets for growth based on selected uplift values.

---

## 📊 Account, Facility & Commodity Insights

- **LostAmountByAccount / WonAmountByAccount / LostAmountByFacility**  
  Aggregated amounts tied to Accounts, Facilities, or Hubs.

- **FilteredAccounts**  
  Displays accounts with *Closed Lost* but no *Closed Won*.

- **RankAccountByLostAmount / RankAccountByWonAmount / RankCommodityByLostAmount**  
  Ranking logic to order performance by amount fields.

- **Total Commodities / Total Select Commodity**  
  Useful for calculating proportional contributions of each commodity.

---

## 🧩 UX & Labeling Measures

- **TitleSelectedProduct%Total / TitleSelectedProductLost / TitleSelectedProductWon**  
  Custom labels reflecting user selections in visuals.

- **Account Name / Opportunity Owner / JobNumber**  
  Text-based helpers for drillthrough, filters, and dynamic titles.

---

## 🔄 Miscellaneous

- **NoWinJobCount / NoWinsCount**  
  Conditional metrics for flagging underperformance.

- **CountLostJobsByFacility**  
  Useful for bar/column visuals across hubs.

- **SumCommodityTest2**  
  Test measure filtering *Closed Lost* and grouping by product.
