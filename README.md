
<!-- README.md is generated from README.Rmd. Please edit that file -->
Roadmap:

-data exploration

Electricity Load Prediction in Texas
====================================

predict 24 hour load using past history of demands with statistical methods

identify how wind energy affects energy forecasting

data visualizations

Introduction
------------

How do electric companies know how much power they have to generate?

But why is it important to predict hourly demand for electricity at least a day in advance? You need to know much generators needs to be on to meet the expected demand and turning on a generator requires time.

Libraries/packages we will be using
-----------------------------------

``` r
library(ggplot2)
## Warning: package 'ggplot2' was built under R version 3.5.2
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

<img src="README_figs/README-electricity graph-1.png" width="672" />

&lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD
=================================

Wind data

``` r
dfDemand$Windless_Load = dfDemand$ERCOT.Load..MW - dfDemand$Total.Wind.Output..MW
windDemand = dfDemand$Windless_Load
windOutput = dfDemand$Total.Wind.Output..MW

ggplot(dfDemand, aes(x = days, y = windOutput)) + geom_line(color = "orange") + 
  labs(title = "Texas Wind Power Output in 2018", x = "Days in 2018", y = "MegaWatts")
```

<img src="README_figs/README-wind output graph-1.png" width="672" />

Wind Power looks very sporadic while electricity demands seems to have a trend.

Demand Prediction Strategy and Data Aggregation
-----------------------------------------------

For our independent variables we will use past week, past 2 days, past 1 day to predict the electiricty demand of tomorrow. i.e days to train on -7, -2, -1

``` r
daysToTrainOn = c(-7,-2,-1)
rangeOfDays = seq(-min(daysToTrainOn), numberOfDays - 1, by = 1)

Y = NULL
for (day in rangeOfDays) {
  Y = rbind(Y, dfDemand$ERCOT.Load..MW[(day * 24): ((day + 1) * 24 - 1)])
}

X = NULL
for (day in rangeOfDays) {
  X_temp = cbind(t(dfDemand$ERCOT.Load..MW[(((day - 7)*24 +1)):((day - 7 + 1)*24)]),
            t(dfDemand$ERCOT.Load..MW[(((day - 2)*24) +1):((day - 2 + 1)*24)]),
            t(dfDemand$ERCOT.Load..MW[(((day - 1)*24) +1):((day - 1 + 1)*24)]))
  X = rbind(X, X_temp)
}
dim(X)
## [1] 358  72
dim(Y)
## [1] 358  24
```

After Organzing the data we will start making our train and test data.

``` r
library(caret)
## Warning: package 'caret' was built under R version 3.5.3
## Loading required package: lattice
test_inds = createDataPartition(y = 1:nrow(Y), p = 0.2, list = F)
X_test = X[test_inds, ]; Y_test = Y[test_inds]
X_train = X[-test_inds, ]; Y_train = Y[-test_inds]
```

Prediction
----------

> > > > > > > 4fc9e8ac32cd46c0045b9a45ff2ebf6db356f7c6
