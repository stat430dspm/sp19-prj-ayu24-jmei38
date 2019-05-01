
<!-- README.md is generated from README.Rmd. Please edit that file -->
Roadmap:

report results/conclusions and descriptive text for visuals

Electricity Load Prediction in Texas
====================================

Introduction
------------

How do electric companies know how much power they have to generate?

But why is it important to predict hourly demand for electricity at least a day in advance? You need to know much generators needs to be on to meet the expected demand and turning on a generator requires time.

Libraries/packages we will be using
-----------------------------------

``` r
new_cran_packages <- c("ggplot2", "caret","stringr", "cowplot", "grid", "gridExtra")
existing_packages <- installed.packages()[,"Package"]
missing_packages <- new_cran_packages[!(new_cran_packages %in% existing_packages)]
if(length(missing_packages)){
    install.packages(missing_packages)
}

library(ggplot2)
library(stringr)
library(caret)
library(cowplot)
library(grid)
library(gridExtra)
```

Load the ERCOT 2018 data
------------------------

Let's see how does load vary over the year in Texas. <img src="README_figs/README-electricity graph-1.png" width="672" />

For fun, let's look at the production of wind energy of the year.

<img src="README_figs/README-wind output graph-1.png" width="672" />

Wind Power looks very sporadic while electricity demands seems to have a trend.

<img src="README_figs/README-March HeatMap-1.png" width="672" />

<img src="README_figs/README-July HeatMap-1.png" width="672" />

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

Check dimensions of our X and Y sets to see if they are consistent.

``` r
dim(X_train)
## [1] 286  72
dim(X_test)
## [1] 72 72
dim(Y_train)
## [1] 286  24
dim(Y_test)
## [1] 72 24
```

Prediction Using Multiple Linear Regression
-------------------------------------------

Predict Load of tommorrow at hour i based on last 7 days at time i, last 2 days at time i and yesterday at time i where *i* ∈ 1, 2, ...23

In total there will be 24 linear models for each hour of the day.

model and test data setup
-------------------------

``` r
data <- list(dat0 = list(model = mod0, test = newdat0),
             dat1 = list(model = mod1, test = newdat1),
             dat2 = list(model = mod2, test = newdat2),
             dat3 = list(model = mod3, test = newdat3),
             dat4 = list(model = mod4, test = newdat4),
             dat5 = list(model = mod5, test = newdat5),
             dat6 = list(model = mod6, test = newdat6),
             dat7 = list(model = mod7, test = newdat7),
             dat8 = list(model = mod8, test = newdat8),
             dat9 = list(model = mod9, test = newdat9),
             dat10 = list(model = mod10, test = newdat10),
             dat11 = list(model = mod11, test = newdat11),
             dat12 = list(model = mod12, test = newdat12),
             dat13 = list(model = mod13, test = newdat13),
             dat14 = list(model = mod14, test = newdat14),
             dat15 = list(model = mod15, test = newdat15),
             dat16 = list(model = mod16, test = newdat16),
             dat17 = list(model = mod17, test = newdat17),
             dat18 = list(model = mod18, test = newdat18),
             dat19 = list(model = mod19, test = newdat19),
             dat20 = list(model = mod20, test = newdat20),
             dat21 = list(model = mod21, test = newdat21),
             dat22 = list(model = mod22, test = newdat22),
             dat23 = list(model = mod23, test = newdat23))
```

Results
-------

Here we choose 16 days to see how well our predictions were. <img src="README_figs/README-unnamed-chunk-9-1.png" width="672" />

Check if the number of predictions we have matches the number of measured values in the test set.

``` r
predictions = sapply(data, function(dat) predict(dat$model, newdata = dat$test))

dim(predictions)
## [1] 72 24
dim(Y_test)
## [1] 72 24
```

### Testing Accuracy using Min-Max (closer to 1, the better)

``` r
min_max <- mean(apply(act_pred, 1, min) / apply(act_pred, 1, max))
print(min_max)
## [1] 0.8117769
```

### Other Assessment Metrics

``` r
source('Functions.R')
error = act_pred$actuals - act_pred$predicteds
mae(error) # Mean Absolute Error
## [1] 9634.81
rmse(error) # Root Mean Squared Error
## [1] 12225.49
```

Conclusion/Future Work
----------------------

\`
