
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

After Organzing the data we will start making our train and test data.

``` r
test_inds = createDataPartition(y = 1:nrow(Y), p = 0.2, list = F)

X_test = X[test_inds, ]; Y_test = Y[test_inds,]
X_train = X[-test_inds, ]; Y_train = Y[-test_inds,]


colnames(Y_train) = c("day0.00", "day0.01",  "day0.02", "day0.03", "day0.04", "day0.05", "day0.06", "day0.07"
                , "day0.08", "day0.09", "day0.10", "day0.11", "day0.12", "day0.13", "day0.14", "day0.15",
                 "day0.16", "day0.17", "day0.18", "day0.19", "day0.20", "day0.21", "day0.22", "day0.23")

colnames(Y_test) = c("day0.00", "day0.01",  "day0.02", "day0.03", "day0.04", "day0.05", "day0.06", "day0.07"
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

colnames(X_test) = c("day7.00", "day7.01",  "day7.02", "day7.03", "day7.04", "day7.05", "day7.06", "day7.07"
                , "day7.08", "day7.09", "day7.10", "day7.11", "day7.12", "day7.13", "day7.14", "day7.15",
                 "day7.16", "day7.17", "day7.18", "day7.19", "day7.20", "day7.21", "day7.22", "day7.23",
                
                "day2.00", "day2.01",  "day2.02", "day2.03", "day2.04", "day2.05", "day2.06", "day2.07"
                , "day2.08", "day2.09", "day2.10", "day2.11", "day2.12", "day2.13", "day2.14", "day2.15",
                 "day2.16", "day2.17", "day2.18", "day2.19", "day2.20", "day2.21", "day2.22", "day2.23",
                
                "day1.00", "day1.01",  "day1.02", "day1.03", "day1.04", "day1.05", "day1.06", "day1.07"
                , "day1.08", "day1.09", "day1.10", "day1.11", "day1.12", "day1.13", "day1.14", "day1.15",
                 "day1.16", "day1.17", "day1.18", "day1.19", "day1.20", "day1.21", "day1.22", "day1.23")
dim(X_train)
## [1] 286  72
dim(X_test)
## [1] 72 72
dim(Y_train)
## [1] 286  24
dim(Y_test)
## [1] 72 24
```

Prediction
----------

### linear regression

Predict Load of tommorrow at hour i based on last 7 days at time i, last 2 days at time i and yesterday at time i where i is 0,1,2..23.

``` r
data00 = data.frame(Y_train$day0.00,X_train$day7.00,X_train$day2.00,X_train$day1.00)
colnames(data00) <- c('day0','pday7','pday2','pday1')
mod00 = lm(day0 ~., data = data00)

newdat = data.frame(pday7 = X_test$day7.00, pday2=X_test$day2.00, pday1=X_test$day1.00)
predict(mod00, newdata = newdat)
##        1        2        3        4        5        6        7        8 
## 47101.79 38059.60 40423.26 33729.86 37223.97 37042.99 36885.83 42080.69 
##        9       10       11       12       13       14       15       16 
## 32634.22 35331.29 37207.14 34172.04 33964.29 31295.35 33189.34 33234.56 
##       17       18       19       20       21       22       23       24 
## 34603.77 31859.52 36305.96 34737.34 32308.97 31739.48 33155.59 34586.92 
##       25       26       27       28       29       30       31       32 
## 37167.77 40436.08 42686.92 40622.46 42454.42 46830.57 44841.61 43539.50 
##       33       34       35       36       37       38       39       40 
## 42342.62 47025.74 48988.10 49635.94 43659.05 43409.59 46760.42 51241.48 
##       41       42       43       44       45       46       47       48 
## 42360.83 46507.06 46961.52 48218.63 46292.55 48109.32 48162.19 48848.21 
##       49       50       51       52       53       54       55       56 
## 48809.18 38192.71 41748.28 36136.49 42892.92 38485.54 40958.65 36341.30 
##       57       58       59       60       61       62       63       64 
## 37933.60 37294.58 33529.70 32061.62 31661.42 35682.32 34123.25 33169.59 
##       65       66       67       68       69       70       71       72 
## 39837.28 33239.85 33601.55 38027.95 34733.81 41822.21 40293.50 42288.25

#repeat this for the other times, then visualize the results
```
