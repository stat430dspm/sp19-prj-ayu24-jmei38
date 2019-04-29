
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

Predict Load of tommorrow at hour i based on last 7 days at time i, last 2 days at time i and yesterday at time i where *i* ∈ 1, 2, ...23

``` r
#predict the load at each hour using linear regression

#retrieve training data at hour i
data00 = data.frame(Y_train$day0.00,X_train$day7.00,X_train$day2.00,X_train$day1.00)
data01 = data.frame(Y_train$day0.01,X_train$day7.01,X_train$day2.01,X_train$day1.01)
data02 = data.frame(Y_train$day0.02,X_train$day7.02,X_train$day2.02,X_train$day1.02)
data03 = data.frame(Y_train$day0.03,X_train$day7.03,X_train$day2.03,X_train$day1.03)
data04 = data.frame(Y_train$day0.04,X_train$day7.04,X_train$day2.04,X_train$day1.04)
data05 = data.frame(Y_train$day0.05,X_train$day7.05,X_train$day2.05,X_train$day1.05)
data06 = data.frame(Y_train$day0.06,X_train$day7.06,X_train$day2.06,X_train$day1.06)
data07 = data.frame(Y_train$day0.07,X_train$day7.07,X_train$day2.07,X_train$day1.07)
data08 = data.frame(Y_train$day0.08,X_train$day7.08,X_train$day2.08,X_train$day1.08)
data09 = data.frame(Y_train$day0.09,X_train$day7.09,X_train$day2.09,X_train$day1.09)
data10 = data.frame(Y_train$day0.10,X_train$day7.10,X_train$day2.10,X_train$day1.10)
data11 = data.frame(Y_train$day0.11,X_train$day7.11,X_train$day2.11,X_train$day1.11)
data12 = data.frame(Y_train$day0.12,X_train$day7.12,X_train$day2.12,X_train$day1.12)
data13 = data.frame(Y_train$day0.13,X_train$day7.13,X_train$day2.13,X_train$day1.13)
data14 = data.frame(Y_train$day0.14,X_train$day7.14,X_train$day2.14,X_train$day1.14)
data15 = data.frame(Y_train$day0.15,X_train$day7.15,X_train$day2.15,X_train$day1.15)
data16 = data.frame(Y_train$day0.16,X_train$day7.16,X_train$day2.16,X_train$day1.16)
data17 = data.frame(Y_train$day0.17,X_train$day7.17,X_train$day2.17,X_train$day1.17)
data18 = data.frame(Y_train$day0.18,X_train$day7.18,X_train$day2.18,X_train$day1.18)
data19 = data.frame(Y_train$day0.19,X_train$day7.19,X_train$day2.19,X_train$day1.19)
data20 = data.frame(Y_train$day0.20,X_train$day7.20,X_train$day2.20,X_train$day1.20)
data21 = data.frame(Y_train$day0.21,X_train$day7.21,X_train$day2.21,X_train$day1.21)
data22 = data.frame(Y_train$day0.22,X_train$day7.22,X_train$day2.22,X_train$day1.22)
data23 = data.frame(Y_train$day0.23,X_train$day7.23,X_train$day2.23,X_train$day1.23)


#name our variables
colnames(data00) <- c('day0','pday7','pday2','pday1')
colnames(data01) <- c('day0','pday7','pday2','pday1')
colnames(data02) <- c('day0','pday7','pday2','pday1')
colnames(data03) <- c('day0','pday7','pday2','pday1')
colnames(data04) <- c('day0','pday7','pday2','pday1')
colnames(data05) <- c('day0','pday7','pday2','pday1')
colnames(data06) <- c('day0','pday7','pday2','pday1')
colnames(data07) <- c('day0','pday7','pday2','pday1')
colnames(data08) <- c('day0','pday7','pday2','pday1')
colnames(data09) <- c('day0','pday7','pday2','pday1')
colnames(data10) <- c('day0','pday7','pday2','pday1')
colnames(data11) <- c('day0','pday7','pday2','pday1')
colnames(data12) <- c('day0','pday7','pday2','pday1')
colnames(data13) <- c('day0','pday7','pday2','pday1')
colnames(data14) <- c('day0','pday7','pday2','pday1')
colnames(data15) <- c('day0','pday7','pday2','pday1')
colnames(data16) <- c('day0','pday7','pday2','pday1')
colnames(data17) <- c('day0','pday7','pday2','pday1')
colnames(data18) <- c('day0','pday7','pday2','pday1')
colnames(data19) <- c('day0','pday7','pday2','pday1')
colnames(data20) <- c('day0','pday7','pday2','pday1')
colnames(data21) <- c('day0','pday7','pday2','pday1')
colnames(data22) <- c('day0','pday7','pday2','pday1')
colnames(data23) <- c('day0','pday7','pday2','pday1')

#linear model to predict load at hour i
mod0 = lm(day0 ~., data = data00)
mod1 = lm(day0 ~., data = data01)
mod2 = lm(day0 ~., data = data02)
mod3 = lm(day0 ~., data = data03)
mod4 = lm(day0 ~., data = data04)
mod5 = lm(day0 ~., data = data05)
mod6 = lm(day0 ~., data = data06)
mod7 = lm(day0 ~., data = data07)
mod8 = lm(day0 ~., data = data08)
mod9 = lm(day0 ~., data = data09)
mod10 = lm(day0 ~., data = data10)
mod11 = lm(day0 ~., data = data11)
mod12 = lm(day0 ~., data = data12)
mod13 = lm(day0 ~., data = data13)
mod14 = lm(day0 ~., data = data14)
mod15 = lm(day0 ~., data = data15)
mod16 = lm(day0 ~., data = data16)
mod17 = lm(day0 ~., data = data17)
mod18 = lm(day0 ~., data = data18)
mod19 = lm(day0 ~., data = data19)
mod20 = lm(day0 ~., data = data20)
mod21 = lm(day0 ~., data = data21)
mod22 = lm(day0 ~., data = data22)
mod23 = lm(day0 ~., data = data23)

#retrieve testing data at hour i
newdat0 = data.frame(pday7 = X_test$day7.00, pday2=X_test$day2.00, pday1=X_test$day1.00)
newdat1 = data.frame(pday7 = X_test$day7.01, pday2=X_test$day2.01, pday1=X_test$day1.01)
newdat2 = data.frame(pday7 = X_test$day7.02, pday2=X_test$day2.02, pday1=X_test$day1.02)
newdat3 = data.frame(pday7 = X_test$day7.03, pday2=X_test$day2.03, pday1=X_test$day1.03)
newdat4 = data.frame(pday7 = X_test$day7.04, pday2=X_test$day2.04, pday1=X_test$day1.04)
newdat5 = data.frame(pday7 = X_test$day7.05, pday2=X_test$day2.05, pday1=X_test$day1.05)
newdat6 = data.frame(pday7 = X_test$day7.06, pday2=X_test$day2.06, pday1=X_test$day1.06)
newdat7 = data.frame(pday7 = X_test$day7.07, pday2=X_test$day2.07, pday1=X_test$day1.07)
newdat8 = data.frame(pday7 = X_test$day7.08, pday2=X_test$day2.08, pday1=X_test$day1.08)
newdat9 = data.frame(pday7 = X_test$day7.09, pday2=X_test$day2.09, pday1=X_test$day1.09)
newdat10 = data.frame(pday7 = X_test$day7.10, pday2=X_test$day2.10, pday1=X_test$day1.10)
newdat11 = data.frame(pday7 = X_test$day7.11, pday2=X_test$day2.11, pday1=X_test$day1.11)
newdat12 = data.frame(pday7 = X_test$day7.12, pday2=X_test$day2.12, pday1=X_test$day1.12)
newdat13 = data.frame(pday7 = X_test$day7.13, pday2=X_test$day2.13, pday1=X_test$day1.13)
newdat14 = data.frame(pday7 = X_test$day7.14, pday2=X_test$day2.14, pday1=X_test$day1.14)
newdat15 = data.frame(pday7 = X_test$day7.15, pday2=X_test$day2.15, pday1=X_test$day1.15)
newdat16 = data.frame(pday7 = X_test$day7.16, pday2=X_test$day2.16, pday1=X_test$day1.16)
newdat17 = data.frame(pday7 = X_test$day7.17, pday2=X_test$day2.17, pday1=X_test$day1.17)
newdat18 = data.frame(pday7 = X_test$day7.18, pday2=X_test$day2.18, pday1=X_test$day1.18)
newdat19 = data.frame(pday7 = X_test$day7.19, pday2=X_test$day2.19, pday1=X_test$day1.19)
newdat20 = data.frame(pday7 = X_test$day7.20, pday2=X_test$day2.20, pday1=X_test$day1.20)
newdat21 = data.frame(pday7 = X_test$day7.21, pday2=X_test$day2.21, pday1=X_test$day1.21)
newdat22 = data.frame(pday7 = X_test$day7.22, pday2=X_test$day2.22, pday1=X_test$day1.22)
newdat23 = data.frame(pday7 = X_test$day7.23, pday2=X_test$day2.23, pday1=X_test$day1.23)


#use our model to predict the expect load at hour 0
predict(mod0, newdata = newdat0)[1]
##        1 
## 35404.69

# 72 numbers should be returned; 1 for each day
```

``` r
temp = NULL
for (i in 1:24) {
  temp = rbind(temp, c(i-1, predict(eval(as.name(paste0('mod',i-1))), newdata = eval(as.name(paste0('newdat',i-1))))[1]))
}
temp
##                 1
##  [1,]  0 35404.69
##  [2,]  1 33788.02
##  [3,]  2 33020.14
##  [4,]  3 32755.51
##  [5,]  4 33146.93
##  [6,]  5 34779.76
##  [7,]  6 38964.72
##  [8,]  7 44029.63
##  [9,]  8 45744.20
## [10,]  9 43869.92
## [11,] 10 42693.49
## [12,] 11 42033.56
## [13,] 12 41126.56
## [14,] 13 40512.10
## [15,] 14 39980.77
## [16,] 15 39631.62
## [17,] 16 39307.17
## [18,] 17 39637.21
## [19,] 18 41997.60
## [20,] 19 43511.69
## [21,] 20 44395.65
## [22,] 21 44864.98
## [23,] 22 43679.22
## [24,] 23 41841.00
```

\`
