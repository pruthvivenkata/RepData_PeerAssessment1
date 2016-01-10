
## Loading and preprocessing the data

```r
ActivityData = read.csv("activity.csv", header = TRUE, sep = ",")
ActivityData_NoNA=ActivityData[complete.cases(ActivityData),]
ActivityData_NoNA$date = as.Date(ActivityData_NoNA$date)
```

## What is mean total number of steps taken per day?  

1.What is mean total number of steps taken per day?

```r
Activitydata_Grouped= aggregate(ActivityData_NoNA$steps,
                                by=list(date=ActivityData_NoNA$date),FUN=sum)
```
2.f you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
library("ggplot2")
```

```
## Warning: package 'ggplot2' was built under R version 3.2.3
```

```r
qplot(x,data=Activitydata_Grouped,xlab = "Total Number of Steps Taken Each Day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)\
3.Calculate and report the mean and median of the total number of steps taken per day

```r
summary(Activitydata_Grouped$x)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10760   10770   13290   21190
```
The mean number of steps per day= 10,760.The median number of steps per day= 10,770.

## What is the average daily activity pattern?  

1.Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
Activitydata_Grouped_Interval= aggregate(ActivityData_NoNA$steps,
                                         by=list(Interval=ActivityData_NoNA$interval),FUN=mean)
plot(Activitydata_Grouped_Interval$Interval,Activitydata_Grouped_Interval$x,
     type = 'l',xlab ="Interval", ylab ="Average number of steps taken for all days ")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)\
2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
Activitydata_Grouped_Interval[Activitydata_Grouped_Interval$x==
                                      max(Activitydata_Grouped_Interval$x),]
```

```
##     Interval        x
## 104      835 206.1698
```
It is the 835 5 minute interval that contains the maximum steps of 206.16

## Imputing missing values 

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

```r
sum(!complete.cases(ActivityData))
```

```
## [1] 2304
```
There are 2304 rows where there are NAs

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
ActivityData_merged=merge(x=ActivityData, y=Activitydata_Grouped_Interval,
                          by.x ="interval",by.y = "Interval", all.x=TRUE)
ActivityData_merged$steps[which(is.na(ActivityData_merged$steps))]=
        ActivityData_merged$x[which(is.na(ActivityData_merged$steps))]
```
The mean number of steps for 5 minute interval across all days has been used for imputation and the new data set hase been created(ActivityData_merged).

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
ActivityData_merged_Grouped= aggregate(ActivityData_merged$steps,
                                       by=list(date=ActivityData_merged$date),FUN=sum)
library("ggplot2")
qplot(x,data=ActivityData_merged_Grouped,xlab = "Total Number of Steps Taken Each Day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)\

```r
summary(ActivityData_merged_Grouped$x)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
```
The data was very slightly left skewed before imputation. Now, after imputation, the data is normally distributed. This point can be re-iterated by observing that both mean and median are equal
 
## Are there differences in activity patterns between weekdays and weekends?
 
 1.Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
 ActivityData_merged$date= as.Date(ActivityData_merged$date)
ActivityData_merged$Weekday= factor((weekdays(ActivityData_merged$date)  %in%
                                             c('Monday','Tuesday','Wednesday','Thursday','Friday')),levels=c(FALSE,TRUE), 
                                    labels= c('weekend','weekday'))
```
2.Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
ActivityData_merged_grouped_IntWeekday=ActivityData_merged %>% group_by(interval,Weekday) %>% summarize(Average_steps=mean(steps))
qplot(interval,Average_steps,data= ActivityData_merged_grouped_IntWeekday,geom ="line",facets =Weekday~.)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)\


