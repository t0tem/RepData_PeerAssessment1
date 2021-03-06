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
```

And here is a histogram of the total number of steps taken each day

```r
hist(total.by.date$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

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

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

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

so we can see that 'steps' variable is the only one with missing values and there are 2304 of them

### *The strategy for filling in all of the missing values in the dataset*
We'll fill in missing values with the mean of its' 5-minute interval. To do that we'll use again 'group_by' from 
dplyr package together with 'mutate' and 'replace'. The result will be directly assigned to a new data frame 'activity.no.NA'


```r
activity.no.NA <- activity %>% 
      group_by(interval) %>%
      mutate(steps = replace(steps, is.na(steps), mean(steps, na.rm = TRUE))) %>%
      as.data.frame()
```

So now we can repeat the step of aggregating this new data frame by date with total number of steps
and plotting a histogram of this data 


```r
total.by.date.no.NA <- aggregate(steps ~ date, data = activity.no.NA, sum)
hist(total.by.date.no.NA$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Let's calculate some summary statistics for the total number of steps taken each day

```r
summary(total.by.date.no.NA$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
```

recalling the initial result of dataset with missing values

```r
summary(total.by.date$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10760   10770   13290   21190
```

As we can see our strategy of imputing missing values impacted the Median of dataset (changed from 10760 to 10770) but not the Mean (stayed equal 10770)


## Are there differences in activity patterns between weekdays and weekends?
To address this question we'll need to add a new factor variable 'day' with two levels – “weekday” and “weekend”
indicating whether a given date is a weekday or weekend day.

first we'll create a vector of weekend days names

```r
wknd <- c("Sat", "Sun")
```

And then we'll add a new variable 'day' using 'mutate' from dplyr package paired with 'weekdays' function

```r
activity.no.NA <- activity.no.NA %>%
      mutate(date = as.Date(date)) %>%
      mutate(day = factor(weekdays(date, abbreviate = TRUE) %in% wknd, 
                          levels = c(FALSE, TRUE), labels = c("weekday", "weekend"))) %>%
      as.data.frame()
```

So we can now create a new data frame aggregated by weekday and interval with average number of steps taken

```r
total.by.wday <- aggregate(steps ~ interval + day, data = activity.no.NA, mean)
```

And make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days. We'll use 'ggplot' for that

```r
ggplot(total.by.wday, aes( x = interval, y = steps)) + geom_line() + facet_grid(day~.)
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png)<!-- -->



