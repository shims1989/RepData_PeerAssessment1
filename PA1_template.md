---
title: "Assignment of reproducible research: week2"
author: "Mingshu Shi"
date: "24 September 2017"
output: html_document
---



# Activity Monitoring
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Loading and preprocessing the data

### 1. Download and read raw data.  
In this step, the raw data was downloaded as zip file. The "unzip"" function was used and csv file were read by "read.csv"" function. Dataset was stored into "raw_dataset".  


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
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", "file.zip")
unzip("file.zip")
dataset <- read.csv("activity.csv")
head(dataset)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

## What is mean total number of steps taken per day?
### 2. Plot histogram for total number of steps taken each day.
In this step, firstly we need to summarise the total step that taken each day. We use "group_by" followed by "summarise" function from "dplyr" package to summrise data.

```r
summary_total_step <- dataset %>% group_by(date) %>% summarise(Total_steps = sum(steps))
print(summary_total_step)
```

```
## # A tibble: 61 x 2
##          date Total_steps
##        <fctr>       <int>
##  1 2012-10-01          NA
##  2 2012-10-02         126
##  3 2012-10-03       11352
##  4 2012-10-04       12116
##  5 2012-10-05       13294
##  6 2012-10-06       15420
##  7 2012-10-07       11015
##  8 2012-10-08          NA
##  9 2012-10-09       12811
## 10 2012-10-10        9900
## # ... with 51 more rows
```
Next, a histogram would be plotted by using "hist" function.

```r
with(summary_total_step, hist(Total_steps))
```

![plot of chunk histogram](figure/histogram-1.png)

### 3. Mean and median number of steps taken each day.
In this step, we need to calculate the mean and median number of steps taken each day. Similar with step 2, "group_by" and "summarise" function are needed for this task.  
Firstly, we could calculate mean number.  

```r
mean(summary_total_step$Total_steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
Similarly, we could also generate the calculation of median number of steps.  

```r
median(summary_total_step$Total_steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
### 4. Time series plot of the average number of steps taken.  
In this step, we need to plot the time series of the average number of steps taken. Firstly, we need to transfer the type of "date" variable from "factor" to "Date" by using "as.Date" function. 

```r
dataset <- mutate(dataset, date = as.Date(date))
class(dataset$date)
```

```
## [1] "Date"
```

Next, we could summarise the average of steps by date and plot it with time series.    

```r
summary_mean_step <- dataset %>% group_by(date) %>% summarise(Average_steps = mean(steps, na.rm = TRUE))
print(summary_mean_step)
```

```
## # A tibble: 61 x 2
##          date Average_steps
##        <date>         <dbl>
##  1 2012-10-01           NaN
##  2 2012-10-02       0.43750
##  3 2012-10-03      39.41667
##  4 2012-10-04      42.06944
##  5 2012-10-05      46.15972
##  6 2012-10-06      53.54167
##  7 2012-10-07      38.24653
##  8 2012-10-08           NaN
##  9 2012-10-09      44.48264
## 10 2012-10-10      34.37500
## # ... with 51 more rows
```

```r
with(summary_mean_step, plot(date, Average_steps, type = "l"))
```

![plot of chunk summarise mean steps](figure/summarise mean steps-1.png)

We found there are some missing values in our plotting. We can fix it in the steps later on.  

### 5. The 5-minute interval that, on average, contains the maximum number of steps.  

```r
summary_by_interval <- dataset %>% group_by(interval) %>% summarise(Average_steps = mean(steps, na.rm = TRUE))
print(summary_by_interval)
```

```
## # A tibble: 288 x 2
##    interval Average_steps
##       <int>         <dbl>
##  1        0     1.7169811
##  2        5     0.3396226
##  3       10     0.1320755
##  4       15     0.1509434
##  5       20     0.0754717
##  6       25     2.0943396
##  7       30     0.5283019
##  8       35     0.8679245
##  9       40     0.0000000
## 10       45     1.4716981
## # ... with 278 more rows
```

```r
with(summary_by_interval, plot(interval, Average_steps, type = "l", xlab = "Interval(hhmm)"))
```

![plot of chunk summarise by interval](figure/summarise by interval-1.png)

We could find the interval that maximum average steps were taken, by using "which.max" function. 

```r
summary_by_interval[which.max(summary_by_interval$Average_steps), ]
```

```
## # A tibble: 1 x 2
##   interval Average_steps
##      <int>         <dbl>
## 1      835      206.1698
```

From the result above, we could draw that the interval of 8 hours 35 minutes contain the maximum steps taken, averaged by all measured days.  

## Imputing missing values
### 6. Code to describe and show a strategy for imputing missing data.  
Firstly, let us calculate the total number of missing values in our raw data.  

```r
sum(is.na(dataset))
```

```
## [1] 2304
```

In our strategy, the missing value could be filled by the average of 5 minutes interval. We already have the data named "summary_by_interval".  
We could merge the raw data and "summary_by_interval" data by using "inner_join" function.  

```r
dataset_mrg <- inner_join(dataset, summary_by_interval)
```

```
## Joining, by = "interval"
```

```r
head(dataset_mrg)
```

```
##   steps       date interval Average_steps
## 1    NA 2012-10-01        0     1.7169811
## 2    NA 2012-10-01        5     0.3396226
## 3    NA 2012-10-01       10     0.1320755
## 4    NA 2012-10-01       15     0.1509434
## 5    NA 2012-10-01       20     0.0754717
## 6    NA 2012-10-01       25     2.0943396
```

```r
head(dataset_mrg[289:nrow(dataset_mrg), ])
```

```
##     steps       date interval Average_steps
## 289     0 2012-10-02        0     1.7169811
## 290     0 2012-10-02        5     0.3396226
## 291     0 2012-10-02       10     0.1320755
## 292     0 2012-10-02       15     0.1509434
## 293     0 2012-10-02       20     0.0754717
## 294     0 2012-10-02       25     2.0943396
```

Next, we could write a loop: when there is a missing value, replace the "steps" varible value by "Average_steps" variable value.  

```r
for (i in 1:nrow(dataset_mrg)){
    if(is.na(dataset_mrg$steps[i])){
        dataset_mrg$steps[i] <- dataset_mrg$Average_steps[i]
    }
}
head(dataset_mrg)
```

```
##       steps       date interval Average_steps
## 1 1.7169811 2012-10-01        0     1.7169811
## 2 0.3396226 2012-10-01        5     0.3396226
## 3 0.1320755 2012-10-01       10     0.1320755
## 4 0.1509434 2012-10-01       15     0.1509434
## 5 0.0754717 2012-10-01       20     0.0754717
## 6 2.0943396 2012-10-01       25     2.0943396
```

```r
head(dataset_mrg[289:nrow(dataset_mrg), ])
```

```
##     steps       date interval Average_steps
## 289     0 2012-10-02        0     1.7169811
## 290     0 2012-10-02        5     0.3396226
## 291     0 2012-10-02       10     0.1320755
## 292     0 2012-10-02       15     0.1509434
## 293     0 2012-10-02       20     0.0754717
## 294     0 2012-10-02       25     2.0943396
```

Now we can compare the dataset before and after filling NA. We found that value of "steps" variable was successfully replaced by "Average_steps" when it is a missing value (NA). Conversely, the non-missing value was kept the same without any modification.  

At last, update the original dataset with our new "step" varible, by using "mutate" function.  

```r
dataset <- mutate(dataset, steps = dataset_mrg$steps)
head(dataset)
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```

### 7. Histogram of the total number of steps taken each day after missing values are imputed
We could plot a new histogram after filling missing value.  

```r
summary_total_step_new <- dataset %>% group_by(date) %>% summarise(Total_steps = sum(steps))
with(summary_total_step_new, hist(Total_steps, main = "Histogram of Total_steps (after filling missing value)"))
```

![plot of chunk histogram after filling missing value](figure/histogram after filling missing value-1.png)

Also calculate mean and median value again.  
Mean:  

```r
mean(summary_total_step_new$Total_steps)
```

```
## [1] 10766.19
```

Median:  

```r
median(summary_total_step_new$Total_steps)
```

```
## [1] 10766.19
```

From the result, we find that the histogram before and after filling missing value is the same. Mean value is also the same and median value is changed a little bit from 10765 to 10766.19. 

## Are there differences in activity patterns between weekdays and weekends?
### 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends.  
Firstly, we could generate a new variable called "weekdays" by "weekdays" function.  

```r
dataset <- mutate(dataset, weekdays = weekdays(date))
table(dataset$weekdays)
```

```
## 
##    Friday    Monday  Saturday    Sunday  Thursday   Tuesday Wednesday 
##      2592      2592      2304      2304      2592      2592      2592
```

Secondly, we could use conditional mutate by "mutate" function associate with "if_else" function.  

```r
dataset <- mutate(dataset, weekdays = if_else(dataset$weekdays %in% c("Saturday", "Sunday"), "weekends", "weekday"))
table(dataset$weekdays)
```

```
## 
##  weekday weekends 
##    12960     4608
```

At last, we could plot the summary of steps by "lattice" package.  

```r
summary_by_interval_weekdays <- dataset %>% group_by(interval, weekdays) %>% summarise(Average_steps = mean(steps))
print(summary_by_interval_weekdays)
```

```
## # A tibble: 576 x 3
## # Groups:   interval [?]
##    interval weekdays Average_steps
##       <int>    <chr>         <dbl>
##  1        0  weekday   2.251153040
##  2        0 weekends   0.214622642
##  3        5  weekday   0.445283019
##  4        5 weekends   0.042452830
##  5       10  weekday   0.173165618
##  6       10 weekends   0.016509434
##  7       15  weekday   0.197903564
##  8       15 weekends   0.018867925
##  9       20  weekday   0.098951782
## 10       20 weekends   0.009433962
## # ... with 566 more rows
```

```r
library(lattice)
xyplot(Average_steps ~ interval | weekdays, summary_by_interval_weekdays, layout=c(1,2), type = "l", xlab = "Interval(hhmm)", ylab = "Number of steps")
```

![plot of chunk plot time series separated by weekday and weekends](figure/plot time series separated by weekday and weekends-1.png)

From the results, we found that during the weekdays, steps reach the peak at around 8:35, which is in accordance with the total average in step 5. However, the number of steps taken at weekends seem more stable during daily time. 
