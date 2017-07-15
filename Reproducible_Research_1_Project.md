Chandrashekar Baskaran July 2017

This is the first document in the Reproducible Research course in
Coursera's Introduction to Data Science track

### Aim:

The purpose of the project was to execute: - Loading and Preprocessing
data - Impute missing values - Answer research questions

### Data:

The data for this assignment can be downloaded from the course web site:

-   Dataset: [Activity monitoring
    data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
    \[52K\]

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)
-   date: The date on which the measurement was taken in YYYY-MM-DD
    format
-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

### Loading and Preprocessing the Data:

Download,unzip the data provided and load it into data frame data.

    require(downloader)

    ## Loading required package: downloader

    dataset_url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(dataset_url,dest="data.zip", mode="wb")
    unzip("data.zip", exdir="./")
    data<-read.csv("activity.csv")

#### What is mean total number of steps taken per day?

Sum steps per day, create histogram and compute mean and median

    steps_by_day <- aggregate(steps ~ date,data,sum)
    hist(steps_by_day$steps, main=paste("Total steps each day"), col= "blue",xlab = "Number of Steps")

![](Reproducible_Research_1_Project_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    rmean <- mean(steps_by_day$steps)
    rmedian <- median(steps_by_day$steps)

The mean is 1.076618910^{4} and the median is 10765

##### What is the average daily activity pattern?

Plot a time series with average steps taken and compute which 5 minute
interval has maximum steps

    steps_by_interval<-aggregate(steps~interval, data, mean)
    plot(steps_by_interval$interval, steps_by_interval$steps, type= "l", xlab="Interval", ylab = "Number of steps")

![](Reproducible_Research_1_Project_files/figure-markdown_strict/unnamed-chunk-3-1.png)

    max_interval<-steps_by_interval[which.max(steps_by_interval$steps),1]

The 5-minute interval on an average across all days containing maximum
number of steps is 835

#### Impute missing values and compare imputed and non imputed data

Missing values need to be imputed. A simple approach was used here where
we substitute the missing values with the average for each interval.

    incomplete<-sum(!complete.cases(data))
    imputed_data <- transform(data, steps = ifelse(is.na(data$steps),steps_by_interval$steps[match(data$interval,steps_by_interval$interval)],data$steps))

Zeroes were imputed for data 10/01/2012 as it was the first day

    imputed_data[as.character(imputed_data$date)=="2012-10-01",1]<-0

Recount total steps by day to create histogram to show the difference

    steps_by_day_i <- aggregate(steps~date,imputed_data,sum)
    hist(steps_by_day_i$steps, main=paste("Total steps each day"),col="blue",xlab="Number of Steps")
    hist(steps_by_day$steps, main=paste("Total steps each day"), col= "red",xlab = "Number of Steps",add = T)
    legend("topright", c("Imputed", "Non-imputed"), col=c("blue", "red"), lwd=10)

![](Reproducible_Research_1_Project_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Calculate the new mean and median

    rmean.i <- mean(steps_by_day_i$steps)
    rmedian.i <- median(steps_by_day_i$steps)

Calculate difference between imputed and non-imputed data

    mean_diff <- rmean.i - rmean
    med_diff <- rmedian.i - rmedian

Calculate the total difference

    total_diff <- sum(steps_by_day_i$steps) - sum(steps_by_day$steps)

-   The imputed data mean is 1.058969410^{4}
-   The imputed data median is 1.076618910^{4}
-   The difference between non-imputed and imputed mean is -176.4948964
-   The difference between non-imputed and imputed median is 1.1886792
-   The difference between total number of steps between imputed and
    non-imputed data is 7.536332110^{4}. There were 7.536332110^{4} more
    steps in imputed data

#### Are there differences in activity patterns between weekdays and weekends?

Create a plot to contrast the number of steps between weekdays and
weekends. There is higher peak in weekdays than weekends

    weekdays <- c("Monday","Tuesday","Wednesday","Thursday","Friday", "Saturday")
    imputed_data$dow = as.factor(ifelse(is.element(weekdays(as.Date(imputed_data$date)),weekdays), "Weekday", "Weekend"))
    steps_by_interval_i <- aggregate(steps ~ interval + dow, imputed_data, mean)
    library(lattice)
    xyplot(steps_by_interval_i$steps ~ steps_by_interval_i$interval|steps_by_interval_i$dow, main="Average Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")

![](Reproducible_Research_1_Project_files/figure-markdown_strict/unnamed-chunk-10-1.png)
