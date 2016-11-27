# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

We're loading the data directly, without unzipping it


```r
activity <- read.csv(unz("activity.zip", "activity.csv"))
```

## What is mean total number of steps taken per day?

We're creating a new data frame with total amount of steps aggregated by date
and plotting a histogram of this data 


```r
total.by.date <- aggregate(steps ~ date, data = activity, sum)
hist(total.by.date$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

We can check out the mean and median of the total number of steps taken per day
by calculating some summary statistics of aggregated data frame


```r
summary(total.by.date$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10760   10770   13290   21190
```

So the mean is 10770, and the median is 10760

## What is the average daily activity pattern?

To explore the average daily activity pattern we'll first load dplyr package


```r
library(dplyr)
```

now we can easily group data frame and calculate average number of steps taken per interval, averaged across all days


```r
aver.by.interv <- activity %>% group_by(interval) %>% summarise(steps = mean(steps, na.rm = TRUE))
```

and make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis) using ggplot2 package (we'll load it first)


```r
library(ggplot2)
ggplot(aver.by.interv, aes( x = interval, y = steps)) + geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

we can see on the graph that the maximum number of steps happens in the interval somewhere between 7:50
(i.e. interval value = 750) and 10:00 (interval value = 1000)
for a precise calculation we'll use 'which.max' function subsetting our aggregated data frame 


```r
aver.by.interv[which.max(aver.by.interv$steps),]
```

```
## # A tibble: 1 × 2
##   interval    steps
##      <int>    <dbl>
## 1      835 206.1698
```

so we see that the maximum average number of steps is 206.17 and it happens in the 5-minute interval beginning at 8:35

## Imputing missing values

To check the total number of missing values in the dataset we'll calculate some simple summary statistics


```r
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```


## Are there differences in activity patterns between weekdays and weekends?
