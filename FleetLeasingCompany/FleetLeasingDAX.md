# 🧲 Fleet Leasing – DAX Measures Reference

This document outlines key DAX measures used in the Fleet Leasing Power BI report. The calculations support metrics across win/loss analysis, performance ratios, trend evaluation, and account insights.

---

## 🔍 Win/Loss Performance

### **TotalAmtWonHub**  
Sum of amounts marked *Closed Won* by hub.

```DAX
TotalAmtWonHub =	
    CALCULATE (
        SUM ( WonLostPipeline[Amount] ),
        WonLostPipeline[stage] = "Closed Won",
        FILTER ( WonLostPipeline, WonLostPipeline[Hub] = RELATED(Location[Hub]) )
    )
```

### **TotalAmtLostHub**  
Sum of amounts marked *Closed Lost* by hub.

```DAX
TotalAmtLostHub =
    CALCULATE (
        SUM ( WonLostPipeline[Amount] ),
        WonLostPipeline[stage] = "Closed Lost",
        FILTER ( WonLostPipeline, WonLostPipeline[Hub] = RELATED(Location[Hub]) )
    )
```

### **TotalWonAmount / TotalLostAmount**  
Overall amount values for Closed Won and Closed Lost stages.

```DAX
TotalWonAmount = 
COALESCE (
    CALCULATE (
        SUM ( WonLostPipeline[Amount] ),
        WonLostPipeline[stage] = "Closed Won"
    ),
    BLANK()
)

TotalLostAmount = COALESCE (
    CALCULATE (
        SUM ( WonLostPipeline[Amount] ),
        WonLostPipeline[stage] = "Closed Lost"
    ),
    BLANK()
)
```

### **CountWonJobs / CountLostJobs / CountTotalJobsQuoted**  
Job counts by deal outcome, used in ratio calculations and KPIs.

```DAX
CountWonJobs =
CALCULATE (
    COUNTROWS ( WonLostPipeline ),
    WonLostPipeline[Status] = "Closed Won"
)

CountLostJobs = 
CALCULATE(
    COUNTROWS(VALUES(WonLostPipeline[Job Number])),
    FILTER(WonLostPipeline, WonLostPipeline[Stage]="Closed Lost")
)

CountTotalJobsQuoted = 
CALCULATE (
    COUNT ( WonLostPipeline[Job Number] ),
    WonLostPipeline[Stage] IN { "Closed Won", "Closed Lost" }
)
```

### **CountWinLossRatio / WinLossRatio# / WinLossRatio$**  
Various win/loss ratios, with logic to display messages like “No Jobs Won” when counts are zero.

```DAX
CountWinLossRatio =
DIVIDE ( [CountWonJobs], [CountLostJobs] )

WinLossRatio# =
VAR Won = [CountWonJobs]
VAR Lost = [CountLostJobs]
RETURN
    IF (
        Won = 0,
        "No Jobs Won",
        FORMAT ( DIVIDE ( Won, Lost ), "0.00" )
    )

WinLossRatio$ =
VAR WonAmt = [TotalWonAmount]
VAR LostAmt = [TotalLostAmount]
RETURN
    IF (
        WonAmt = 0,
        "No Revenue Won",
        FORMAT ( DIVIDE ( WonAmt, LostAmt ), "0.00" )
    )
```

### **WinLossRatioSwitch-HASONEVALUE / WinLossRatioSwitch**  
Dynamic text-based outputs that adapt based on year selection or win/loss state.

```DAX
WinLossRatioSwitch-HASONEVALUE =
SWITCH(
    TRUE(),
    HASONEVALUE('Date'[Year]), "Single Year Selected",
    "Multi-Year View"
)

WinLossRatioSwitch =
VAR Won = [CountWonJobs]
VAR Lost = [CountLostJobs]
RETURN
    SWITCH(
        TRUE(),
        Won = 0, "No Wins",
        Lost = 0, "No Losses",
        FORMAT(DIVIDE(Won, Lost), "0.0") & " Win/Loss Ratio"
    )
```
... (Truncated due to size, continuation follows in next cell)

## 📈 Trend & Time-Based Measures

### **CumulativeLostJobs**  
Cumulative count of closed lost jobs over time using the ALL('Date') pattern.

```DAX
CumulativeLostJobs =
CALCULATE (
    [CountLostJobs],
    FILTER (
        ALL ( 'Date' ),
        'Date'[Date] <= MAX ( 'Date'[Date] )
    )
)
```

### **CountLostJobsOverTime**  
Count of lost jobs using a date range (for line charts or forecasting).

```DAX
CountLostJobsOverTime =
CALCULATE (
    [CountLostJobs],
    DATESYTD ( 'Date'[Date] )
)
```

### **PreviousYearAmtWon / %ChangeAmtWon**  
Amount won for prior year and its % change vs. current.

```DAX
PreviousYearAmtWon =
CALCULATE (
    [TotalWonAmount],
    SAMEPERIODLASTYEAR ( 'Date'[Date] )
)

%ChangeAmtWon =
DIVIDE (
    [TotalWonAmount] - [PreviousYearAmtWon],
    [PreviousYearAmtWon]
)
```

## 🌟 Target & Adjustment Metrics

### **TargetLostAmt / AdjustedTargetLostAmt**  
Targets based on user-selected reduction percentages.

```DAX
TargetLostAmt = [TotalLostAmount] * 0.9

AdjustedTargetLostAmt =
VAR Reduction = SELECTEDVALUE ( 'Target Settings'[Reduction %], 0.1 )
RETURN
    [TotalLostAmount] * ( 1 - Reduction )
```

### **TargetWonAmt / AdjustedWonAmount**  
Targets for growth based on selected uplift values.

```DAX
TargetWonAmt = [TotalWonAmount] * 1.1

AdjustedWonAmount =
VAR Uplift = SELECTEDVALUE ( 'Target Settings'[Uplift %], 0.1 )
RETURN
    [TotalWonAmount] * ( 1 + Uplift )
```

## 📊 Account, Facility & Commodity Insights

### **LostAmountByAccount / WonAmountByAccount / LostAmountByFacility**  
Aggregated amounts tied to Accounts, Facilities, or Hubs.

```DAX
LostAmountByAccount = 
CALCULATE(
    SUM(WonLostPipeline[Amount]),
    WonLostPipeline[Stage] = "Closed Lost",
    FILTER(
        VALUES(WonLostPipeline[Account Name]),
        NOT ISBLANK(WonLostPipeline[Account Name]) &&
        CALCULATE(SUM(WonLostPipeline[Amount]), WonLostPipeline[Stage] = "Closed Lost") <> 0
    )
)

WonAmountByAccount = 
CALCULATE(
    SUM(WonLostPipeline[Amount]),
    WonLostPipeline[Stage]= "Closed Won",
    FILTER(
        VALUES(WonLostPipeline[Account Name]),
        NOT ISBLANK(WonLostPipeline[Account Name]) &&
        CALCULATE(SUM(WonLostPipeline[Amount]), WonLostPipeline[Stage] = "Closed Won") <> 0
    )
)

LostAmountByFacility = 
CALCULATE(
    SUM(WonLostPipeline[Amount]),
    FILTER(
        WonLostPipeline,
        NOT(ISBLANK(WonLostPipeline[Hub])) && WonLostPipeline[Stage] = "Closed Lost"
    )
)
```
... (Will finish and append the rest in the next step)

### **FilteredAccounts**  
Displays accounts with Closed Lost but no Closed Won.

```DAX
FilteredAccounts = 
VAR AccountsWithClosedWon =
    SELECTCOLUMNS (
        FILTER ( WonLostPipeline, WonLostPipeline[Stage] = "Closed Won" ),
        "Account Name", WonLostPipeline[Account Name]
    )
RETURN
EXCEPT (
    VALUES ( WonLostPipeline[Account Name] ),
    AccountsWithClosedWon
)
```

### **RankAccountByLostAmount / RankAccountByWonAmount / RankCommodityByLostAmount**  
Ranking logic to order performance by amount fields.

```DAX
RankAccountByLostAmount = 
RANKX(
    ALL(WonLostPipeline[Account Name]),
    [LostAmountByAccount],
    ,
    DESC,
    Dense
)

RankAccountByWonAmount = 
RANKX(
    ALL(WonLostPipeline[Account Name]),
    [WonAmountByAccount],
    ,DESC,
    Dense
)

RankCommodityByLostAmount = 
RANKX (
    ALL ( WonLostPipeline[Product] ),
    [TotalLostAmount],
    ,
    DESC,
    DENSE
)
```

### **Total Commodities / Total Select Commodity**  
Useful for calculating proportional contributions of each commodity.

```DAX
Total Commodities = 
CALCULATE ( 
    SUM ( WonLostPipeline[Amount] ), 
    ALL ( WonLostPipeline[Product] ) 
)

Total Select Commodity =
CALCULATE (
    SUM ( WonLostPipeline[Amount] ),
    WonLostPipeline[Product] = SELECTEDVALUE ( WonLostPipeline[Product] )
)
```

## 🧩 UX & Labeling Measures

### **TitleSelectedProduct%Total / TitleSelectedProductLost / TitleSelectedProductWon**  
Custom labels reflecting user selections in visuals.

```DAX
TitleSelectedProduct%Total =
"%  of " & SELECTEDVALUE ( WonLostPipeline[Product] ) & " to all Products"

TitleSelectedProductLost =
"% Lost of " & SELECTEDVALUE(WonLostPipeline[Product])

TitleSelectedProductWon =
"% Won of " & SELECTEDVALUE(WonLostPipeline[Product])
```

### **Account Name / Opportunity Owner / JobNumber**  
Text-based helpers for drillthrough, filters, and dynamic titles.

```DAX
Account Name = 
SELECTEDVALUE(WonLostPipeline[Account Name])

Opportunity Owner = CONCATENATEX
    (VALUES(WonLostPipeline[Opportunity Owner]),
        WonLostPipeline[Opportunity Owner], 
        ", ", 
        WonLostPipeline[Opportunity Owner]
)

JobNumber =
IF (
    [TotalWonAmount] <> 0
        && [TotalLostAmount] <> 0,
    SELECTEDVALUE ( WonLostPipeline[Job Number] )
)
```

## 🔄 Miscellaneous

### **NoWinJobCount / NoWinsCount**  
Conditional metrics for flagging underperformance.

```DAX
NoWinJobCount =
IF([TotalWonAmount]=0,
 [CountLostJobs], BLANK())
```

### **CountLostJobsByFacility**  
Useful for bar/column visuals across hubs.

```DAX
CountLostJobsByFacility =
CALCULATE (
    COUNTROWS ( WonLostPipeline ),
    WonLostPipeline[Stage] = "Closed Lost",
    WonLostPipeline[Hub]
)
```
