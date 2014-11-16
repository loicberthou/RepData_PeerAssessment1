---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
unzip("activity.zip", overwrite = TRUE, exdir = "./data")
activityData <- read.csv("./data/activity.csv")
```

## What is mean total number of steps taken per day?

```r
activityDataClean <- activityData[!is.na(activityData$steps), ]
totalDataClean <- aggregate(activityDataClean$steps, by=list(date=activityDataClean$date), FUN = sum)
names(totalDataClean) <- c("date", "total.steps")
hist(totalDataClean$total.steps)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r
meanSteps <- mean(totalDataClean$total.steps)
medianSteps <- median(totalDataClean$total.steps)
```
Mean total number of steps taken per day = 10766  
Median total number of steps taken per day = 10765  

## What is the average daily activity pattern?

```r
averageDataClean <- aggregate(activityDataClean$steps, by=list(interval=activityDataClean$interval), FUN = mean)
names(averageDataClean) <- c("interval", "average.steps")
plot(averageDataClean$interval, averageDataClean$average.steps, type="l", xlab = "Interval", ylab="Average number of steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
intervalMaxMean <- averageDataClean[averageDataClean$average.steps==max(averageDataClean$average.steps), "interval"]
```
On average across all the days in the dataset, the 5-minute interval that contains the maximum number of steps is 835.

## Imputing missing values

```r
rowsNotCompleteSum <- sum(!complete.cases(activityData))
rowsStepsIsNa <- sum(is.na(activityData$steps))
```
The total number of missing complete values (rows) in the dataset is 2304.  
The total number of missing values in the steps column is 2304.  
I will replace the missing value (NA) in the steps column with the mean for that 5-minute interval.


```r
mergedData <- merge(activityData, averageDataClean)
mergedDataClean <- mergedData
mergedDataClean$steps[is.na(mergedDataClean$steps)] <- mergedDataClean$average.steps[is.na(mergedDataClean$steps)]
mergedDataClean$average.steps <- NULL
mergedDataCleanTotal <- aggregate(mergedDataClean$steps, by=list(date=mergedDataClean$date), FUN = sum)
names(mergedDataCleanTotal) <- c("date", "total.steps")
hist(mergedDataCleanTotal$total.steps)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

```r
meanSteps <- mean(mergedDataCleanTotal$total.steps)
medianSteps <- median(mergedDataCleanTotal$total.steps)
```
Mean total number of steps taken per day (After inputing missing values) = 10766  
Median total number of steps taken per day (After inputing missing values) = 10766  

## Are there differences in activity patterns between weekdays and weekends?

```r
mergedDataClean$day <- ifelse(is.element(weekdays(as.Date(mergedDataClean$date, format = "%Y-%m-%d")), c("Saturday", "Sunday")), "weekend", "weekday")
mergedAverageDataClean <- aggregate(mergedDataClean$steps,
                                    by=list(interval=mergedDataClean$interval, day=mergedDataClean$day),
                                    FUN = mean)
names(mergedAverageDataClean) <- c("interval", "day", "average.steps")

plot(mergedAverageDataClean$interval[mergedAverageDataClean$day=="weekday"],
     mergedAverageDataClean$average.steps[mergedAverageDataClean$day=="weekday"],
     type="l",
     xlab = "Interval",
     ylab="Average number of steps",
     xlim=range(mergedAverageDataClean$interval),
     ylim=range(mergedAverageDataClean$average.steps))
lines(mergedAverageDataClean$interval[mergedAverageDataClean$day=="weekend"],
      mergedAverageDataClean$average.steps[mergedAverageDataClean$day=="weekend"],
      type="l",
      col=2)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 
