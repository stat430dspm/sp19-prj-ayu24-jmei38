
<!-- README.md is generated from README.Rmd. Please edit that file -->
Roadmap:

predict electricity load\* with and without wind power using linear and non-linear methods (neural network, svm?),

report results/conclusions

-data exploration,data visualizations (make heatmap)

\*predict 24 hour load using past history of demands and identify how wind energy affects energy forecasting \# Electricity Load Prediction in Texas

Introduction
------------

How do electric companies know how much power they have to generate?

But why is it important to predict hourly demand for electricity at least a day in advance? You need to know much generators needs to be on to meet the expected demand and turning on a generator requires time.

Libraries/packages we will be using
-----------------------------------

``` r
library(ggplot2)
library(caret)
library(dplyr)
library(stringr)
```

Load the ERCOT 2018 data
------------------------

Let's see how does load vary over the year in Texas. <img src="README_figs/README-electricity graph-1.png" width="672" />

Wind data
---------

<img src="README_figs/README-wind output graph-1.png" width="672" />

Wind Power looks very sporadic while electricity demands seems to have a trend.

<img src="README_figs/README-January HeatMap-1.png" width="672" /> <img src="README_figs/README-July HeatMap-1.png" width="672" />

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

Windless
--------

``` r
daysToTrainOn = c(-7,-2,-1)
rangeOfDays = seq(-min(daysToTrainOn), numberOfDays - 1, by = 1)

Y = NULL
for (day in rangeOfDays) {
  Y = rbind(Y, dfDemand$Windless_Load[(day * 24): ((day + 1) * 24 - 1)])
}

X = NULL
for (day in rangeOfDays) {
  X_temp = cbind(t(dfDemand$Windless_Load[(((day - 7)*24 +1)):((day - 7 + 1)*24)]),
            t(dfDemand$Windless_Load[(((day - 2)*24) +1):((day - 2 + 1)*24)]),
            t(dfDemand$Windless_Load[(((day - 1)*24) +1):((day - 1 + 1)*24)]))
  X = rbind(X, X_temp)
}
dim(X)
## [1] 358  72
dim(Y)
## [1] 358  24
```

After Organzing the data we will start making our train and test data. \*talk about normalizing the data

``` r
test_inds = createDataPartition(y = 1:nrow(Y), p = 0.2, list = F)
X_test = X[test_inds, ]; Y_test = Y[test_inds]
X_train = X[-test_inds, ]; Y_train = Y[-test_inds]

X_train_scaled = scale(X_train)
X_test_scaled = scale(X_test, center=attr(X_train_scaled, "scaled:center"), 
                              scale=attr(X_train_scaled, "scaled:scale"))

mean(X_train_scaled); mean(X_test_scaled)
## [1] 3.99837e-17
## [1] 0.03969436
sd(X_train_scaled); sd(X_test_scaled)
## [1] 0.9982745
## [1] 1.040938
```

Prediction
----------

### try neural network

``` r
#lin <- lm(Y_train ~ X_train_scaled)
#summary(lin)
```
