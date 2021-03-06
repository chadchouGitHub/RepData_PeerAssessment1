# Assignment1
Chia-Ching Chou  
August 13, 2014  

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:
# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Read data into R, and convert date col into Date format.
Remove missing data in "step" col.


```r
StepData <-  read.csv("activity.csv", header = TRUE, na.strings="NA")
StepData$date <- as.Date(StepData$date, format="%Y-%m-%d")
newStepData <- StepData[!is.na(StepData$step),]
```


## What is mean total number of steps taken per day?


```r
splitDataByDate <- split(newStepData$steps, newStepData$date)
allDays <- as.Date(sort(unique(newStepData$date)), format = "%Y-%m-%d")
sumStepByDate <- as.data.frame(sapply(splitDataByDate, sum))
names(sumStepByDate) <- c("Steps")

png(filename='./figures/EachDaySteps.png', width=480, height=480, units="px")
hist(sumStepByDate$Steps, main="Total steps by day", xlab="Number of steps in each day", ylab="Number of days", col="red", breaks = nrow(sumStepByDate))
dev.off()
```

```
## pdf 
##   2
```
The mean for total number of steps taken per day is

```r
mean(sumStepByDate$Steps)
```

```
## [1] 10766
```

The median for total steps in each day is

```r
median(sumStepByDate$Steps)
```

```
## [1] 10765
```


The graphic of total steps in eaach day in "EachDaySteps.png"

## What is the average daily activity pattern?


```r
IntervalsData <- sort(unique(newStepData$interval))
DataByInterval <- split(newStepData$steps, newStepData$interval)
meanStepByInterval <- as.data.frame(sapply(DataByInterval, mean))
names(meanStepByInterval) <- c("Steps")


png(filename='./figures/5interval.png', width=480, height=480, units="px")
plot(IntervalsData, meanStepByInterval$Steps, type="l", main="5 Interval step mean", xlab="5 interval", ylab="Mean of steps")

dev.off()
```

```
## pdf 
##   2
```

This max steps happened in

```r
which.max(meanStepByInterval$Steps)
```

```
## [1] 104
```
The mean max steps number is

```r
meanStepByInterval[which.max(meanStepByInterval$Steps),]
```

```
## [1] 206.2
```


## Imputing missing values


```r
rNAStepData <- StepData
meanByDate <- sapply(splitDataByDate, mean)
for(i in 1:nrow(rNAStepData)){
      if  (is.na(rNAStepData[i,1])){
              datei <- rNAStepData[i,2]
              rNAStepData[i,1] <- meanByDate[[as.factor(datei)]]
      }
}
```
Use new Dataset to make new graphic, and caculate new mean.

```r
splitnewDataByDate <- split(rNAStepData$steps, rNAStepData$date)
allnewDays <- as.Date(sort(unique(rNAStepData$date)), format = "%Y-%m-%d")
sumOfrNAStepByDate <- as.data.frame(sapply(splitnewDataByDate, sum))
names(sumOfrNAStepByDate) <- c("Steps")

png(filename='./figures/reNAEachDaySteps.png', width=480, height=480, units="px")
hist(sumOfrNAStepByDate$Steps, main="Total steps by day", xlab="Number of steps in each day", ylab="Number of days", col="red", breaks = nrow(sumOfrNAStepByDate))
dev.off()
```

```
## pdf 
##   2
```
The new mean for total number of steps taken per day is

```r
mean(sumOfrNAStepByDate$Steps)
```

```
## [1] 9371
```

The new median for total steps in each day is

```r
median(sumOfrNAStepByDate$Steps)
```

```
## [1] 10395
```


## Are there differences in activity patterns between weekdays and weekends?

Adding new variable into dataset to indicate the weekday and weekend.
The weekdays function add the new variable from Sunday to Monday, and for loop convert to weekday and weekend


```r
rNAStepData$dateWeek <- weekdays(rNAStepData$date)
for(i in 1: nrow(rNAStepData)){
        if(rNAStepData[i,4]=="Sunday" | rNAStepData[i,4]=="Saturday"){
                rNAStepData[i,4] <- "Weekend"    
        }else{
                rNAStepData[i,4] <- "Weekday"    
        }
}
```

Split data set to weekday and weekend.


```r
weekDaySplit <- split(rNAStepData, rNAStepData$dateWeek)
weekendData <- weekDaySplit[["Weekend"]]
weekdayData <- weekDaySplit[["Weekday"]]
```

Process weekendData and weekdayData by "interval".

weekendData

```r
weInterval <- unique(sort(weekendData$interval))
splitweekendDataIntervals <- split(weekendData$steps, weInterval)
meanSplitweekendDataIntervals <- as.data.frame(sapply(splitweekendDataIntervals, mean))
names(meanSplitweekendDataIntervals) <- "meanSteps"
wEMatrixValues <- as.data.frame(cbind(meanSplitweekendDataIntervals$meanSteps, weInterval, "Weekend"))
names(wEMatrixValues) <- c("meanSteps", "interval", "weekdate")
```
weekdayData

```r
wdInterval <- unique(sort(weekdayData$interval))
splitweekdayDataIntervals <- split(weekdayData$steps, wdInterval)
meanSplitweekdayDataIntervals <- as.data.frame(sapply(splitweekdayDataIntervals, mean))
names(meanSplitweekdayDataIntervals) <- "meanSteps"
wDMatrixValues <- as.data.frame(cbind(meanSplitweekdayDataIntervals$meanSteps, wdInterval, "Weekday"))
names(wDMatrixValues) <- c("meanSteps", "interval", "weekdate")
```

Make new set of weekday weekend data with variable "meanSteps", "interval", and "weekdate".


```r
meanStepMatrix <- as.data.frame(rbind(wDMatrixValues,wEMatrixValues))
meanStepMatrix$meanSteps <- as.numeric(as.character(meanStepMatrix$meanSteps))
meanStepMatrix$interval <- as.numeric(as.character(meanStepMatrix$interval))
```
Make figure that show in assignment instruction.

```r
library("lattice")
fig<- xyplot(meanStepMatrix$meanSteps ~ meanStepMatrix$interval | meanStepMatrix$weekdate, meanStepMatrix, layout=c(1,2), type="l", xlab="Interval", ylab="Number of steps")

print(fig)
```

![plot of chunk unnamed-chunk-17](./PA1_template_files/figure-html/unnamed-chunk-17.png) 

The weekend and weekday are different.

