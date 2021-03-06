library(ggplot2)
library(dplyr)
library(chron)
a <- read.csv("activity.csv", header = TRUE)

head(a)
aggsteps<- aggregate(steps ~ date, a, FUN=sum)

#Aggregated Data (all steps added for a particular date)
head(aggsteps)
hist(aggsteps$steps, 
     col="red", 
     xlab = "Frequency", 
     ylab = "Steps",
     main = "Total Number Of Steps Taken Each day")
amean <- mean(aggsteps$steps)
amedian <- median(aggsteps$steps)

#Mean total number of steps taken per day
amean
amedian

agginterval <- aggregate(steps ~ interval, a, FUN=sum)

#Plotting line graph using plot() from Base Plotting for Total Steps vs 5-Minute Interval
plot(agginterval$interval, agginterval$steps, 
     type = "l", lwd = 2,
     xlab = "Interval", 
     ylab = "Total Steps",
     main = "Total Steps vs. 5-Minute Interval")
filter(agginterval, steps==max(steps))
table(is.na(a))
meaninterval<- aggregate(steps ~ interval, a, FUN=mean)

#Merging the mean of total steps for a date with the original data set
anew <- merge(x=a, y=meaninterval, by="interval")

#Replacing the NA values with the mean for that 5-minute interval
anew$steps <- ifelse(is.na(anew$steps.x), anew$steps.y, anew$steps.x)

#Merged dataset which will be subsetted in the next step by removing not required columns
head(anew)anew <- select(anew, steps, date, interval)

#New dataset with NA imputed by mean for that 5-minute interval
head(anew)
aggsteps_new<- aggregate(steps ~ date, anew, FUN=sum)

#Plotting
#Setting up the pannel for one row and two columns
par(mfrow=c(1,2))

#Histogram after imputing NA values with mean of 5-min interval
hist(aggsteps_new$steps, 
     col="green",
     xlab = "Steps", 
     ylab = "Frequency",
     ylim = c(0,35),
     main = "Total Number Of Steps Taken Each day \n(After imputing NA values with \n mean of 5-min interval)",
     cex.main = 0.7)

#Histogram with the orginal dataset
hist(aggsteps$steps, 
     col="red", 
     xlab = "Steps", 
     ylab = "Frequency",
     ylim = c(0,35),
     main = "Total Number Of Steps Taken Each day \n(Orginal Dataset)",
     cex.main = 0.7)
par(mfrow=c(1,1)) #Resetting the panel

amean_new <- mean(aggsteps_new$steps)
amedian_new <- median(aggsteps_new$steps)

#Comparing Means
paste("New Mean      :", round(amean_new,2), "," ,  
      " Original Mean :", round(amean,2),"," , 
      " Difference :",round(amean_new,2) -  round(amean,2))

paste("New Median    :", amedian_new, ",", 
      " Original Median :", amedian,"," , 
      " Difference :",round(amedian_new-amedian,2))
table(is.weekend(anew$date))

anew$dayofweek <- ifelse(is.weekend(anew$date), "weekend", "weekday")

#Number of Weekdays and Weekends
table(anew$dayofweek)

head(anew)
meaninterval_new<- aggregate(steps ~ interval + dayofweek, anew, FUN=mean)

#Aggregated Data
head(meaninterval_new)
ggplot(meaninterval_new, aes(x=interval, y=steps)) + 
  geom_line(color="blue", size=1) + 
  facet_wrap(~dayofweek, nrow=2) +
  labs(x="\nInterval", y="\nNumber of steps")
