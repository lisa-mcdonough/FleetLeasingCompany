🧲 Fleet Leasing – DAX Measures Reference

This document outlines key DAX measures used in the Fleet Leasing Power BI report. The calculations support metrics across win/loss analysis, performance ratios, trend evaluation, and account insights.

🔍 Win/Loss Performance

TotalAmtWonHubSum of amounts marked Closed Won by hub.

TotalAmtWonHub =	
        CALCULATE (
            SUM ( WonLostPipeline[Amount] ),
            WonLostPipeline[stage] = "Closed Won",
            FILTER ( WonLostPipeline, WonLostPipeline[Hub] = RELATED(Location[Hub])
            )

TotalAmtLostHubSum of amounts marked Closed Lost by hub.

TotalAmtLostHub =
        CALCULATE (
            SUM ( WonLostPipeline[Amount] ),
            WonLostPipeline[stage] = "Closed Lost",
            FILTER ( WonLostPipeline, WonLostPipeline[Hub] = RELATED(Location[Hub])
            )


TotalWonAmount / TotalLostAmountOverall amount values for Closed Won and Closed Lost stages.

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
    blank()
)

CountWonJobs / CountLostJobs / CountTotalJobsQuotedJob counts by deal outcome, used in ratio calculations and KPIs.

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

CountWinLossRatio / WinLossRatio# / WinLossRatio$Various win/loss ratios, with logic to display messages like “No Jobs Won” when counts are zero.

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

WinLossRatioSwitch-HASONEVALUE / WinLossRatioSwitchDynamic text-based outputs that adapt based on year selection or win/loss state.

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

📈 Trend & Time-Based Measures

CumulativeLostJobsCumulative count of closed lost jobs over time using the ALL('Date') pattern.

CumulativeLostJobs =
CALCULATE (
    [CountLostJobs],
    FILTER (
        ALL ( 'Date' ),
        'Date'[Date] <= MAX ( 'Date'[Date] )
    )
)

CountLostJobsOverTimeCount of lost jobs using a date range (for line charts or forecasting).

CountLostJobsOverTime =
CALCULATE (
    [CountLostJobs],
    DATESYTD ( 'Date'[Date] )
)

PreviousYearAmtWon / %ChangeAmtWonAmount won for prior year and its % change vs. current.

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

🌟 Target & Adjustment Metrics

TargetLostAmt / AdjustedTargetLostAmtTargets based on user-selected reduction percentages.

TargetLostAmt = [TotalLostAmount] * 0.9

AdjustedTargetLostAmt =
VAR Reduction = SELECTEDVALUE ( 'Target Settings'[Reduction %], 0.1 )
RETURN
    [TotalLostAmount] * ( 1 - Reduction )

TargetWonAmt / AdjustedWonAmountTargets for growth based on selected uplift values.

TargetWonAmt = [TotalWonAmount] * 1.1

AdjustedWonAmount =
VAR Uplift = SELECTEDVALUE ( 'Target Settings'[Uplift %], 0.1 )
RETURN
    [TotalWonAmount] * ( 1 + Uplift )

📊 Account, Facility & Commodity Insights

LostAmountByAccount / WonAmountByAccount / LostAmountByFacilityAggregated amounts tied to Accounts, Facilities, or Hubs.

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
LostAmountByFacility = 
CALCULATE(
    SUM(WonLostPipeline[Amount]),
    FILTER(
        WonLostPipeline,
        NOT(ISBLANK(WonLostPipeline[Hub])) && WonLostPipeline[Stage] = "Closed Lost"
    )
)

FilteredAccountsDisplays accounts with Closed Lost but no Closed Won.

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

RankAccountByLostAmount / RankAccountByWonAmount / RankCommodityByLostAmountRanking logic to order performance by amount fields.

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
    ,DESC
    ,Dense
   
)

RankCommodityByLostAmount = 
RANKX ( ALL ( WonLostPipeline[Product] ), [TotalLostAmount],, DESC, DENSE )


Total Commodities / Total Select CommodityUseful for calculating proportional contributions of each commodity.

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
🧩 UX & Labeling Measures

TitleSelectedProduct%Total / TitleSelectedProductLost / TitleSelectedProductWonCustom labels reflecting user selections in visuals.

TitleSelectedProduct%Total =
"%  of " & SELECTEDVALUE ( WonLostPipeline[Product] ) & " to all Products"

TitleSelectedProductLost =
"% Lost of " & SELECTEDVALUE(WonLostPipeline[Product])

TitleSelectedProductWon =
"% Won of " & SELECTEDVALUE(WonLostPipeline[Product])

Account Name / Opportunity Owner / JobNumberText-based helpers for drillthrough, filters, and dynamic titles.

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
🔄 Miscellaneous

NoWinJobCount / NoWinsCountConditional metrics for flagging underperformance.

NoWinJobCount =
IF([TotalWonAmount]=0,
 [CountLostJobs], BLANK())


CountLostJobsByFacilityUseful for bar/column visuals across hubs.

CountLostJobsByFacility =
CALCULATE (
    COUNTROWS ( WonLostPipeline ),
    WonLostPipeline[Stage] = "Closed Lost",
    WonLostPipeline[Hub]
)



