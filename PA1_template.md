# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
raw <- read.csv("activity.csv", na.strings = "NA")
dates <- unique(raw$date)
intervals <- unique(raw$interval)
```


## What is mean total number of steps taken per day?

```r
### DATES ###

# remove NA values from initial dataset, keep only values without NAs for column steps
stepsOfDay <- function(date) {
  stepsOfDayForDataset(date, raw)
}

# remove NA values from initial dataset, keep only values without NAs for column steps
stepsOfDayForDataset <- function(date, dataset) {
  temp <- dataset[which(dataset$date == date), ]
  temp[!is.na(temp$steps),]
}

# sum of steps for given date
sumOfStepsForDay <- function(date){
  sumOfStepsForDayForDataset(date, raw)
}

# sum of steps for given date
sumOfStepsForDayForDataset <- function(date, dataset){
  temp <- stepsOfDayForDataset(date, dataset)
  sum(temp$step)
}

# vecor of sum of steps for all dates
sumOfStepsForAllDays <- function(){
  sumOfStepsForAllDaysForDataset(raw)
}

# vecor of sum of steps for all dates
sumOfStepsForAllDaysForDataset <- function(dataset){
  dat <- data.frame ()
  for (i in 1:(length(dates))){
    dat <- rbind(dat, sumOfStepsForDayForDataset(dates[i], dataset))
  }
  return (dat)
}

# mean of steps for given date
meanOfStepsForDay <- function(date){
  meanOfStepsForDayForDataset(date, raw)
}

# mean of steps for given date
meanOfStepsForDayForDataset <- function(date, dataset){
  steps <- stepsOfDayForDataset(date, dataset)
  mean(steps$step, na.rm = TRUE)
}

# median of steps for given date
medianOfStepsForDay <- function(date){
  medianOfStepsForDayForDataset(date, raw)
}

# median of steps for given date
medianOfStepsForDayForDataset <- function(date, dataset){
  steps <- stepsOfDayForDataset(date, dataset)
  median(steps$step, na.rm = TRUE)
}


# 1. Make a histogram of the total number of steps taken each day
hist(sumOfStepsForAllDays()[,1] , main = "Histogram of total Number of Steps taken each Day", xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

```r
# 2. Calculate and report the mean and median total number of steps taken per day
meantoprint <- round(mean(sumOfStepsForAllDays()[,1]),2)
mediantoprint <- round(median(sumOfStepsForAllDays()[,1]),2) 
print (paste("Mean total number of steps taken per day:", meantoprint ) )
```

```
## [1] "Mean total number of steps taken per day: 9354.23"
```

```r
print (paste("Median total number of steps taken per day:", mediantoprint ) )
```

```
## [1] "Median total number of steps taken per day: 10395"
```


## What is the average daily activity pattern?


```r
### INTERVALS ###

# only values without NAs for intervals
stepsOfInterval <- function(interval) {
  temp <- raw[which(raw$interval == interval),]
  temp[!is.na(temp$steps),]
}

# sum of steps for given interval
sumOfStepsForInterval <- function(interval){
  temp <- stepsOfInterval(interval)
  sum(temp$step)
}

sumOfStepsForAllIntervals <- function(){
  dat <- data.frame ()
  for (i in 1:(length(intervals))){
    dat <- rbind(dat, sumOfStepsForInterval(intervals[i]))
  }
  return (dat)
}

meanOfStepsForInterval <- function(interval){
  steps <- stepsOfInterval(interval)
  mean(steps$step, na.rm = TRUE)
}

meanOfStepsForAllIntervals <- function(){
  dat <- data.frame ()
  for (i in 1:(length(intervals))){
    dat <- rbind(dat, meanOfStepsForInterval(intervals[i]))
  }
  return (dat)
}

medianOfStepsForInterval <- function(interval){
  steps <- stepsOfInterval(interval)
  median(steps$step, na.rm = TRUE)
}

# 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
atvalues <- c(0,72,144,216,288)
labelvalues <- c("0:00", "6:00", "12:00", "18:00", "24:00")

plot(meanOfStepsForAllIntervals()[,1], type="l", main = "Average Number of Steps for any Interval", 
     xlab = "Interval (Time)", ylab = "Average Numbe of Steps", xaxt="n")
axis(side=1, at=atvalues, labels = labelvalues)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

```r
# 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
maxinterval <- which.max( meanOfStepsForAllIntervals()[,1] )
timeinterval <- intervals[maxinterval]
print (paste("Maximum value of steps is in Interval:", timeinterval, "(read as hh:mm)"))
```

```
## [1] "Maximum value of steps is in Interval: 835 (read as hh:mm)"
```

## Imputing missing values

```r
# 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

# getNumberOfNAForColumn <- function(column){
getNumberOfNA <- function(column = raw$steps){
  counter <- 0
  
  for (i in 1:(length(column))){
    if (is.na(column[i])) {
      counter <- counter +1
    }
  }
  
  return (counter)
}

print (paste("Total number of missing values in the dataset:", getNumberOfNA()))
```

```
## [1] "Total number of missing values in the dataset: 2304"
```

```r
# 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

# different strategies for filling in steps
fillInNA <- function(strategy = "basic"){
  steps <- raw$steps
  
  for (i in 1:(length(steps))){
    if (is.na(steps[i])) {
      # fill in
      if (strategy == "basic"){
        # basic: simply set to zero
        steps[i] <- 0
      }
      
      if (strategy == "meaninterval"){
        # mean: simply set to mean of interval
        steps[i]<- meanOfStepsForInterval(raw$interval[i])
      }
      if (strategy == "medianinterval"){
        # mean: simply set to median of interval
        steps[i]<- medianOfStepsForInterval(raw$interval[i])
      }          
    }
  }
  
  return(steps)  
}


# 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
createNewDataset <- function(strategy = "basic"){
  steps <- fillInNA(strategy)
  return(cbind(steps, raw[,2:3]))  
}
```

*Strategy a) 'basic' -> simlply set steps with NA as value to zero.*


```r
# fill dataset according to strategy
filled <- createNewDataset("basic")

meanraw <- round( mean(sumOfStepsForAllDays()[,1]) , 2)
meanfilled <- round( mean(sumOfStepsForAllDaysForDataset(filled)[,1]) , 2)
medianraw <- round( median(sumOfStepsForAllDays()[,1]) , 2)
medianfilled <- round( median(sumOfStepsForAllDaysForDataset(filled)[,1]) , 2)

print (paste("Mean of Steps for raw dates vs. basic filled data:", meanraw, "vs", meanfilled ))
```

```
## [1] "Mean of Steps for raw dates vs. basic filled data: 9354.23 vs 9354.23"
```

```r
print (paste("Median of Steps for raw dates vs. basic filled data:", medianraw, "vs", medianfilled ))
```

```
## [1] "Median of Steps for raw dates vs. basic filled data: 10395 vs 10395"
```

Not different because of use of argument na.rm = TRUE in calculation of mean / median for raw data.

*Strategy b) 'meaninterval' -> steps with NA value are set to mean of same interval.*


```r
# fill dataset according to strategy
filled2 <- createNewDataset("meaninterval")

meanfilled <- round( mean(sumOfStepsForAllDaysForDataset(filled2)[,1]) , 2)
medianfilled <- round( median(sumOfStepsForAllDaysForDataset(filled2)[,1]) , 2)

print (paste("Mean of Steps for raw dates vs. meaninterval filled data:", meanraw, "vs", meanfilled ))
```

```
## [1] "Mean of Steps for raw dates vs. meaninterval filled data: 9354.23 vs 10766.19"
```

```r
print (paste("Median of Steps for raw dates vs. meaninterval filled data:", medianraw, "vs", medianfilled ))
```

```
## [1] "Median of Steps for raw dates vs. meaninterval filled data: 10395 vs 10766.19"
```

Different because of setting NA values to mean value, so steps have greater values when missing.


```r
# 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

# histogram for second strategy
hist(sumOfStepsForAllDaysForDataset(filled2)[,1] , main = "Histogram of total Number of Steps taken each Day", xlab = "Steps filled with mean for interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

## Are there differences in activity patterns between weekdays and weekends?


```r
# 1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

# test if day is weekday - German localisation with 'Samstag' == 'Saturday', 'Sonntag' == 'Sunday'
isweekday <- function(day) {
    t <- weekdays(as.Date(day))
    if (t == "Samstag" | t == "Sonntag") {
      return (FALSE)
    } else { 
      return (TRUE)
    }
}

# return TRUE / FALSE for new weekday column
factorweekdays <- function(dataset){
  sapply(dataset$date,isweekday)
}

# return dataset with factor for weekday/weekend label
createFactor <- function(dataset = raw){
  f <- factorweekdays(dataset)
  dayofweek <- ifelse(f,"weekday","weekend")
  return(cbind(dataset,dayofweek))  
}

# create and show new dataset with factor column
datasetF <- createFactor()  
str(datasetF)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps    : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date     : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ dayofweek: Factor w/ 2 levels "weekday","weekend": 1 1 1 1 1 1 1 1 1 1 ...
```

```r
# 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

dsweekday <- datasetF[which(datasetF$dayofweek == "weekday"),]
dsweekend <- datasetF[which(datasetF$dayofweek == "weekend"),]
  
stepsweekday <- sumOfStepsForAllDaysForDataset(dsweekday)[,1]
stepsweekend <- sumOfStepsForAllDaysForDataset(dsweekend)[,1]

par(mfrow=c(2,1), mar = c(4,3,3,1))
  {
  plot(stepsweekday, t="l", xaxt="n", ylab = "Steps per Interval", 
       xlab = "Interval", main = "Steps on Weekdays")
  plot(stepsweekend, t="l", xaxt="n", ylab = "Steps per Interval", 
       xlab = "Interval", main = "Steps on Weekends")
  }
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 
