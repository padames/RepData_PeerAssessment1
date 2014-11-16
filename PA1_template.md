---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

Under the assumption that the target file is in the local
directory where the script is running the following code 
loads it into memory:


```r
defaultFileName <- "activity.csv"
if( !file.exists( file.path(defaultFileName))) {
    defaultFileName <- unzip( zipfile="activity.zip" )  
}
loaded_data <- suppressWarnings(read.csv(file = defaultFileName, 
                                 header=TRUE,
                                 quote='"',
                                 stringsAsFactors=FALSE,
                                 encoding="UTF-8"))
rm(defaultFileName)
```
The result is a data frame called `loaded_data`:


```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```
A few of the first, last, and some intermediate rows look as follows:

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
```

```
##       steps       date interval
## 12308     7 2012-11-12     1735
## 12309    41 2012-11-12     1740
## 12310     7 2012-11-12     1745
```

```
##       steps       date interval
## 17566    NA 2012-11-30     2345
## 17567    NA 2012-11-30     2350
## 17568    NA 2012-11-30     2355
```
Before proceeding it is a good idea to transform the dates from strings 
into date format.

```r
clean_data <- loaded_data
clean_data$date <- as.Date(x= clean_data$date,format="%Y-%m-%d")
str(clean_data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?

First a histogram of steps per day was generated with ggplot
`geom_bar`:


```r
require(ggplot2)
## The data is already in the long format so ggplot can be used to generate the
## histogram
ggplot(data=clean_data, aes(x=date,y=steps)) +
    geom_bar(stat="identity") +
    labs(title="Number of steps per day") +
    labs(x= "Date", y= "Number of steps") 
```

```
## Warning: Removed 2304 rows containing missing values (position_stack).
```

![plot of chunk :plot steps per day](figure/:plot steps per day-1.png) 

Then the mean and median of the average number of steps per day
was computed from a transformation of the `steps`vector
of the `clean_data` data frame using the `mean` function
and the common date for each entry as the criterion for filtering
using the R function `tapply`.


```r
count_steps_per_day <-tapply(X=clean_data$steps,INDEX=clean_data$date,sum)
```
Finally the mean and median of the steps per day are computed:


```r
mean_steps_per_day <- mean(count_steps_per_day, na.rm=TRUE)
median_steps_per_day <- median(count_steps_per_day, na.rm=TRUE)
```
The values are as follows:


```r
options(digits=2)
paste( "Mean steps per day  = " , mean_steps_per_day )
```

```
## [1] "Mean steps per day  =  10766.1886792453"
```

```r
paste( "Median steps per day = ", median_steps_per_day )
```

```
## [1] "Median steps per day =  10765"
```

```r
# avoid side effects after this step
rm(count_steps_per_day,mean_steps_per_day, median_steps_per_day)
```

These values check with those plotted in the histogram according to visual
inspection of the y-axis values.

## What is the average daily activity pattern?

The original data frame has time intervals of 5 seconds that span
24 hours (24*60/5=288 time markers) for 61 days (31 in October and 30 in November). 




```r
plot(x=names(steps_per_interval),
     y=steps_per_interval,
     xlab="Daily intervals",
     ylab="steps",
     main="Activity pattern as steps per daily time interval",
     type="l",
     ylim=c(-5,205),
     pch=7,
     axes=FALSE)

## prepare labels  for x axis in this plot
lbls<- as.character(clean_data$interval[0:288])
insertColon <- function(l){
    nchars <- nchar(l)
    if( nchars < 2 ) {
        paste0("00:0",l)
    }
    else if ( nchars < 3 ) {
        paste0("00:", l)
    }
    else if( nchars < 4 ) {
        paste0("0", substr(l,1,1),":", substr(l,2,3) )
    }
    else if( nchars < 5 ) {
        paste0( substr(l,1,2), ":", substr(l,3,4))
    }
    else
        ""
}
prettylbls <- lapply(lbls,FUN=insertColon)

Axis(side=1,
     at=second_vector_per_day[c(1,25,50,75,100,125,150,175,200,225,250,275,288)],
     labels= prettylbls[c(1,25,50,75,100,125,150,175,200,225,250,275,288)],
     col='blue',
     col.ticks = 'red',
     col.axis = 'blue',
     tck = 0.03,
     xaxs="r")

Axis(side=3,
     at=second_vector_per_day[c(1,25,50,75,100,125,150,175,200,225,250,275,288)],
     labels= c("","","","","","","","","","","","",""),
     col = 'blue',
     col.ticks = 'red',
     col.axis = 'blue',
     las=2,
     tck = 1, # 100% of width or "grid lines on"
     lty= "dotted",
     xaxs="r")


Axis(side=2,
     at=c(0,50,100,150,200),
     labels= c("0","50","100", "150", "200"),
     col='blue',
     col.ticks = 'red',
     col.axis = 'blue',
     tck = 0.03,
     xaxs="r")

Axis(side=4,
     at=c(0,50,100,150,200),
     labels= c("","","", "", ""),     
     col = 'blue',
     col.ticks = 'red',
     col.axis = 'blue',
     las=2,
     tck = 1, # 100% of width or "grid lines on"
     lty= "dotted",
     xaxs="r")
```

![plot of chunk : plot activity pattern](figure/: plot activity pattern-1.png) 

Let's compute the values for the peak number of steps seen in the plot
occurring shortly after 8:15 on average during the two-month period of the
study.


```r
max(steps_per_interval)
```

```
## [1] 206.1698
```

```r
hour_of_day_for_max_steps <- floor(as.numeric(names(which.max(x=steps_per_interval)))/60)
minutes_of_day_for_max_steps <- as.numeric(names(which.max(x=steps_per_interval))) %% 60
```

The maximum average number of steps is 206.17 
and occurs at time 
8:35 of the day on average.


## Imputing missing values

The number of days that contain `NA` can be computed with a table
over a logical vector testing for `NA` in the vector that
contains the count of the number of steps per day.


```r
count_steps_per_day <-tapply(X=clean_data$steps,INDEX=clean_data$date,sum)
table(is.na(count_steps_per_day))
```

```
## 
## FALSE  TRUE 
##    53     8
```

There are 8 days that had `NA` data in them.

However the total number of actual `NA` readings can be computed with th following 
similar R code:


```r
table(is.na(clean_data$steps))
```

```
## 
## FALSE  TRUE 
## 15264  2304
```
So there are 2304 `NA` entries in the `step`
column.

They can be filled in with the average number of steps per interval computed in the
previous step.

## Are there differences in activity patterns between weekdays and weekends?
