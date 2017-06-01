# Data analysis from a personal activity monitoring device
Peggy Courtois  
31 May 2017  



<br> <br>

# Introduction

<br> 

The data are collected from a device at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data are available in the present GitHub commit (*activity.csv*)
The variables included in this dataset are:
- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

The processing done on the data include a variable (*time*) stating the time based on *date* and *interval* columns.


```r
data <- read.csv(file = "activity.csv")
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
hours     <- as.character(floor(data$interval/100))
minutes   <- as.character(data$interval %% 100)
time      <- paste(hours,minutes,sep=":")
date      <- paste(data$date,time,sep=" ")
data$time <- strptime(date,format = "%Y-%m-%d %H:%M")
```
<br> 

# Questions

<br> 

## What is the mean total number of steps taken per day?

<br> 

What follows shows a histogram of the total number of steps per day (*DailySteps*). The mean and the median of the total number of daily steps is indicated in the summary.


```r
DailySteps <- with(data,tapply(steps,date,sum,na.rm = FALSE))
hist(DailySteps,breaks = 20, col = "cyan", main="",xlab = "Total steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
summary(DailySteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##      41    8841   10760   10770   13290   21190       8
```

<br> 

## What is the average daily activity pattern?

<br> 

The maximum number of steps is obtained at 8:35 with 207 steps.


```r
xHours <- format( seq.POSIXt(as.POSIXct(Sys.Date()), as.POSIXct(Sys.Date()+1), by = "5 min"),"%H%M",tz = "GMT")
xHours <- xHours[1:288]
AveragedDailySteps <- with(data,tapply(steps,interval,mean,na.rm = TRUE))
plot(xHours,AveragedDailySteps, type = "l", lwd = 2, col="red")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
ind_max <- which(AveragedDailySteps==max(AveragedDailySteps))
AveragedDailySteps[ind_max]
```

```
##      835 
## 206.1698
```

```r
xHours[ind_max]
```

```
## [1] "0835"
```

<br> 

## Imputing missing values

<br> 

There are 2304 (*NbNA*) missing values (NA) in the *steps* variable.


```r
NbNA <- sum(is.na(data$steps))
```

<br>
We decide to replace the missing values by the mean for that 5-minute interval calculated previously. This is done in the new dataset (*NewData*).


```r
ind  <- which(is.na(data$steps) == TRUE)
NewData <- data
xHoursnum <- data$interval[1:288]
for(i in c(1:length(ind))){
    temp_ind <- which(xHoursnum == NewData$interval[ind[i]])
    NewData$steps[ind[i]] <- AveragedDailySteps[temp_ind]
}
```

The shape of the histogram is identical but not the frequency. For example, with the dataset with missing values, 10 days were achieved with 10,000 steps, while 15 days were achieved with the dataset with non-missing values. The mean, min and max are identical, but the quantiles and median differ.


```r
NewDailySteps <- with(NewData,tapply(steps,date,sum,na.rm = FALSE))
hist(NewDailySteps,breaks = 20, col = "cyan", main="",xlab = "Total steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
summary(NewDailySteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
```

<br>

## Are there differences in activity patterns between weekdays and weekends?

<br>

Additionally to the 4 variables, we created a new variable called *DayType*, indicating if the day is a weekday or not.


```r
weekdays        <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')
NewData$DayType <- factor((weekdays(NewData$time) %in% weekdays), levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
```

The following plot shows the difference in number of steps between weekend and weekdays.


```r
Weekday <- subset(NewData,DayType == "weekday")
Weekend <- subset(NewData,DayType == "weekend")
WeekdaySteps <- with(Weekday,tapply(steps,interval,mean))
WeekendSteps <- with(Weekend,tapply(steps,interval,mean))
par(mfrow = c(2,1))
plot(xHours,WeekdaySteps,type="l",col="blue",main="Weekday",ylim=c(0,200),xlab="Time",ylab="Averaged steps",lwd=2)
plot(xHours,WeekendSteps,type="l",col="blue",main="Weekend",ylim=c(0,200),xlab="Time",ylab="Averaged steps",lwd=2)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

