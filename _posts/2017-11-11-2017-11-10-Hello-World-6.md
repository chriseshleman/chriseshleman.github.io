---
title: "No. Nothing to see here."
author: "Christopher Eshleman"
date: "11/10/2017"
output: html_document
---



In lieu of a substantive post this is just a placeholder as I figure out how to post straight from R Studio (using R Markdown) to the site. 

```
qcewGetAreaData = function(year, qtr, area) {
  url <- "http://data.bls.gov/cew/data/api/YEAR/QTR/area/AREA.csv"
  url <- sub("YEAR", year, url, ignore.case=FALSE)
  url <- sub("QTR", tolower(qtr), url, ignore.case=FALSE)
  url <- sub("AREA", toupper(area), url, ignore.case=FALSE)
  read.csv(url, header = TRUE, sep = ",", quote="\"", dec=".", na.strings=" ", skip=0)
}

us_1601 <- qcewGetAreaData("2016", "4", "US000") 
us_1601 = subset(us_1601, us_1601$industry_code==331 & us_1601$own_code==5) 

subb = subset(us_1601,us_1601$est>1) 
boxplot(subb$est) 
```

I'm not embedding any plots right now -- skipping it for the moment. 
