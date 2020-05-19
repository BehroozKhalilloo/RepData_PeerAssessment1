---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
    fig_caption: yes
---


## 1. Loading and preprocessing the data


```r
library(readr)
myData <- read_csv("activity.zip", na = "NA")
```

```
## Parsed with column specification:
## cols(
##   steps = col_double(),
##   date = col_date(format = ""),
##   interval = col_double()
## )
```

create a separate dataset with 0's instead of NA's for easier computation:

```r
myData0 <- myData
myData0[is.na(myData0)] <- 0
```

## 2. Histogram of the total number of steps taken each day

Calculate number of steps for each day:

```r
library(dplyr)
totalSteps <- myData0 %>% group_by(date) %>% summarise(total = sum(steps))
```

Create histogram:

```r
library(ggplot2)
ggplot(totalSteps, aes(x=total)) + geom_histogram(fill="#69b3a2", color="#e9ecef", alpha=0.9) + xlab("Total Number of Steps") + ylab("")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

## Mean and median number of steps taken each day

Mean number of steps:

```r
mean(totalSteps$total)
```

```
## [1] 9354.23
```
Median number of steps:

```r
median(totalSteps$total)
```

```
## [1] 10395
```

## Time series plot of the average number of steps taken

Sort the data by 5 min interval:

```r
int <- myData0 %>% group_by(interval) %>% summarise(total = sum(steps)) %>% as.data.frame(int)
```

Create time series plot:

```r
ggplot(int, aes(x=interval, y=total)) + geom_line( color="#69b3a2") + xlab("5 min Interval")+ylab("Total Number of Steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)

## The 5-minute interval that, on average, contains the maximum number of steps


```r
int[int$total==max(int$total),]
```

```
##     interval total
## 104      835 10927
```

## Code to describe and show a strategy for imputing missing data

Calculate the number of missing values:

```r
sum(is.na(myData$steps))
```

```
## [1] 2304
```

Strategy for filling in missing values - use mean for same time interval as the missing value.

Calculate means for each time interval:

```r
intMean <- myData0 %>% group_by(interval) %>% summarise(Mean = mean(steps))
```

Add new column with means to the original data set(repeated 61 times for 61 days):

```r
myData <- mutate(myData, intMean = rep(intMean$Mean, 61)) %>% as.data.frame(myData)
```

Fill in the NA's in steps column with means:

```r
myData$steps <- coalesce(myData$steps, myData$intMean)
```

## Histogram of the total number of steps taken each day after missing values are imputed

Calculate number of steps for each day:

```r
fillSteps <- myData %>% group_by(date) %>% summarise(total = sum(steps))
```

Create histogram:

```r
ggplot(fillSteps, aes(x=total)) + geom_histogram(fill="#69b3a2", color="#e9ecef", alpha=0.9) + xlab("Total Number of Steps") + ylab("")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png)

Mean number of steps:

```r
mean(fillSteps$total)
```

```
## [1] 10581.01
```
Median number of steps:

```r
median(fillSteps$total)
```

```
## [1] 10395
```

Compared to the data before I filled in missing values, the mean increased around 13% and the median stayed the same.

## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

Create a new column with factor for weekend/weekday:

```r
weekend <- c("Saturday", "Sunday")
myData$day <- factor((weekdays(myData$date) %in% weekend), levels=c(TRUE, FALSE), labels=c("Weekend", "Weekday"))
```

Create the panel plot:

```r
g1 <- ggplot(subset(myData, day == "Weekend"), aes(x=interval, y=steps)) + geom_line( color="#69b3a2") + xlab("Weekend")+ylab("Total Number of Steps")

g2 <- ggplot(subset(myData, day == "Weekday"), aes(x=interval, y=steps)) + geom_line( color="#69b3a2") + xlab("Weekdays")+ylab("Total Number of Steps")

library(gridExtra)
grid.arrange(g1, g2, nrow=2)
```

![plot of chunk unnamed-chunk-19](figure/unnamed-chunk-19-1.png)
