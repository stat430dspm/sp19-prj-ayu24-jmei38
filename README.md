
<!-- README.md is generated from README.Rmd. Please edit that file -->
Roadmap:

predict electricity load\* with and without wind power using linear and non-linear methods,

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
new_cran_packages <- c("ggplot2", "caret","stringr")
existing_packages <- installed.packages()[,"Package"]
missing_packages <- new_cran_packages[!(new_cran_packages %in% existing_packages)]
if(length(missing_packages)){
    install.packages(missing_packages)
}

library(ggplot2)
library(stringr)
library(caret)
require(neuralnet)
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

Y = data.frame(Y)
X = data.frame(X)
```

Windless
--------

``` r
#daysToTrainOn = c(-7,-2,-1)
#rangeOfDays = seq(-min(daysToTrainOn), numberOfDays - 1, by = 1)

#Y = NULL
#for (day in rangeOfDays) {
 # Y = rbind(Y, dfDemand$Windless_Load[(day * 24): ((day + 1) * 24 - 1)])
#}

#X = NULL
#for (day in rangeOfDays) {
 # X_temp = cbind(t(dfDemand$Windless_Load[(((day - 7)*24 +1)):((day - 7 + 1)*24)]),
  #          t(dfDemand$Windless_Load[(((day - 2)*24) +1):((day - 2 + 1)*24)]),
   #         t(dfDemand$Windless_Load[(((day - 1)*24) +1):((day - 1 + 1)*24)]))
  #X = rbind(X, X_temp)
#}
#dim(X)
#dim(Y)
```

After Organzing the data we will start making our train and test data. \*talk about normalizing the data

``` r
test_inds = createDataPartition(y = 1:nrow(Y), p = 0.2, list = F)

X_test = X[test_inds, ]; Y_test = Y[test_inds,]
X_train = X[-test_inds, ]; Y_train = Y[-test_inds,]

X_train_scaled = scale(X_train)
X_test_scaled = scale(X_test, center=attr(X_train_scaled, "scaled:center"), 
                              scale=attr(X_train_scaled, "scaled:scale"))

mean(X_train_scaled); mean(X_test_scaled)
## [1] -6.02314e-17
## [1] 0.09678187
sd(X_train_scaled); sd(X_test_scaled)
## [1] 0.9982745
## [1] 1.033151

colnames(Y_train) = c("day0.00", "day0.01",  "day0.02", "day0.03", "day0.04", "day0.05", "day0.06", "day0.07"
                , "day0.08", "day0.09", "day0.10", "day0.11", "day0.12", "day0.13", "day0.14", "day0.15",
                 "day0.16", "day0.17", "day0.18", "day0.19", "day0.20", "day0.21", "day0.22", "day0.23")

colnames(X_train) = c("day7.00", "day7.01",  "day7.02", "day7.03", "day7.04", "day7.05", "day7.06", "day7.07"
                , "day7.08", "day7.09", "day7.10", "day7.11", "day7.12", "day7.13", "day7.14", "day7.15",
                 "day7.16", "day7.17", "day7.18", "day7.19", "day7.20", "day7.21", "day7.22", "day7.23",
                
                "day2.00", "day2.01",  "day2.02", "day2.03", "day2.04", "day2.05", "day2.06", "day2.07"
                , "day2.08", "day2.09", "day2.10", "day2.11", "day2.12", "day2.13", "day2.14", "day2.15",
                 "day2.16", "day2.17", "day2.18", "day2.19", "day2.20", "day2.21", "day2.22", "day2.23",
                
                "day1.00", "day1.01",  "day1.02", "day1.03", "day1.04", "day1.05", "day1.06", "day1.07"
                , "day1.08", "day1.09", "day1.10", "day1.11", "day1.12", "day1.13", "day1.14", "day1.15",
                 "day1.16", "day1.17", "day1.18", "day1.19", "day1.20", "day1.21", "day1.22", "day1.23")
```

Prediction
----------

### linear regression

Predict Load of tommorrow at hour i based on last 7 days at time i, last 2 days at time i and yesterday at time i where i is 0,1,2..23.

``` r
lm(Y_train$day0.00 ~ X_train$day7.00 + X_train$day2.00 + X_train$day1.00)
## 
## Call:
## lm(formula = Y_train$day0.00 ~ X_train$day7.00 + X_train$day2.00 + 
##     X_train$day1.00)
## 
## Coefficients:
##     (Intercept)  X_train$day7.00  X_train$day2.00  X_train$day1.00  
##       2922.8986           0.1044          -0.2283           1.1028
```
