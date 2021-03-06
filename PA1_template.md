---
title: "PA1_template"
author: "Juan Eduardo Hernandez"
date: "16 de octubre de 2015"
output: html_document
---
## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This report makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Data

The data is included in the repo and was downloaded from Coursera

File: activity.csv
Dataset: Activity monitoring data [52K]
The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

date: The date on which the measurement was taken in YYYY-MM-DD format

interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


## Library needed

```{r,echo=TRUE}
library(ggplot2)
library(Hmisc)
```


## Loading and preprocessing the data

```{r,echo=TRUE}
activityData <- read.csv('activity.csv')
```

## Calculate the mean and median of total steps taken per day

1. First calculate the total step taken per day, ignoring the missing values (rm=TRUE)

```{r,echo=TRUE}
steps_per_day <- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)
head(steps_per_day)
```

2. Make a Histogram of the total number of steps taken each day

```{r, echo=TRUE}
qplot(steps_per_day, xlab='Total steps per day', ylab='Frequency', binwidth=500)
```

3. Calculate the mean and media of the total number of steps taken per day.

```{r,echo=TRUE}
steps_per_day_mean <- mean(steps_per_day)
steps_per_day_median <- median(steps_per_day)
steps_per_day_mean
steps_per_day_median
```

## What is the average daily activity pattern?

1. Calculate average steps for each interval for all days, ignorin the missing values.

```{r,echo=TRUE}
average_steps_interval <- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)
```

2. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r,echo=TRUE}
ggplot(data=average_steps_interval, aes(x=interval, y=meanSteps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 
```

3. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r,echo=TRUE}
max_interval <- average_steps_interval[which.max(average_steps_interval$meanSteps),1]
max_interval
```

## Imputing Missing Values

1. Calculate the total number of missing values.

```{r,echo=TRUE}
activity_NA <- sum(is.na(activityData))
activity_NA
```

2. Devise a strategy for filling in all of the missing values in the dataset.

Missing values were imputed by inserting the average value.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in

```{r,echo=TRUE}
imputed_data <- activityData
imputed_data$steps <- impute(activityData$steps, fun=mean)
```

4. Make a histogram of the total number of steps taken each day.

```{r,echo=TRUE}
steps_per_day_imputed <- tapply(imputed_data$steps, imputed_data$date, sum)
qplot(steps_per_day_imputed, xlab='Total steps per day (Imputed)', ylab='Frequency', binwidth=500)
```

5. Calculate and report the mean and median total number of steps taken per day.

```{r,echo=TRUE}
steps_per_day_imputed_mean <- mean(steps_per_day_imputed)
steps_per_day_imputed_median <- median(steps_per_day_imputed)
steps_per_day_imputed_mean
steps_per_day_imputed_median
```

6. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r,echo=TRUE}
mean_diff <- steps_per_day_imputed_mean - steps_per_day_mean 
med_diff <- steps_per_day_imputed_median - steps_per_day_median
mean_diff
med_diff
```

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{r,echo=TRUE}
#change this according to your locale
days_en <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
days_es <- c("lunes","martes","miércoles","jueves","viernes")
imputed_data$dayType <- as.factor(ifelse(is.element(weekdays(as.Date(imputed_data$date)),days_es), "Weekday", "Weekend"))
```

2. Make a panel plot containing a time series plot

```{r,echo=TRUE}
avg_dayType_imputed <- aggregate(steps ~ interval + dayType, data=imputed_data, mean)
ggplot(avg_dayType_imputed, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dayType ~ .) +
    xlab("5-minute interval") + 
    ylab("average number of steps")
```
