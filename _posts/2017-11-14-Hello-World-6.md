---
title: "Trying to develop a template for future posts."
author: "Christopher Eshleman"
date: "11/12/2017"
output: html_document
---

So this is just a training wheels post.

First, I sync R Studio to my GitHub Pages website. 

Second, I open an R Markdown (Rmd) file and script something. (That 'something' is what you're reading now.)

Third, I save the Rmd file locally. Notice I don't say I knit it to HTML — there's a problem with that step that I haven't totally figured out. 

Fourth, I use a separate R script that Nicole White developed (much appreciated) to use the terminal to convert the Rmd file to a plain markdown (md) file. Here's her fantastic tutorial on the topic: https://nicolewhite.github.io/2015/02/07/r-blogging-with-rmarkdown-knitr-jekyll.html. 

There are really two smaller steps within step number four — first I use 'cd' to navigate the terminal to the subfolder I want, then I use the command './r2jekyll.R {file name}.Rmd' and it makes the conversion. (In this case the file name is 'Hello-World-6'.)

Fifth and finally, I manually upload both the markdown file and any images to my GitHub page. The markdown file goes in my 'posts' folder and the image goes, creatively, into the 'image' folder. 



Since it's just practice, I'm monkeying around with R's built-in 'mtcars' data set. 


```r
head(mtcars,4)
```

```
##                 mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4      21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag  21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710     22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive 21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
```

Here's a scatter plot of the relationship between vehicle weights and miles per gallon. A negative relationship, as would've been expected. 


```r
library(ggplot2) 
ggplot(mtcars, aes(x=wt, y=mpg)) +
    geom_point(shape=1) +   
    geom_smooth(method=lm)  
```

![plot of chunk plot-this-post-6](/Users/chriseshleman/Dropbox/pages/chriseshleman.github.io/images/post-6-says-plot-this-post-6-1.png)


There are plenty of problems with this posting process. For example, the last step (embedding script to host the image) means rendering the original R Markdown file useless in R Studio itself — it trips up the knitting process. But leaving it out means going into the post once its in GitHub and manually inserting the line.  

This'll do for now, though. 
