---
title: "Woof" 
author: "Christopher Eshleman" 
date: "Dec. 27, 2017" 
output: 
  pdf_document: default 
  word_document: default 
---

I was asked a couple of weeks to do a little multivariate causal inference using time series. And I had a few days to do it. Woof. So I gave it a crack. Here's an extremely simplified toy version of what I did, just to illustrate the process. 

```r
library(forecast)
library(tseries)
a=c(1,1,1,1,1,1,1,1,1,1,1.25,1.25,1.25,1.25,1.25,1.25,1.25,1.25,1.25,1.25,
    1.50,1.50,1.50,1.50,1.50,1.50,1.50,1.50,1.50,1.50,2,2,2,2,2,2,2,2,2,2,
    2.50,2.50,2.50,2.50,2.50,2.50,2.50,2.50,2.50,2.50) 
b=sample(500:1000, 50) 
c=sample(125000:225000,50) 
d=sample(350:500,50) 
e = seq(as.Date("2000/1/1"), by = "month", length.out = 50)
```

So I'll use Rob Hyndman's packages and his algorithm for finding a good ARIMA fit. Packaged the covariates into a box and left the actual month vector out of the regression. 

```r
stuff=as.matrix(data.frame(a,b,d)) 
```

And of course I held 'c' out of there. (That's my simulated ridership.) X variable of interest here is price (the 'a' variable).

I'd basically be looking to the literature to help me determine the best controls. Confounding variables are sticky when it comes to causal inference, and I haven't gotten comfortable enough with machine learning yet to trust it with causal inference questions; I know multiple econometricians and statisticians that will tell you it's not the right bag of tools at all. For now, I stuck with the literature to inform me regarding what covarites might help, then worked my way through them at bivariate levels with some stratification. Basically, I read and then explored the data, then used Professor Hyndman's auto.arima function to turn my brain off when it came to time series components. 

So I'd be looking here for a reasonably unbiased 'a' coefficient. If this wasn't completely made up data. Which it is. 

```r
fit_me = auto.arima(ts(log(a)),xreg=stuff) 
summary(fit_me) 
```

```
## Series: ts(log(a)) 
## Regression with ARIMA(0,1,0) errors 
## 
## Coefficients:
```

```
## Warning in sqrt(diag(x$var.coef)): NaNs produced
```

```
##            a  b      d
##       0.5702  0  0e+00
## s.e.     NaN  0  2e-04
## 
## sigma^2 estimated as 0.0002544:  log likelihood=134.8
## AIC=-261.59   AICc=-260.68   BIC=-254.03
## 
## Training set error measures:
##                       ME       RMSE        MAE MPE MAPE      MASE
## Training set 0.001153381 0.01529893 0.00514414 NaN  Inf 0.2750905
##                    ACF1
## Training set 0.01665829
```
