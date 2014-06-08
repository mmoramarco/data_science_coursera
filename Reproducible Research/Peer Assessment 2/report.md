Severe Weather Events in the United States - 1950 through 2011
========================================================

### Synopsis

We analyze severe weather events in the United States to determine what event types lead to the greatest affect population health and economic consequences. As regards this analysis, population health is used to describe both fatalities and injuries. Similarly, economic consequences is used to describe both property damage and crop damage. Based on this analysis, tornadoes have the greatest impact on population health while floods generate the greatest economic consequences.

### Data Processing

All necessary packages for this analysis are loaded first.


```r
library(R.utils)
```

```
## Loading required package: R.oo
## Loading required package: R.methodsS3
## R.methodsS3 v1.6.1 (2014-01-04) successfully loaded. See ?R.methodsS3 for help.
## R.oo v1.18.0 (2014-02-22) successfully loaded. See ?R.oo for help.
## 
## Attaching package: 'R.oo'
## 
## The following objects are masked from 'package:methods':
## 
##     getClasses, getMethods
## 
## The following objects are masked from 'package:base':
## 
##     attach, detach, gc, load, save
## 
## R.utils v1.32.4 (2014-05-14) successfully loaded. See ?R.utils for help.
## 
## Attaching package: 'R.utils'
## 
## The following object is masked from 'package:utils':
## 
##     timestamp
## 
## The following objects are masked from 'package:base':
## 
##     cat, commandArgs, getOption, inherits, isOpen, parse, warnings
```

```r
library(ggplot2)
library(plyr)
```


Next the data set is downloaded and extracted. This set is then loaded into the 'data' data frame.


```r
data_url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
download.file(data_url, destfile = "data.csv.bz2", method = "wget")
bunzip2("data.csv.bz2")
data <- read.csv("data.csv")
```


Next, we consider our two categories of analysis: human loss and economic loss. With regards to human loss, we create a subset using the appropriate columns (event type, fatalities, injuries). We then create a new column, casualities, that sums the fatalities and injuries columns. This column better reflects the true human loss created by the event. This information is then summarized by event type using the plyr package.


```r
health <- data[, c("EVTYPE", "FATALITIES", "INJURIES")]
health$CASUALTIES <- health$FATALITIES + health$INJURIES
healthsummary <- ddply(health, .(EVTYPE), summarize, TOTAL = sum(CASUALTIES))
```


Similar processes are conducted for the appropriate economic damage related columns (property damage and crop damage). One caveat with this data is the use of exponents for damage calculations (i.e. the PROPDMGEXP column contains the value 'k' when the PROPDMG value is in 1,000s). Additional columns are constructed to input the multiplier needed for the most common exponents (billions, millions, thousands, and hundreds). This information is then computed into 'TOTAL' column. The package, plyr, is again used to summarize the data across event types.


```r
damage <- data[, c("EVTYPE", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")]
damage$PROPMULT <- rep(0, nrow(damage))
damage$PROPMULT[damage$PROPDMGEXP == "B"] <- 1e+09
damage$PROPMULT[damage$PROPDMGEXP == "M"] <- 1e+06
damage$PROPMULT[damage$PROPDMGEXP == "K"] <- 1000
damage$PROPMULT[damage$PROPDMGEXP == "H"] <- 100
damage$CROPMULT <- rep(0, nrow(damage))
damage$CROPMULT[damage$CROPDMGEXP == "B"] <- 1e+09
damage$CROPMULT[damage$CROPDMGEXP == "M"] <- 1e+06
damage$CROPMULT[damage$CROPDMGEXP == "K"] <- 1000
damage$CROPMULT[damage$CROPDMGEXP == "H"] <- 10
damage$TOTAL <- damage$PROPDMG * damage$PROPMULT + damage$CROPDMG * damage$CROPMULT
damagesummary <- ddply(damage, .(EVTYPE), summarize, TOTAL = sum(TOTAL))
```


### Results

The below plot details the total number of casualties (fatalities and injuries) by event type. This indicates that tornados lead to the greatest human loss of any individual event type.


```r
ggplot(healthsummary[order(healthsummary$TOTAL, decreasing = TRUE)[1:10], ], 
    aes(x = reorder(EVTYPE, TOTAL), y = TOTAL)) + geom_bar(stat = "identity") + 
    coord_flip() + xlab("Event") + ylab("Casualties") + ggtitle("Casualties by Event")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 


The below plot details the total economic loss (property and crop damage) by event type. This indicates that floods lead to the greatest economic loss of any individual event type.


```r
ggplot(damagesummary[order(damagesummary$TOTAL, decreasing = TRUE)[1:10], ], 
    aes(x = reorder(EVTYPE, TOTAL), y = TOTAL)) + geom_bar(stat = "identity") + 
    coord_flip() + xlab("Event") + ylab("Economic Losses") + ggtitle("Economic Losses by Event")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 
