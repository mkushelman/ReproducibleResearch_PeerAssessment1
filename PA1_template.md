---
title: "Reproducible Research: Peer Assessment 1"
author: "Mark Kushelman"
output: 
  html_document:
    keep_md: true
---



### Loading libraries

```r
library(dplyr); library(ggplot2); library(ggpubr)
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

```
## Warning: package 'ggpubr' was built under R version 3.4.4
```

```
## Loading required package: magrittr
```


## Loading and preprocessing the data
1.  Load the data

```r
activity <- read.csv("activity.csv",colClasses=c("integer","Date","integer"))
dim(activity); names(activity)
```

```
## [1] 17568     3
```

```
## [1] "steps"    "date"     "interval"
```

2.  Prepare data  
* We use the following denotations:
1.  A0 - original activity
2.  A1 - subset of A0 with steps which are not NA
3.  A2 - subset of A0 with steps which are NA
4.  B0, B1, B2 - data sets which are gotten from A0, A1, A2 correspondingly by the following dplyr functions: at first, 'group by date' and then 'summarize using sum'
5.  C0 - data sets which are gotten from A0 by the following dplyr functions: at first, 'group by interval' and then 'summarize using mean'

```r
A0 <- activity
GroupByDateA0 <- group_by(A0, date)
SumStepsPerDayA0 <- summarize(GroupByDateA0, SUM=sum(steps, na.rm=TRUE))
B0 <- SumStepsPerDayA0
GroupByIntervalA0 <- group_by(A0, interval)
MeanStepsPerIntervalA0 <- summarize(GroupByIntervalA0, MEAN=mean(steps, na.rm=TRUE))
C0 <- MeanStepsPerIntervalA0

A1 <- filter(activity, !is.na(activity$steps))
GroupByDateA1 <- group_by(A1, date)
SumStepsPerDayA1 <- summarize(GroupByDateA1, SUM=sum(steps, na.rm=TRUE))
B1 <- SumStepsPerDayA1

A2 <- filter(activity, is.na(activity$steps))
GroupByDateA2 <- group_by(A2, date)
SumStepsPerDayA2 <- summarize(GroupByDateA2, SUM=sum(steps, na.rm=TRUE))
B2 <- SumStepsPerDayA2
```
* Check date in B0, B1 and B2    
By definition, is should be number of dates in B0 is equal number of dates in B1 plus number of dates in B2

```r
N0 <- nrow(B0); N1 <- nrow(B1); N2 <- nrow(B2) ## N0, N1, N2 are numbers of dates in B0, B1, B2
print(paste0("N0 = ",N0)); print(paste0("N1 = ",N1)); print(paste0("N2 = ",N2))
```

```
## [1] "N0 = 61"
```

```
## [1] "N1 = 53"
```

```
## [1] "N2 = 8"
```

Really N0 = N1 + N2. 

Because N2 is small we can see the dates in B2:

```r
print("Dates in B2:"); c(B2$date)
```

```
## [1] "Dates in B2:"
```

```
## [1] "2012-10-01" "2012-10-08" "2012-11-01" "2012-11-04" "2012-11-09"
## [6] "2012-11-10" "2012-11-14" "2012-11-30"
```
By the way we can note that the date column in B1 starts from 2012-10-02 (because the date column in B2 starts from 2012-10-01)


## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.  

**It means that for this part we continue with ```B1```**

1.  Calculate the total number of steps taken per day

* Now we prepare ```B1``` for histograms.

```r
Month <- format(B1$date,"%m")
B1$month <- as.numeric(Month)
```


2.  Make a histogram of the total number of steps taken each day


```r
GG <- ggplot(B1, aes(date, SUM))+ 
    geom_col(colour = "white", fill = "steelblue") + 
    facet_grid(. ~ month, scales = "free") +
    ggtitle("Histogram for B1: Total Number of Steps by dates")+
    xlab("Date (in total 53 days)")+ylab("Number of steps for day")
GG
```

![](PA1_template_files/figure-html/Histogram for B1-1.png)<!-- -->

```r
## Save the plot into pgn file
ggsave(filename <- "Histogram for B1.png", GG, width = 9, height = 6, dpi = 120, units = "in", device='png')
unlink(filename)
```

3.  Calculate and report the mean and median of the total number of steps taken per day

```r
m1 <- round(mean(B1$SUM),2)
m2 <- round(median(B1$SUM),2)
print(paste0("For B1 mean=", m1,"; median=",m2))
```

```
## [1] "For B1 mean=10766.19; median=10765"
```

## What is the average daily activity pattern?
1.  Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
* Note: This average **(mean)** is taken across all days in **October and November** for each 5-minute interval.  

**By definition it is the ```C0``` data set**

```r
head(C0)
```

```
## # A tibble: 6 x 2
##   interval   MEAN
##      <int>  <dbl>
## 1        0 1.72  
## 2        5 0.340 
## 3       10 0.132 
## 4       15 0.151 
## 5       20 0.0755
## 6       25 2.09
```

* Time series plot

```r
GG <- ggplot(C0,aes(x=interval,y=MEAN))+ geom_line(color = "blue", size = 0.9)+
    ggtitle("Time series: Average steps by 5-min interval")+
    xlab("5-minute intervals") +
    ylab("Average (across Oct and Nov) number of steps")
GG
```

![](PA1_template_files/figure-html/Time series-1.png)<!-- -->

```r
## Save the plot into pgn file
ggsave(filename <- "Time series.png", GG, width = 9, height = 6, dpi = 120, units = "in", device='png')
unlink(filename)
```



2.  Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
MAX <- max(C0$MEAN)
f <- filter (C0, MEAN == MAX)
print(paste0("The interval ",f$interval," contains this ",round(f$MEAN,2)," required maximum"))
```

```
## [1] "The interval 835 contains this 206.17 required maximum"
```

## Imputing missing values
1.  Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with **NA**s)

```r
print(paste0("The total number of missing values is ", sum(is.na(A0$steps))))
```

```
## [1] "The total number of missing values is 2304"
```

2.  Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.   
**Rule: For filling NA in ```steps``` we use the average for the steps in the ```B1``` data set.**

3.  Create a new dataset that is equal to the original dataset but with the missing data filled in.  
* Plan:
1.  For above average we use mean ```x``` for steps in ```B1``` 
2.  Fill in all ```B2``` steps with ```x```
3.  Bind ```B1``` with ```B2```


```r
D1 <- B1
D2 <- B2
## Step 1
x = mean(D1$SUM)
## Step 2
D2$SUM <- rep(x,nrow(D2))
Month <- format(D2$date,"%m")
D2$month <- as.numeric(Month)
## Step 3
D0 <- rbind(D1, D2)
D0 <- arrange(D0, date)
head(D0)
```

```
## # A tibble: 6 x 3
##   date         SUM month
##   <date>     <dbl> <dbl>
## 1 2012-10-01 10766  10.0
## 2 2012-10-02   126  10.0
## 3 2012-10-03 11352  10.0
## 4 2012-10-04 12116  10.0
## 5 2012-10-05 13294  10.0
## 6 2012-10-06 15420  10.0
```

* We build 3 plots:
1.  Histogram for B0
2.  Histogram for D0
3.  A line graph helping visually distinct B0 from B1   

Now we prepare ```B0``` and ```D0``` for graphs.

```r
B <- B0
Month <- format(B$date,"%m")
B$month <- as.numeric(Month)
B$type <- rep("B",nrow(B))
D0$type <- rep("D0",nrow(D0))
rbind.B.and.D0 <- rbind(B, D0)
```

* Build 2 histograms and 1 plot

```r
g0 <- ggplot(B, aes(date, SUM))+  
    geom_col(colour = "white", fill = "lightblue")+
    ggtitle("Histogram for B: Total Number of Steps by dates")+
    xlab("Date (in total 61 days)")+ylab("Number of steps for day")

g1 <- ggplot(D0, aes(date, SUM))+  
    geom_col(colour = "white", fill = "steelblue")+
    ggtitle("Histogram for D0: Total Number of Steps by dates")+
    xlab("Date (in total 61 days)")+ylab("Number of steps for day")

g <- qplot(date, SUM, data = rbind.B.and.D0, colour=type, geom = c("line","point"))+
    ggtitle("Plot for B&D0: Total Number of Steps by dates")+
    xlab("Date (in total 61 days)")+
    ylab("Number of steps for day")

GG <- ggarrange(g0, g1, g, ncol = 1, nrow = 3)
GG
```

![](PA1_template_files/figure-html/3 graphs-1.png)<!-- -->

On the ```type B``` plot we see 8 points with 0 values. These values correspond to all 8 dates in ```B2```. Values for these dates in ```type D0``` plot are 10766.19 which is ```x``` from the Step 1 of above Plan.


```r
## Save the plot into pgn file
ggsave(filename <- "3 graphs.png", GG, width = 9, height = 7, dpi = 120, units = "in", device='png')
unlink(filename)
```

Calculate and report the mean and median of the B0 and B1. Do these values differ?

```r
print(paste0("For B mean=", round(mean(B$SUM),2),"; median=",round(median(B$SUM),2)))
```

```
## [1] "For B mean=9354.23; median=10395"
```

```r
print(paste0("For D0 mean=", round(mean(D0$SUM),2),"; median=",round(median(D0$SUM),2)))
```

```
## [1] "For D0 mean=10766.19; median=10766.19"
```
**The mean/median for B differ from the mean/median for D0**

## Are there differences in activity patterns between weekdays and weekends?

1.  Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
W <- A0
WeekDay <- format(W$date,"%A")
W$WeekDayEnd <- as.factor(WeekDay)

levels(W$WeekDayEnd) <- list(
    weekday = c("Monday","Tuesday","Wednesday","Thursday","Friday"),
    weekend = c("Saturday", "Sunday"))

levels(W$WeekDayEnd)
```

```
## [1] "weekday" "weekend"
```

```r
names(W)
```

```
## [1] "steps"      "date"       "interval"   "WeekDayEnd"
```

```r
distinct(W,WeekDayEnd)
```

```
##   WeekDayEnd
## 1    weekday
## 2    weekend
```

2.  Make a panel plot containing a time serises plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

* Prepare the ```A0``` set for the plot (using mean for average). The prepared set is ```WeekDayEnd```

```r
WeekDay <- filter(W, WeekDayEnd == "weekday")
WeekEnd <- filter(W, WeekDayEnd == "weekend")

GroupByIntervalWeekDay <- group_by(WeekDay, interval)
GroupByIntervalWeekEnd <- group_by(WeekEnd, interval)

MeanStepsPerIntervalWeekDay <- summarize(GroupByIntervalWeekDay, MEAN=mean(steps, na.rm=TRUE))
MeanStepsPerIntervalWeekEnd <- summarize(GroupByIntervalWeekEnd, MEAN=mean(steps, na.rm=TRUE))

WeekDay <- MeanStepsPerIntervalWeekDay
WeekEnd <- MeanStepsPerIntervalWeekEnd
v <-  rep("weekday",nrow(WeekDay))
WeekDay$type <- v
v <-  rep("weekend",nrow(WeekEnd))
WeekEnd$type <- v

# WeekDayEndC0 <- rbind(WeekDay,WeekEnd)
rBind <- rbind(WeekDay,WeekEnd)
WeekDayEnd <- group_by(rBind, type)
```

* Plot the ```WeekDayEnd```

```r
GG <- qplot(interval, MEAN, data = WeekDayEnd, colour=type, geom = c("line","smooth"))+
    ggtitle("Week Days: Average steps by 5-min interval")+
    xlab("5-minute intervals")+
    ylab("Average (across Oct and Nov) number of steps")
GG
```

```
## `geom_smooth()` using method = 'loess'
```

![](PA1_template_files/figure-html/Week Days-1.png)<!-- -->

```r
## Save the plot into pgn file
ggsave(filename <- "Week Days.png", GG, width = 9, height = 6, dpi = 120, units = "in", device='png')
```

```
## `geom_smooth()` using method = 'loess'
```

```r
unlink(filename)
```
