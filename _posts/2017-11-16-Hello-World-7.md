---
title: "Green Jobs"
author: "Christopher Eshleman"
date: "11/16/2017"
output: html_document
---



## Alaska & Green Jobs 


```r
a05 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2005.annual.singlefile.csv")
a06 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2006.annual.singlefile.csv")
a07 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2007.annual.singlefile.csv")
a08 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2008.annual.singlefile.csv")
a09 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2009.annual.singlefile.csv")
a10 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2010.annual.singlefile.csv")
a11 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2011.annual.singlefile.csv")
a12 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2012.annual.singlefile.csv")
a13 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2013.annual.singlefile.csv")
a14 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2014.annual.singlefile.csv")
a15 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2015.annual.singlefile.csv")
a16 = read.csv("/Users/chriseshleman/Dropbox/Research_and_training_2/QCEW and green jobs/2016.annual.singlefile.csv")

a = rbind(a05,a06,a07,a08,a09,a10,a11,a12,a13,a14,a15,a16) 
head(a) 
```

```
##   area_fips own_code industry_code agglvl_code size_code year qtr
## 1     01000        0            10          50         0 2005   A
## 2     01000        1            10          51         0 2005   A
## 3     01000        1           102          52         0 2005   A
## 4     01000        1          1021          53         0 2005   A
## 5     01000        1          1022          53         0 2005   A
## 6     01000        1          1023          53         0 2005   A
##   disclosure_code annual_avg_estabs annual_avg_emplvl total_annual_wages
## 1                            117064           1894616        65549226595
## 2                              1193             51451         3114231240
## 3                              1193             51451         3114231240
## 4                               575             12982          616225136
## 5                                 3                 8             168445
## 6                                25               155            9886704
##   taxable_annual_wages annual_contributions annual_avg_wkly_wage
## 1          14067542860            299023667                  665
## 2                    0                    0                 1164
## 3                    0                    0                 1164
## 4                    0                    0                  913
## 5                    0                    0                  418
## 6                    0                    0                 1230
##   avg_annual_pay lq_disclosure_code lq_annual_avg_estabs
## 1          34598                                    1.00
## 2          60528                                    1.65
## 3          60528                                    1.66
## 4          47468                                    2.02
## 5          21735                                    3.54
## 6          63957                                    1.66
##   lq_annual_avg_emplvl lq_total_annual_wages lq_taxable_annual_wages
## 1                 1.00                  1.00                       1
## 2                 1.31                  1.55                       0
## 3                 1.33                  1.58                       0
## 4                 1.04                  1.18                       0
## 5                 0.09                  0.03                       0
## 6                 0.77                  0.76                       0
##   lq_annual_contributions lq_annual_avg_wkly_wage lq_avg_annual_pay
## 1                       1                    1.00              1.00
## 2                       0                    1.19              1.19
## 3                       0                    1.19              1.19
## 4                       0                    1.13              1.13
## 5                       0                    0.38              0.38
## 6                       0                    0.98              0.98
##   oty_disclosure_code oty_annual_avg_estabs_chg
## 1                                          2628
## 2                                            70
## 3                                            70
## 4                                            51
## 5                                             0
## 6                                             0
##   oty_annual_avg_estabs_pct_chg oty_annual_avg_emplvl_chg
## 1                           2.3                     42847
## 2                           6.2                       498
## 3                           6.2                       498
## 4                           9.7                      -120
## 5                           0.0                        -1
## 6                           0.0                        -9
##   oty_annual_avg_emplvl_pct_chg oty_total_annual_wages_chg
## 1                           2.3                 3673445874
## 2                           1.0                  183583961
## 3                           1.0                  183583961
## 4                          -0.9                  -11790205
## 5                         -11.1                     -37215
## 6                          -5.5                     -78903
##   oty_total_annual_wages_pct_chg oty_taxable_annual_wages_chg
## 1                            5.9                    607344491
## 2                            6.3                            0
## 3                            6.3                            0
## 4                           -1.9                            0
## 5                          -18.1                            0
## 6                           -0.8                            0
##   oty_taxable_annual_wages_pct_chg oty_annual_contributions_chg
## 1                              4.5                     50416342
## 2                              0.0                            0
## 3                              0.0                            0
## 4                              0.0                            0
## 5                              0.0                            0
## 6                              0.0                            0
##   oty_annual_contributions_pct_chg oty_annual_avg_wkly_wage_chg
## 1                             20.3                           22
## 2                              0.0                           58
## 3                              0.0                           58
## 4                              0.0                           -9
## 5                              0.0                          -34
## 6                              0.0                           64
##   oty_annual_avg_wkly_wage_pct_chg oty_avg_annual_pay_chg
## 1                              3.4                   1184
## 2                              5.2                   3011
## 3                              5.2                   3011
## 4                             -1.0                   -465
## 5                             -7.5                  -1769
## 6                              5.5                   3314
##   oty_avg_annual_pay_pct_chg
## 1                        3.5
## 2                        5.2
## 3                        5.2
## 4                       -1.0
## 5                       -7.5
## 6                        5.5
```

```r
# 
b = subset(a,a$area_fips=="02000" & a$industry_code=="22111")
bb = subset(b, b$own_code==1)
```

## Green Jobs. 

Well lookie what we have here. 


```r
head(bb,8)
```

```
##  [1] area_fips                        own_code                        
##  [3] industry_code                    agglvl_code                     
##  [5] size_code                        year                            
##  [7] qtr                              disclosure_code                 
##  [9] annual_avg_estabs                annual_avg_emplvl               
## [11] total_annual_wages               taxable_annual_wages            
## [13] annual_contributions             annual_avg_wkly_wage            
## [15] avg_annual_pay                   lq_disclosure_code              
## [17] lq_annual_avg_estabs             lq_annual_avg_emplvl            
## [19] lq_total_annual_wages            lq_taxable_annual_wages         
## [21] lq_annual_contributions          lq_annual_avg_wkly_wage         
## [23] lq_avg_annual_pay                oty_disclosure_code             
## [25] oty_annual_avg_estabs_chg        oty_annual_avg_estabs_pct_chg   
## [27] oty_annual_avg_emplvl_chg        oty_annual_avg_emplvl_pct_chg   
## [29] oty_total_annual_wages_chg       oty_total_annual_wages_pct_chg  
## [31] oty_taxable_annual_wages_chg     oty_taxable_annual_wages_pct_chg
## [33] oty_annual_contributions_chg     oty_annual_contributions_pct_chg
## [35] oty_annual_avg_wkly_wage_chg     oty_annual_avg_wkly_wage_pct_chg
## [37] oty_avg_annual_pay_chg           oty_avg_annual_pay_pct_chg      
## <0 rows> (or 0-length row.names)
```

## Ha bou dat? 


```
## Don't know how to automatically pick scale for object of type function. Defaulting to continuous.
```

```
## Error in FUN(X[[i]], ...): object 'speed' not found
```

```
## geom_point: na.rm = FALSE
## stat_identity: na.rm = FALSE
## position_identity
```

![Things.]({{site.baseurl}}/images/pic.png)

```
## quartz_off_screen 
##                 2
```
