
<!-- README.md is generated from README.Rmd. Please edit that file -->
Electricity Load Prediction in Texas
====================================

predict 24 hour load using past history of demands with statistical methods

identify how wind energy affects energy forecasting

data visualizations

Introduction
------------

Libraries/packages we will be using
-----------------------------------

``` r
library(ggplot2)
```

Load the ERCOT 2018 data
------------------------

``` r
dfDemand = read.csv("ERCOT_2018_Hourly_Wind_Output.csv")
demands = dfDemand$ERCOT.Load..MW
numberOfDays = length(demands)/24
```

Let's see how does load vary over the year in Texas.

``` r
days = vector(length = numberOfDays * 24)
for (hour in seq_len(numberOfDays * 24)) {
  days[hour] = hour / 24 
}

ggplot(dfDemand, aes(x = days, y = demands)) + geom_line(color = "dodgerblue") + 
  labs(title = "Texas Electricity Demands in 2018", x = "Days in 2018", y = "Net Demand of Texas (in MW)") 
```

<img src="README_figs/README-unnamed-chunk-4-1.png" width="672" />
