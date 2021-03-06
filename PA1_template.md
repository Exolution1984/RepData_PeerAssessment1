# Reproducible Research: Peer Assessment 1
========================================================
## Loading and preprocessing the data

```r
data <- read.csv("activity.csv")
filter <- !is.na(data$steps)
totalSteps <- tapply(data$steps[filter], data$date[filter], sum)
```

## What is mean total number of steps taken per day?
Make a histogram of the total number of steps taken each day

```r
hist(totalSteps)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

Calculate and report the mean and median total number of steps taken per day

```r
mean <- mean(totalSteps, na.rm = TRUE)
median <- median(totalSteps, na.rm = TRUE)
mean
```

```
## [1] 10766
```

```r
median
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
meanSteps <- tapply(data$steps, data$interval, mean, na.rm = TRUE)
plot(unique(data$interval), meanSteps, type = "l")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
unique(data$interval)[which.max(meanSteps)]
```

```
## [1] 835
```

## Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(!complete.cases(data))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
filledData <- data
filter <- is.na(data$steps)
naData <- data[filter, ]
for (i in 1:dim(naData)[1]) {
    rownum <- as.numeric(rownames(naData[i, ]))
    interval <- as.character(naData[i, 3])
    filledData[rownum, 1] <- meanSteps[interval]
}
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r


totalSteps <- tapply(filledData$steps, filledData$date, sum)
hist(totalSteps)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

```r
mean <- mean(totalSteps, na.rm = TRUE)
median <- median(totalSteps, na.rm = TRUE)
mean
```

```
## [1] 10766
```

```r
median
```

```
## [1] 10766
```

Missing don't have much effect on median and mean value
## Are there differences in activity patterns between weekdays and weekends?


```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
weekdays <- weekdays(as.Date(filledData$date))
filter <- weekdays %in% c("Saturday", "Sunday")
dayType <- rep("", length(filledData$date))
dayType[filter] <- "weekend"
dayType[!filter] <- "weekday"
meanSteps <- tapply(filledData$steps, list(dayType, filledData$interval), mean)

condition <- expand.grid(rownames(meanSteps), colnames(meanSteps))
dim(meanSteps) <- c(dim(meanSteps)[2] * dim(meanSteps)[1], 1)
meanSteps <- data.frame(meanSteps, condition)
names(meanSteps) <- c("stepMean", "dayType", "interval")
library("lattice")
```

```
## Warning: package 'lattice' was built under R version 3.0.3
```

```r
xyplot(stepMean ~ as.numeric(interval) | dayType, data = meanSteps, layout = c(1, 
    2), type = "l", ylab = "Number of steps", xlab = "Interval")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 


