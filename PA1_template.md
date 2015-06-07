# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
I load the data from activity.csv and convert the date collumn to class=date.

```r
library(dplyr)
steps.df <- read.csv("activity.csv", stringsAsFactors = FALSE)
steps.df$date <-as.Date(steps.df$date)
```

## What is mean total number of steps taken per day?

```r
stepsPerDay <- aggregate(steps ~  date, data=steps.df, sum)
hist(stepsPerDay$steps, breaks=100, main="Histogram of Steps per Day", xlab = "", ylim=range(0:5))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

```r
stepsPerDay.mean <- as.character(round(mean(stepsPerDay$steps)))
stepsPerDay.median <- as.character(median(stepsPerDay$steps))
```
The mean steps per day are 10766 and the median is 10765.

## What is the average daily activity pattern?

```r
stepsPerDay <- aggregate(steps ~  date, data=steps.df, sum)
hist(stepsPerDay$steps, breaks=100, main="Histogram of Steps per Day", xlab = "", ylim=range(0:5))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

```r
stepsPerDay.mean <- mean(stepsPerDay$steps)
stepsPerDay.median <- median(stepsPerDay$steps)
stepsPerInterval <- aggregate(steps ~  interval, data=steps.df, mean)
plot(stepsPerInterval, type="l", main="Average Steps per Daily 5 Minute Time Interval", 
     at=seq(0,2400,200))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-2.png) 

```r
IntervalWithMostSteps <-stepsPerInterval[stepsPerInterval$steps==max(stepsPerInterval$steps),]
```
The interval with the most steps is 835 with 206.17 steps.

## Imputing missing values

```r
NACount <- sum(is.na(steps.df$steps))
```
There are 2304 records with missing/NA step values.
There are also many records with 0 values, but we do not consider those to be missing because it should mean that the person did not take any steps.


```r
#Make a copy of data set and replace NA value with the average steps for that intval
NAfilled.df <- steps.df

for (i in 1:nrow(NAfilled.df)) {
  if (is.na(NAfilled.df[i,]$steps)) {
    meanSteps <- stepsPerInterval[stepsPerInterval$interval==NAfilled.df[i,]$interval,]
    NAfilled.df[i,]$steps <- meanSteps$steps
  }
}

stepsPerDay2 <- aggregate(steps ~  date, data=NAfilled.df, sum)
hist(stepsPerDay2$steps, breaks=100, main="Histogram of Steps per Day", xlab = "", ylim=range(0:5))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

```r
stepsPerDay2.mean <- as.character(round(mean(stepsPerDay2$steps)))
stepsPerDay2.median <- as.character(round(median(stepsPerDay2$steps)))
```
The mean steps per day for this data set are 10766 and the median is 10766. There is very little difference between the two datasets. The mean is the same,and in the second dataset the median is just one step higher and now equals the mean. I expected there to be more difference between the datasets so I wonder if I have made a mistake in my analysis. 

## Are there differences in activity patterns between weekdays and weekends?

```r
#Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
NAfilled.df$weekday <- ifelse((weekdays(NAfilled.df$date)=="Sunday") | 
          (weekdays(NAfilled.df$date)=="Saturday"), "Weekend", "Weekday")

stepsPerInterval.weekend <- aggregate(steps ~  interval, data=NAfilled.df[NAfilled.df$weekday=="Weekend",], mean)
stepsPerInterval.weekday <- aggregate(steps ~  interval, data=NAfilled.df[NAfilled.df$weekday=="Weekday",], mean)

par(mfrow = c(2, 1))

plot(stepsPerInterval.weekday, type="l", main="Weekday: Average Steps per Daily 5 Minute Time Interval", 
     axes = FALSE)
axis(side = 2, at = c(50,100,150,200))
axis(side = 1, at = seq(0,2400,200))

plot(stepsPerInterval.weekend, type="l", main="Weekend: Average Steps per Daily 5 Minute Time Interval", axes = FALSE)
axis(side = 2, at = c(50,100,150,200))
axis(side = 1, at = seq(0,2400,200))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 


I can see several differences between the trends on weekdays and weekends. On weekdays the steps begin around 5:30 AM, and on the weekends they do not reach that same level until around 8:00 am. 

Weekends and weekdays both spike around 8:15 am, though on weekends there are secondary spikes around noon and 4:00 PM. And on weekends activity is very low after 9:00 PM but on weekends there is a final skike around 10:00 PM.
