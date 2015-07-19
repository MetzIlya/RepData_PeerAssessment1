---
title: "Activity data report"
author: "Ilya Metz"
date: "Sunday, July 19, 2015"
output: html_document
---

Loading and preprocessing the data
==================================

```r
library(dplyr)

em_url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
em_zip <- "./ActivityData.zip"
em_unzip <- "./data/activity.csv"
if(!file.exists(em_zip)){
  print(paste("Downloading Activity Data zip file into ",em_zip ,  " ..."))
  download.file(em_url,destfile=em_zip, mode="wb")
}else{
  print(paste(em_zip, " file already downloaded"))
}
```

```
## [1] "./ActivityData.zip  file already downloaded"
```

```r
if(!file.exists(em_unzip)){
  print(paste("Unzip file ",  em_unzip))
  unzip(em_zip, exdir="./data")   
}else{
  print(paste(em_unzip, " file already unziped"))
}
```

```
## [1] "./data/activity.csv  file already unziped"
```

```r
act_data <- read.csv("./data/activity.csv");
act_tbl <- tbl_df(act_data)
```

Total number of steps taken per day 
===================================

Historgram for total number of steps per day: 

```r
act_hist <- act_tbl %>% group_by(date) %>% summarise(steps=sum(steps, na.rm = TRUE))
                                                   
hist(act_hist$steps, main="Histogram of steps per day", xlab = "Steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

Mean and median are quite close to each other: 

```r
mean(act_hist$steps)
```

```
## [1] 9354.23
```

```r
median(act_hist$steps)
```

```
## [1] 10395
```

What is the average daily activity pattern
==========================================
Graph for day with 5 minutes invterval steps:

```r
plot(act_tbl %>% na.omit() %>% group_by(interval) %>% 
       summarise(steps=mean(steps)), type="l")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

We can better see peak on the following graphs:

```r
plot(act_tbl %>% na.omit() %>% group_by(interval) %>% filter(interval>700, interval<1000) %>% summarise(steps=mean(steps)), type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

```r
plot(act_tbl %>% na.omit() %>% group_by(interval) %>% filter(interval>840, interval<860) %>% summarise(steps=mean(steps)), type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-2.png) 
We can see maximum at 850 5-minutes interval of day activity.

Replacing missing values with mean value
========================================
Count na and interval with values

```r
# How many missing values do we have?  
act_tbl %>% filter(is.na(steps)) %>% count()
```

```
## Source: local data frame [1 x 1]
## 
##      n
## 1 2304
```

```r
# How many intervals wtih values do we have? 
act_tbl %>% filter(!is.na(steps)) %>% count()
```

```
## Source: local data frame [1 x 1]
## 
##       n
## 1 15264
```

Replacing NAs with mean, build histogram

```r
# Creating new dataset
act_tbl_not_null <- act_tbl

# Replacing missed value with mean value
act_tbl_not_null[is.na(act_tbl$steps),]$steps <- mean((act_tbl %>% filter(!is.na(steps)))$steps)

# Create histogram for this dataset
act_hist_not_null <- act_tbl_not_null %>% group_by(date) %>% summarise(steps=sum(steps, na.rm = TRUE))
hist(act_hist_not_null$steps, main="Histogram of steps per day", xlab = "Steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png) 

Differences in activity patterns between weekdays and weekends
==============================================================

```r
act_tbl_not_null <- mutate(act_tbl_not_null, day = 'weekend') 
act_tbl_not_null$day <- ifelse(as.POSIXlt(act_tbl_not_null$date)$wday %in% c(0,6), 'weekend', 'weekday')
act_tbl_not_null$day <- as.factor(act_tbl_not_null$day)

plot(act_tbl_not_null %>% group_by(interval) %>% filter(day=='weekday') %>% summarise(steps=mean(steps, na.rm = TRUE)), type="l", main = "Weekday steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

```r
plot(act_tbl_not_null %>% group_by(interval) %>% filter(day=='weekend') %>% summarise(steps=mean(steps, na.rm = TRUE)), type="l", main = "Weekend steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-2.png) 
