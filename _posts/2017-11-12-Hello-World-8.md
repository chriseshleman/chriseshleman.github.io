---
title: "More monkeying around"
author: "Christopher Eshleman"
date: "11/12/2017"
output: html_document
---

Continuing the same thing ...


```r
knitr::opts_chunk$set(fig.path='{{ site.url }}/images/exploring-the-cars-dataset-')
```

This is exhilarating. 


```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```

Maybe I can get a plot this time? 


```r
plot(cars)
library(ggplot2) 
```

![plot of chunk my-first-plot]({{ site.url }}/images/exploring-the-cars-dataset-my-first-plot-1.png)

```r
png("/Users/chriseshleman/Dropbox/pages/chriseshleman.github.io/images/samplepic.png", width = 500, height = 300)
ggplot(data=cars, aes(x=dist, y=speed, group=1)) + 
  geom_line(color="red") 
  geom_point() 
```

```
## geom_point: na.rm = FALSE
## stat_identity: na.rm = FALSE
## position_identity
```

```r
dev.off()   
```

```
## quartz_off_screen 
##                 2
```
