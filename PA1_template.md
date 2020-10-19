kint2html()
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
```{r,warning=FALSE,eval=FALSE}
if(!file.exists("Activity monitoring data")) {
        temp <- tempfile()
        download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
        unzip(temp)
        unlink(temp)
}
```
library(ggplot2)
```{r}
activity<-read.csv("activity.csv")
head(activity)
```

### calculating mean total number of steps taken per day
```{r,warning=FALSE}
activity <- read.csv("activity.csv")

activity$date <- as.POSIXct(activity$date, "%Y-%m-%d")
weekday <- weekdays(activity$date)
activity <- cbind(activity,weekday)
summary(activity)
```

```{r,warning=FALSE,fig.height=5,fig.width=5,fig.align='center'}

png("plot1.png")
activity_total_steps <- with(activity, aggregate(steps, by = list(date), FUN = sum, na.rm = TRUE))
names(activity_total_steps) <- c("date", "steps")
hist(activity_total_steps$steps, main = "Total number of steps taken per day", xlab = "Total steps taken per day", col = "pink", ylim = c(0,20), breaks = seq(0,25000, by=2500))
dev.off()
```
```{r,warning=FALSE}
mean(activity_total_steps$steps)

```
```{r,warning=FALSE}
median(activity_total_steps$steps)
```
### ploting the average activity pattern
```{r,warning=FALSE,fig.width=5,fig.height=5,fig.align='center'}
png("plot2.png")
average_daily_activity <- aggregate(activity$steps, by=list(activity$interval), FUN=mean, na.rm=TRUE)
names(average_daily_activity) <- c("interval", "mean")
plot(average_daily_activity$interval, average_daily_activity$mean, type = "l", col="blue", lwd = 2, xlab="Interval", ylab="Average number of steps", main="Average number of steps per intervals")
dev.off()
```
### finding the maxiumnumber of steps
```{r,warning=FALSE}
average_daily_activity[which.max(average_daily_activity$mean), ]$interval
```
### imputing missing values
```{r,warning=FALSE}
sum(is.na(activity$steps))
```
```{r,warning=FALSE}
imputed_steps <- average_daily_activity$mean[match(activity$interval, average_daily_activity$interval)]
```
```{r,warning=FALSE}
activity_imputed <- transform(activity, steps = ifelse(is.na(activity$steps), yes = imputed_steps, no = activity$steps))
total_steps_imputed <- aggregate(steps ~ date, activity_imputed, sum)
names(total_steps_imputed) <- c("date", "daily_steps")
```
```{r,warning=FALSE,fig.align='center',fig.width=5,fig.height=5}
png("plot3.png")
hist(total_steps_imputed$daily_steps, col = "blue", xlab = "Total steps per day", ylim = c(0,30), main = "Total number of steps taken each day", breaks = seq(0,25000,by=2500))
dev.off()
```
```{r}
mean(total_steps_imputed$daily_steps)
```
```{r}
median(total_steps_imputed$daily_steps)
```
### checking differences in activity patterns between weekdays and weekends
```{r,warning=FALSE}
activity$date <- as.Date(strptime(activity$date, format="%Y-%m-%d"))
activity$datetype <- sapply(activity$date, function(x) {
        if (weekdays(x) == "Sábado" | weekdays(x) =="Domingo") 
                {y <- "Weekend"} else 
                {y <- "Weekday"}
                y
        })
```
```{r,warning=FALSE}
png("plot4.png")
library(ggplot2)
activity_by_date <- aggregate(steps~interval + datetype, activity, mean, na.rm = TRUE)
plot<- ggplot(activity_by_date, aes(x = interval , y = steps, color = datetype)) +
       geom_line() +
       labs(title = "Average daily steps by type of date", x = "Interval", y = "Average number of steps") +
       facet_wrap(~datetype, ncol = 1, nrow=2)
print(plot)
dev.off()
```

