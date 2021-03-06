---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---



## Loading and preprocessing the data

```r
unzip('activity.zip')
dataSet <- read.csv('activity.csv')
df <- dataSet[with(dataSet, !is.na({steps})), ]
```

## What is mean total number of steps taken per day?
* Calculate the total number of steps taken per day
* Make a histogram of the total number of steps taken each day
* Calculate and report the mean and median of the total number of steps taken per day

```r
steps_per_day <- aggregate(steps ~ date, df, sum)
colnames(steps_per_day) <- c("date","steps")
ggplot(steps_per_day, aes(x = steps)) + 
       geom_histogram(fill = "red", binwidth = 1000) + 
        labs(title="Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times in a day(Count)")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

### Mean and Median

```r
steps_day_mean <- mean(steps_per_day$steps)
steps_day_median <- median(steps_per_day$steps)
```
* Mean: 1.0766189\times 10^{4}
* Median:  10765

## What is the average daily activity pattern?
* Make a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avg <- tapply(dataSet$steps, dataSet$interval, mean, na.rm=TRUE)
plot(names(avg), avg, xlab="5-min interval", type="l", ylab="Average no. of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
maxavg <- max(avg)
maxinterval <- as.numeric(names(avg)[which(avg==max(avg))])
```
* Max Avg. Value 206.1698113
* Interval 5-min 835

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA|NAs)

```r
totalNAs <- sum(is.na(dataSet$steps))
```
Total NAs: 2304

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
# creating a copy of data set so that the missing value can be imputed in it
imputedata <- dataSet

# Devise a strategy for filling in all of the missing values in the dataset.
# In place of NA, using the mean for that 5-minute interval.
imputedata$steps[which(is.na(dataSet$steps))]<- as.vector(avg[as.character(dataSet[which(is.na(dataSet$steps)),3])])
```

* Create a new dataset that is equal to the original dataset but with the missing data filled in.
* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
stepseachday <- tapply(imputedata$steps, imputedata$date, sum, na.rm=TRUE)
qplot(stepseachday, xlab="No. of Steps Taken Each Day", ylab="Total Frequency", binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
mean_each_day_imputed <- mean(stepseachday)
median_each_day_imputed <- median(stepseachday)
```
Mean: 1.0766189\times 10^{4}
Median: 1.0766189\times 10^{4}


## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

* Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
imputedata$dayType<- ifelse(as.POSIXlt(imputedata$date)$wday %in% c(0,6), "weekends","weekdays")
```

*Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
aggregateData<- aggregate(steps ~ interval + dayType, data=imputedata, mean)
ggplot(aggregateData, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dayType ~ .) +
    xlab("5-minute interval") + 
    ylab("avarage number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->
