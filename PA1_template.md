---
title: "Week 2 assignment"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(knitr)
library(dplyr)
library(ggplot2)
library(Hmisc)
```

## Loading and preprocessing the data

If not present, create data folder:

```{r data folder }
if(!file.exists("./data"))
    {dir.create("./data")}
```

If not present dowload zip data, if not already done unzip it:

```{r download zip}
URLfile<-"https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
if(!file.exists("./data/repdata-data-activity.zip")){
    download.file(URLfile, "./data/repdata-data-activity.zip", method = "curl")
}

if(!file.exists("./data/activity.csv")){
    unzip("./data/repdata-data-activity.zip", exdir = "./data")
}
```

Reading data into R:

```{r reading, cache=TRUE}
activity <- read.csv("./data/activity.csv")

```

## What is mean total number of steps taken per day?

Plotting histogram of total number of steps per day and computing mean and median:

```{r stepsperday, cache=TRUE}
dates<-unique(activity$date)
stepsperday<-vector()
datesrep<-vector()

for(current in dates){
    c<-sum(activity[activity$date==current,]$steps)
    
    if(!is.null(c) & !is.na(c) & !is.nan(c)){
        datesrep<- c(datesrep, rep(current,c))
        stepsperday<- c(stepsperday,c)
    }
}
mea <- as.character(mean(as.numeric(stepsperday), na.rm = TRUE))
med <- as.character(median(as.numeric(stepsperday), na.rm = TRUE))
qplot(datesrep, xlab = "Day", ylab = "Steps per day") + theme(axis.text.x = element_text(angle = 45, hjust = 1)) 
```

Total average number of steps taken per day is `r mea`, the median is `r med`

##What is the average daily activity pattern?

Making a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days

```{r pattern}
groPattern <- group_by(activity, interval)
groPattern <- summarise(groPattern, Ave.Step.Interval = mean(steps,na.rm = TRUE))
qplot(groPattern$interval, groPattern$Ave.Step.Interval, xlab = "Interval of day",ylab = "Average number of steps")
maxInt <- groPattern[which.max(groPattern$Ave.Step.Interval),]$interval

```

Maximum number of steps on average happens between `r maxInt` and `r maxInt + 5`

##Imputing missing values

There are a number of days/intervals where there are missing values. The presence of missing days may introduce bias into some calculations or summaries of the data.

Computing the total number of rows with NAs:

```{r NAnumber}
good <- !complete.cases(activity)
totNA <- sum(good)
NAsteps <- sum(is.na(activity$steps))
NAint <- sum(is.na(activity$interval))
NAdate <- sum(is.na(activity$date))
```

There are `r totNA` rows with NAs values, precisely:

- `r NAsteps` NAs in column "steps"

- `r NAint` NAs in column "interval"

- `r NAdate` NAs in column "date"

Filling in missing values in a new dataset (the steps value is filled with the mean of steps in the interval):

```{r NAfilling, cache=TRUE}
activityfilled<-activity

for(i in 1:length(activity$steps)){
    if(is.na(activity[i,]$steps)){
        activityfilled[i,]$steps <- groPattern[groPattern$interval == activity[i,]$interval, ]$Ave.Step.Interval
    }
}

good <- !complete.cases(activityfilled)
totNA <- sum(good)
NAsteps <- sum(is.na(activityfilled$steps))
NAint <- sum(is.na(activityfilled$interval))
NAdate <- sum(is.na(activityfilled$date))
```

Now there are `r totNA` rows with NAs values, precisely:

- `r NAsteps` NAs in column "steps"

- `r NAint` NAs in column "interval"

- `r NAdate` NAs in column "date"

Here's a histogram of the total number of steps taken each day after imputing NAs:

```{r stepsperdayfilled, cache=TRUE}

stepsperdayfilled<-vector()
datesrepfilled<-vector()

for(current in dates){
    c<-sum(activityfilled[activityfilled$date==current,]$steps)
    
    if(!is.null(c) & !is.na(c) & !is.nan(c)){
        datesrepfilled<- c(datesrepfilled, rep(current,c))
        stepsperdayfilled<- c(stepsperdayfilled,c)
    }
}
meafil <- as.character(mean(as.numeric(stepsperdayfilled), na.rm = TRUE))
medfil <- as.character(median(as.numeric(stepsperdayfilled), na.rm = TRUE))
qplot(datesrepfilled, xlab = "Day", ylab = "Steps per day") + theme(axis.text.x = element_text(angle = 45, hjust = 1)) 
```

Total average number of steps taken per day, after NAs filling, is `r meafil`, the median is `r medfil`

It's possible to notice:

- There are more days in the histogram, they now appear because NAs have been filled;

- The mean didn't changed, because we filled NAs with the average of the interval;

- The median took the value of the mean, since now there are many days having this value.

##Are there differences in activity patterns between weekdays and weekends?

Comparison between the pattern of step per interval in weekdays and in weekend:

```{r comparison}
activityfilled <- mutate(activityfilled, date = as.Date(date))
activityfilled <- mutate(activityfilled, day.type = factor((weekdays(date) %in% c("Saturday","Sunday")), labels = c("weekday","weekend")))
groPatternfil <- group_by(activityfilled, day.type, interval)
groPatternfil <- summarise(groPatternfil, Ave.Step.Interval = mean(steps,na.rm = TRUE))
qplot(interval, Ave.Step.Interval,data=groPatternfil , facets = . ~ day.type, xlab = "Interval of day",ylab = "Average number of steps") 
```
