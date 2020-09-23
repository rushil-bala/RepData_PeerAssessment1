Loading and preprocessing the data
----------------------------------

-   Appropriately set the working directory.
-   Download the raw data.

<!-- -->

    library(ggplot2)
    library(dplyr)
    library(Hmisc)

    if(!file.exists("activity.csv")){
      unzip("activity.zip")
    }
    rawdata <- read.csv("activity.csv", header = TRUE)

What is mean total number of steps taken per day?
-------------------------------------------------

    total_steps <- tapply(rawdata$steps, rawdata$date, FUN = sum, na.rm = TRUE)
    qplot(total_steps, binwidth = 500, fill = "red", xlab = "Total number of steps moved every day", ylab = "Frequency") +
      theme(legend.position = "none")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

-   Mean: 9354.2295082
-   Median: 10395

What is the average daily activity pattern?
-------------------------------------------

    average_steps <- aggregate(x = list(avg_steps = rawdata$steps), by = list(fm_interval = rawdata$interval), FUN = mean, na.rm = TRUE)
    ggplot(data = average_steps, aes(x = fm_interval, y = avg_steps, col = "red")) +
      geom_line() +
      labs(x = "5-minute intervals", y = "Average number of steps moved every day") +
      theme(legend.position = "none")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    ans <- average_steps[which.max(average_steps$avg_steps), 1]

-   Answer: 835

Imputing missing values
-----------------------

Note that there are a number of days/intervals where there are missing
values (coded as `NA`. The presence of missing days may introduce bias
into some calculations or summaries of the data.

    num_NA <- length(which(is.na(rawdata$steps)))

-   Number of NA: 2304

<!-- -->

    imputeddata <- rawdata
    imputeddata$steps <- impute(rawdata$steps, FUN = mean)
    imputeddatadate <- tapply(imputeddata$steps, imputeddata$date, sum)
    qplot(imputeddatadate, binwidth = 500, fill = "red", xlab = "Total number of steps moved every day", ylab = "Frequency") +
      theme(legend.position = "none")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

-   Mean: 9354.2295082
-   Median: 1.039510^{4}

Note that `NA` values were initially ignored (`na.rm = TRUE`). The mean
and median increased after these were replaced by the initial means.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

    imputeddata$datecat <- ifelse(weekdays(as.Date(imputeddata$date)) == "Saturday" | weekdays(as.Date(imputeddata$date)) == "Sunday", "Weekend", "Weekday")
    average_imputeddata <- aggregate(data = imputeddata, steps ~ interval + datecat, FUN = mean)
    ggplot(average_imputeddata, aes(x = interval, y = steps, col = "red")) +
      geom_line() +
      facet_grid(datecat ~ .) +
      labs(x = "5-minute intervals", y = "Average number of steps moved") +
      theme(legend.position = "none")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)
