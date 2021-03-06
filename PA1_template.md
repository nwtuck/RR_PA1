---
title: "RR_PA1"
author: "Wei Tuck"
date: "Sunday, December 14, 2014"
output: html_document
---

#Loading and preprocessing the data


```r
unzip("repdata-data-activity.zip")
data <- read.csv("activity.csv")

data$date <- as.Date(data$date, "%Y-%m-%d") #change class of date variable from factor to date
```


#What is mean total number of steps taken per day

Make a histogram of the total number of steps taken each day


```r
library(ggplot2) #ggplot will be used for graphs plotting

ggplot(data, aes(date, steps)) + 
  geom_bar(stat = "identity", colour = "blue", fill = "blue", width = 0.7) + 
  labs(title = "Total Number of Steps Taken Each Day", 
       x = "Date", y = "Total Steps Taken")
```

```
## Warning: Removed 2304 rows containing missing values (position_stack).
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

Calculate and report the mean and median total number of steps taken per day


```r
totalSteps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
mean(totalSteps)
```

```
## [1] 9354.23
```

```r
median(totalSteps)
```

```
## [1] 10395
```


#What is the average daily activity pattern?
Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avgSteps <- aggregate(data$steps, list(interval = as.numeric(as.character(data$interval))), 
                      FUN = "mean", na.rm = TRUE)
colnames(avgSteps)[2] <- "meanSteps"

ggplot(avgSteps, aes(interval, meanSteps)) + 
  geom_line(color = "blue", size = 0.8) + 
  labs(title = "Time Series Plot of Average Steps Taken in 5-minutes Intervals", 
       x = "5-minutes Intervals", y = "Average Steps Taken")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avgSteps[which.max(avgSteps$meanSteps),]
```

```
##     interval meanSteps
## 104      835  206.1698
```

#Imputing missing values
Calculate and report the total number of missing values in the dataset

```r
sum(is.na(data))
```

```
## [1] 2304
```

Fill in all of the missing values in the dataset using the mean for that 5-minute interval.

Create a new dataset that is equal to the original dataset but with the missing data filled in using the mean for the 5-minute interval. 


```r
imputedData <- data 

for (i in 1:nrow(imputedData)) {
  if (is.na(imputedData$steps[i])) {
    imputedData$steps[i] <- avgSteps[which(imputedData$interval[i] == avgSteps$interval), ]$meanSteps
  }
}

sum(is.na(imputedData))
```

```
## [1] 0
```

Make a histogram of the total number of steps taken each day 


```r
ggplot(imputedData, aes(date, steps)) + 
  geom_bar(stat = "identity", colour = "blue", fill = "blue", width = 0.7) + 
  labs(title = "Total Steps Taken Each Day (imputed missing data)", 
       x = "Date", y = "Total Steps Taken")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

Calculate and report the mean and median total number of steps taken per day. 

```r
newTotalSteps <- aggregate(imputedData$steps, 
                           list(Date = imputedData$date), 
                           FUN = "sum")$x

newMean <- mean(newTotalSteps)
newMedian <- median(newTotalSteps)

print(newMean)
```

```
## [1] 10766.19
```

```r
print(newMedian)
```

```
## [1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
newMean - mean(totalSteps, na.rm=TRUE)
```

```
## [1] 1411.959
```

```r
newMedian - median(totalSteps, na.rm=TRUE)
```

```
## [1] 371.1887
```

Mean and median values are higher after imputing missing data. 
By default, the days with NAs in the original dataset are set as 0. Imputing the missing data  with the mean of the 5-minutes intervals increases the total and the mean. The median was changed as a results as well. 

#Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
imputedData$weekdays <- factor(format(imputedData$date, "%A"))
levels(imputedData$weekdays)
```

```
## [1] "Friday"    "Monday"    "Saturday"  "Sunday"    "Thursday"  "Tuesday"  
## [7] "Wednesday"
```

```r
levels(imputedData$weekdays) <- list(weekday = c("Monday", "Tuesday",
                                                 "Wednesday", 
                                                 "Thursday", "Friday"),
                                     weekend = c("Saturday", "Sunday"))
table(imputedData$weekdays)
```

```
## 
## weekday weekend 
##   12960    4608
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
avgSteps <- aggregate(imputedData$steps, 
                      list(interval = as.numeric(as.character(imputedData$interval)), 
                           weekdays = imputedData$weekdays),
                      FUN = "mean")
colnames(avgSteps)[c(2:3)] <- c("day", "meanSteps")


ggplot(avgSteps, aes(interval, meanSteps)) + 
  geom_line(color = "blue", size = 0.8) + 
  facet_grid(day ~ .) + 
  labs(title = "Time Series Plot of Total Steps Taken in 5-minutes Intervals by Weekdays & Weekends", 
       x = "5-minutes Intervals", y = "Total Steps Taken")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
