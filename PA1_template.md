# Reproducible Research: Peer Assessment 1
Henk Bierman  
19 juni 2016  


## Code for reading in the dataset and/or processing the data

Required libraries for this assignment are set, and the raw data is read.

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
library(ggplot2)
activity <- read.csv("activitytest.csv")
```

## Histogram of the total number of steps taken each day

First calculate the total number of steps taken per day

```r
summary <- group_by(activity, date)
stepsPerDay <- as.data.frame(summarise(summary, steps = sum(steps, na.rm = TRUE)))
```

Then the histogram with Steps per Day is plotted.

```r
hist(stepsPerDay$steps, col = "blue", main = "Steps per Day", 
     xlab = "Number of Steps", ylab = "Number of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## Mean and median number of steps taken each day

The Mean of steps taken each day is

```r
mean(stepsPerDay$steps)
```

```
## [1] 9354.23
```

The Median of steps taken each day is

```r
median(stepsPerDay$steps)
```

```
## [1] 10395
```

## Time series plot of the average number of steps taken

First the average daily activity pattern is calculated

```r
summary <- group_by(activity, interval)
avgStepsPerDay <- as.data.frame(summarise(summary, avgSteps = mean(steps, na.rm = TRUE)))
```

Then the time series plot of the average number of steps taken is depicted

```r
with(avgStepsPerDay, plot(interval, avgSteps, "l", col = "blue"), 
     main = "Average steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

## The 5-minute interval that, on average, contains the maximum number of steps

The 5-minute interval which, on average across all the days in the dataset, contains the maximum number of steps

```r
filter(avgStepsPerDay, avgSteps == max(avgSteps, na.rm = TRUE))
```

```
##   interval avgSteps
## 1      835 206.1698
```

## Code to describe and show a strategy for imputing missing data

First the total number of missing values in the dataset (i.e. the total number of rows with NAs) is calculated:

```r
nrow(filter(activity, is.na(steps)))
```

```
## [1] 2304
```

Then all missing activity values are replaced by the overall mean of involved interval across all days 
The result is saved in a copy activity datafram called tidyActivity

```r
tidyActivity <- activity

for (i in 1:nrow(tidyActivity)) {
  if (is.na(tidyActivity[i,1])) {
    modulo <- i %% nrow(avgStepsPerDay)
    if (modulo == 0) {
      tidyActivity[i,1] <- avgStepsPerDay[nrow(avgStepsPerDay), 2]
    } else {
        tidyActivity[i,1] <- avgStepsPerDay[i %% nrow(avgStepsPerDay), 2]
      }
  }
}
```
## Histogram of the total number of steps taken each day after missing values are imputed

With the missing values of steps filled in, once again calculate the total number of steps taken per day

```r
summary <- group_by(tidyActivity, date)
stepsPerDay <- as.data.frame(summarise(summary, steps = sum(steps, na.rm = TRUE)))
```

And the histogram of Steps per Day looks as follows:

```r
hist(stepsPerDay$steps, col = "blue", main = "Steps per Day", 
     xlab = "Number of Steps", ylab = "Number of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

The Mean and median are now respectively:

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10766.19
```

It can be concluded that impact of imputing missing data on the estimates of the total daily number of steps seems to be that both the median and mean increase, and that the mean and median approach the same value

## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

First add column to tidyActivity indicating if a day is a weekend day or weekday

```r
tidyActivity$date <- as.Date(tidyActivity$date)
weekdays1 <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')
tidyActivity$wDay <- factor((weekdays(tidyActivity$date) %in% weekdays1), 
                   levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
```

Then the average interval activity pattern is calculated across averaged across all weekdays respectively weekend days:

```r
summary <- group_by(tidyActivity, interval, wDay)
avgStepsPerDay <- as.data.frame(summarise(summary, avgSteps = mean(steps, na.rm = TRUE)))
```

Finally a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number 
of steps taken, averaged across all weekday days respectively weekend days (y-axis)

```r
qplot(interval, avgSteps, data = avgStepsPerDay, geom = "line", facets = wDay ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)<!-- -->
