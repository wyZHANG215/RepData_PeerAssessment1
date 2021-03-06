---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
# unzip the file
unzip("activity.zip")
# load the data
PA1 <- read.csv("./activity.csv")
str(PA1)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
# handle the date
PA1$date <- as.Date(PA1$date)
```

## What is mean total number of steps taken per day?
1. Make a histogram of the total number of steps taken each day

```r
# calculate the steps per day
step_per_day <- tapply(PA1$steps,PA1$date, FUN = sum, na.rm = TRUE)
# plot the histogram
hist(step_per_day,
     main="Histogram of Steps Per Day",
     xlab="Number of Steps", 
     ylim = c(0,30))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

2. Calculate and report the mean and median total number of steps taken per day

```r
# get the mean from the summary
step_per_day_mean <- mean(step_per_day)
# get the median from the summary
step_per_day_median <- median(step_per_day)
# write the mean and median in variable to faciliate the sub title
legend_mean <- paste("mean =", as.integer(step_per_day_mean))
legend_median <- paste("median =", as.integer(step_per_day_median)) 
sub_title <- paste(legend_mean, ",", legend_median)
# plot
plot(step_per_day, 
    type="l",
    col="blue",
    xlab="Days",
    ylab="Number of Steps", 
    main="Daily Number of Steps", 
    sub = sprintf(sub_title))
abline(h=step_per_day_mean,col="red")
abline(h=step_per_day_median,col="green")
legend("topleft",
       legend=c("Number of Steps","Median","Mean"),
       col=c("blue","red","green"), 
       lty=1,
       cex = 0.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
# get the average for 5-minute interval
step_interval_5m <- aggregate(steps ~ interval, PA1, mean)
# get the interval with max steps
max_interval <- step_interval_5m[which.max(step_interval_5m$steps),1]
# get the steps for the max steps
max_average_step <- step_interval_5m[which.max(step_interval_5m$steps),2]
# prepare the maximum interval and steps data, put it in sub_title variable
sub_title <- paste("The maximum number of steps is",as.integer(max_average_step), "at interval", max_interval)
# plot
plot(step_interval_5m, 
    type="l",
    xlab="5-Minute Interval",
    ylab="Average Steps", 
    main="Average numbers of steps across Days", 
    sub = sprintf(sub_title))
# highlight the max interval point
points(max_interval, max_average_step, pch = 18, col = "red", cex = 1.5)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
missing_values <- sum(is.na(PA1$steps))
missing_values
```

```
## [1] 2304
```
Total missing value is 2304.

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
For missing values, fill it with the mean for the 5-minute interval.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
# get interval average
Interval_avg <- aggregate(x = list(steps = PA1$steps), by = list(interval = PA1$interval), FUN = mean, na.rm = TRUE)
# fill the missing data function, with 5-minute interval average
fill_data <- function(steps,interval) {
    fd <- NA
    if(!is.na(steps))
        fd <- c(steps)
    else
        fd <- (Interval_avg[Interval_avg$interval==interval,"steps"])
    fd
}    
# create the new dataset 
fill_PA1 <- PA1
fill_PA1$steps <- mapply(fill_data, fill_PA1$steps, fill_PA1$interval)
str(fill_PA1)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
# get the daily steps 
fill_step_per_day <- tapply(fill_PA1$steps,fill_PA1$date, FUN = sum, na.rm = TRUE)
# plot histogram with missing value
hist(fill_step_per_day,
     main="Histogram of Steps Per Day",
     xlab="Number of Steps", ylim = c(0,35), 
     sub="Missing value filled with the 5-minute interval average")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
# caculate mean and median total number of steps per day
mean(fill_step_per_day)
```

```
## [1] 10766.19
```

```r
median(fill_step_per_day)
```

```
## [1] 10766.19
```
After filling the missing data,

 - mean of total number of steps taken per day increased from 9354 to 10766.
 
 - median of total number of steps taken per day increased from 10395 to 10766.
 
## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:

```r
#create a new variable day_of_week
fill_PA1$day_of_week <- 
        ifelse(weekdays(fill_PA1$date,abbreviate=TRUE)=="Sat"|
               weekdays(fill_PA1$date,abbreviate=TRUE)=="Sun",
               "weekend",
               "weekday")
str(fill_PA1)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps      : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date       : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval   : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ day_of_week: chr  "weekday" "weekday" "weekday" "weekday" ...
```

```r
class(fill_PA1$day_of_week)
```

```
## [1] "character"
```

```r
# change the variable to factor
fill_PA1$day_of_week <- as.factor(fill_PA1$day_of_week)

#compute the mean value of steps across 5 minute time intervals
steps_by_wday <- with(fill_PA1, aggregate(steps ~ day_of_week + interval, FUN = mean, rstepm.na = TRUE))
steps_by_wday$steps <- round(steps_by_wday$steps, digits = 0)

# plot
library(ggplot2)
g <- ggplot(steps_by_wday, aes(interval, steps)) 
g + geom_line(aes(color = day_of_week)) + 
    scale_colour_discrete(name="days of the week") +
    labs(title="Acitvity patterns") +
    labs(x="5-minute Interval", y="Average Steps") + 
    facet_grid(day_of_week ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

In a nutshell, weekend steps are more consistent through out the day.

