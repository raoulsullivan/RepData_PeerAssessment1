Reproducible Research: Peer Assessment 1
========================================

First, set display options for knitr

```r
# numbers >= 10^5 will be denoted in scientific notation and rounded to 2
# digits
options(scipen = 1, digits = 2)
```


## Loading and preprocessing the data
The following code unzips the file 'activity.zip' and loads it into a data frame
via read.csv.

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?
The following code aggregates the steps variable of the data by the date
varible using the sum function. This is plotted in figure 1 below. It then takes the mean, stripping NA values.

```r
stepsperday <- aggregate(activity$steps, list(activity$date), sum)
mean1 <- mean(stepsperday$x, na.rm = T)
median1 <- median(stepsperday$x, na.rm = T)
hist(stepsperday$x, xlab = "Steps per day", main = "Fig. 1 - Histogram of steps per day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

The mean total number of steps per day is 10766.19. 
The median value is 10765.

## What is the average daily activity pattern?
The following code aggregates again, this time steps by interval and using the mean function (stripping NA values). The interval with the maximum number of steps is then extracted and plotted in figure 2 below.

```r
stepsperinterval <- aggregate(activity$steps, list(activity$interval), FUN = mean, 
    na.rm = T)
maxsteps <- max(stepsperinterval$x, na.rm = T)
maxinterval = stepsperinterval$Group.1[stepsperinterval$x == maxsteps]
plot(stepsperinterval$Group.1, stepsperinterval$x, type = "l", main = "Fig. 2 - Mean steps per 5 minute interval", 
    xlab = "5 minute interval", ylab = "# steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

The 5-minute interval, on average across all the days in the dataset, containing the maximum number of steps is interval number 835.

## Imputing missing values
The following code uses mapply to iterate over the data, passing the number of steps and interval for each row to function 'fillval'. Fillval replaces any NA values for number of steps with the mean number of steps for that interval as calculated previously. Figure 3 displays the result.

```r
countmissingvalues <- table(is.na(activity))[[2]]

# function to return mean values for intervals if there's an NA value
fillval <- function(value, interval) {
    if (is.na(value)) {
        # print('na')
        return(stepsperinterval$x[stepsperinterval$Group.1 == interval])
    } else {
        # print('not na')
        return(value)
    }
}

filledvalues <- activity
filledvalues$steps <- mapply(fillval, filledvalues$steps, filledvalues$interval)
str(filledvalues)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r

stepsperday2 <- aggregate(filledvalues$steps, list(filledvalues$date), sum)
mean2 <- mean(stepsperday2$x, na.rm = T)
median2 <- median(stepsperday2$x, na.rm = T)
hist(stepsperday2$x, xlab = "Steps per day", main = "Fig. 3 - Histogram of steps per day with missing values averaged out")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

The count of missing values in the original data is 2304.
The mean total number of steps per day with the missing values averaged out is 10766.19. 
The median value is 10766.19.
As can be seen, replacing missing data with mean values has no effect on the mean and median results.

## Are there differences in activity patterns between weekdays and weekends?
The following code uses weekdays and strptime on the data 'date' column to populate an 'iswekend' column, which is then coerced to a factor value. melt and dcast from the reshape2 library are then used to calculate means by interval and by isweekend, before lattice is used to create Figure 4 below.  

```r
# returns 1 if the date is a weekend, 0 if a weekday
isweekend <- function(date) {
    if (weekdays(strptime(date, "%F")) %in% c("Saturday", "Sunday")) {
        return(1)
    } else {
        return(0)
    }
}

# create isweekend column
weekdata <- activity
weekdata$isweekend <- sapply(weekdata$date, isweekend)
weekdata$isweekend <- factor(weekdata$isweekend, c(0, 1), c("weekday", "weekend"))

# melt and recast to get means
library(reshape2)
library(plyr)
library(lattice)
melty <- melt(weekdata, id = c("date", "interval", "isweekend"))
solid <- dcast(melty, isweekend + interval ~ variable, mean, na.rm = T)

xyplot(steps ~ interval | isweekend, solid, type = "l", layout = c(1, 2), main = "Fig. 4 - mean number of steps per five minute interval, broken by weekend or weekday")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

