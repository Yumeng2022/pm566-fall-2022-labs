Lab 05
================
Yumeng Gao
2022-09-21

``` r
library(webshot)
webshot::install_phantomjs()
```

    ## It seems that the version of `phantomjs` installed is greater than or equal to the requested version.To install the requested version or downgrade to another version, use `force = TRUE`.

## 1. Read in the data

First download and then read in with data.table:fread()

``` r
library(R.utils)
```

    ## Loading required package: R.oo

    ## Loading required package: R.methodsS3

    ## R.methodsS3 v1.8.2 (2022-06-13 22:00:14 UTC) successfully loaded. See ?R.methodsS3 for help.

    ## R.oo v1.25.0 (2022-06-12 02:20:02 UTC) successfully loaded. See ?R.oo for help.

    ## 
    ## Attaching package: 'R.oo'

    ## The following object is masked from 'package:R.methodsS3':
    ## 
    ##     throw

    ## The following objects are masked from 'package:methods':
    ## 
    ##     getClasses, getMethods

    ## The following objects are masked from 'package:base':
    ## 
    ##     attach, detach, load, save

    ## R.utils v2.12.0 (2022-06-28 03:20:05 UTC) successfully loaded. See ?R.utils for help.

    ## 
    ## Attaching package: 'R.utils'

    ## The following object is masked from 'package:utils':
    ## 
    ##     timestamp

    ## The following objects are masked from 'package:base':
    ## 
    ##     cat, commandArgs, getOption, isOpen, nullfile, parse, warnings

``` r
library(data.table)
library(tidyverse)
```

    ## ── Attaching packages
    ## ───────────────────────────────────────
    ## tidyverse 1.3.2 ──

    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.8     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::between()   masks data.table::between()
    ## ✖ tidyr::extract()   masks R.utils::extract()
    ## ✖ dplyr::filter()    masks stats::filter()
    ## ✖ dplyr::first()     masks data.table::first()
    ## ✖ dplyr::lag()       masks stats::lag()
    ## ✖ dplyr::last()      masks data.table::last()
    ## ✖ purrr::transpose() masks data.table::transpose()

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'
    ## 
    ## The following objects are masked from 'package:data.table':
    ## 
    ##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
    ##     yday, year
    ## 
    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(dtplyr)
```

``` r
if (!file.exists("../lab03/met_all.gz"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz",
    destfile = "met_all.gz",
    method   = "libcurl",
    timeout  = 60
    )
met <- data.table::fread("../lab03/met_all.gz")
```

``` r
# Remove temperatures less than -17C
# Make sure there are no missing data in the key variables coded as 9999, 999, etc
met <- met[temp>-17] [elev == 9999.0, elev := NA]
```

## 2. Load the met data

``` r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```

## 3.Merge met data with stations

``` r
met= merge(
  x = met,
  y = stations,
  by.x = "USAFID",
  by.y = "USAF",
  all.x = TRUE,
  all.y = FALSE
)
```

## Question 1: Representative station for the US

Computer mean temperature, wind speed and atmospheric pressure for each
weather station, and pick the weather stations with the average value
closet to the median for the US.

``` r
station_averages =
  met[, .(
    temp= mean(temp, na.rm=T),
    wind.sp= mean(wind.sp, na.rm=T),
    atm.press= mean(atm.press, na.rm=T)
  ), by= USAFID]
```

The above computes the mean by weather station. Now let’s compute the
median value for each variable.

``` r
statmedians= station_averages[, .(
  temp50 = median(temp, na.rm=T),
  windsp50 = median(wind.sp, na.rm=T),
  atmpress50 = median(atm.press, na.rm=T)
)]
statmedians
```

    ##      temp50 windsp50 atmpress50
    ## 1: 23.68406 2.463685   1014.691

``` r
summary(station_averages[ , temp])
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   8.045  20.902  23.684  23.852  26.809  37.625

A helpful function: ‘which.min()’.

``` r
station_averages[ , 
                  temp_dist50 := abs(temp- statmedians$temp50)][order(temp_dist50)]
```

    ##       USAFID      temp   wind.sp atm.press  temp_dist50
    ##    1: 720458 23.681730  1.209682       NaN  0.002328907
    ##    2: 725515 23.686388  2.709164       NaN  0.002328907
    ##    3: 725835 23.678347  2.652381       NaN  0.005712423
    ##    4: 724509 23.675100  4.066833  1013.863  0.008959632
    ##    5: 720538 23.665932  1.907897       NaN  0.018127186
    ##   ---                                                  
    ## 1584: 722788 36.852459  3.393852       NaN 13.168399783
    ## 1585: 722787 37.258907  2.847381       NaN 13.574848130
    ## 1586: 723805 37.625391  3.532935  1005.207 13.941331392
    ## 1587: 726130  9.189602 12.239908       NaN 14.494456787
    ## 1588: 720385  8.044959  7.298963       NaN 15.639100105

``` r
station_averages[ which.min(temp_dist50)]
```

    ##    USAFID     temp  wind.sp atm.press temp_dist50
    ## 1: 720458 23.68173 1.209682       NaN 0.002328907

It matches the result above.

## Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the
most representative, the median, station per state. This time, instead
of looking at one variable at a time, look at the euclidean distance. If
multiple stations show in the median, select the one located at the
lowest latitude.

``` r
station_averages =
  met[, .(
    temp= mean(temp, na.rm=T),
    wind.sp= mean(wind.sp, na.rm=T),
    atm.press= mean(atm.press, na.rm=T)
  ), by=  .(USAFID, STATE)]
```

``` r
statemeds= station_averages[, .(
  temp50 = median(temp, na.rm=T),
  windsp50 = median(wind.sp, na.rm=T),
  atmpress50 = median(atm.press, na.rm=T)
),by= STATE]
statemeds
```

    ##     STATE   temp50 windsp50 atmpress50
    ##  1:    CA 22.66268 2.561738   1012.557
    ##  2:    TX 29.75188 3.413810   1012.460
    ##  3:    MI 20.51970 2.273423   1014.927
    ##  4:    SC 25.80545 1.696119   1015.281
    ##  5:    IL 22.43194 2.237652   1014.760
    ##  6:    MO 23.95109 2.453547   1014.522
    ##  7:    AR 26.24296 1.938625   1014.591
    ##  8:    OR 17.98061 2.011436   1015.269
    ##  9:    WA 19.24684 1.268571         NA
    ## 10:    GA 26.70404 1.497527   1015.208
    ## 11:    MN 19.63017 2.616482   1015.042
    ## 12:    AL 26.33664 1.662132   1014.959
    ## 13:    IN 22.25059 2.344333   1015.063
    ## 14:    NC 24.72953 1.627306   1015.420
    ## 15:    VA 24.37799 1.654183   1015.107
    ## 16:    IA 21.33461 2.680875   1014.964
    ## 17:    PA 21.69177 1.784167   1015.435
    ## 18:    NE 21.87354 3.192539   1014.332
    ## 19:    ID 20.56798 2.568944   1012.855
    ## 20:    WI 18.85524 2.053283   1014.893
    ## 21:    WV 21.94446 1.632107   1015.762
    ## 22:    MD 24.89883 1.883499   1014.824
    ## 23:    AZ 30.32372 3.074359   1010.144
    ## 24:    OK 27.14427 3.852697   1012.567
    ## 25:    WY 19.80699 3.873986   1013.157
    ## 26:    LA 27.87430 1.712535   1014.593
    ## 27:    KY 23.88844 1.895486   1015.245
    ## 28:    FL 27.57325 2.705069   1015.335
    ## 29:    CO 21.52650 3.098777   1013.334
    ## 30:    OH 22.02062 2.554138   1015.351
    ## 31:    NJ 23.47238 2.148058   1014.825
    ## 32:    NM 24.94447 3.776083   1012.525
    ## 33:    KS 24.21220 3.676997   1013.389
    ## 34:    ND 18.52849 3.956459         NA
    ## 35:    VT 18.61379 1.408247   1014.792
    ## 36:    MS 26.69258 1.637030   1014.836
    ## 37:    CT 22.36880 2.101294   1014.810
    ## 38:    NV 24.56293 3.035050   1012.204
    ## 39:    UT 24.35182 3.110795   1011.972
    ## 40:    SD 20.35662 3.665638   1014.398
    ## 41:    TN 24.88657 1.576035   1015.144
    ## 42:    NY 20.40674 2.304075   1014.887
    ## 43:    RI 22.53551 2.583469   1014.728
    ## 44:    MA 21.30662 2.710944   1014.751
    ## 45:    DE 24.56026 2.753082   1015.046
    ## 46:    NH 19.55054 1.563826   1014.689
    ## 47:    ME 18.79016 2.237210   1014.399
    ## 48:    MT 19.15492 4.151737   1014.186
    ##     STATE   temp50 windsp50 atmpress50

``` r
station_averages= 
  merge(
  x= station_averages,
  y= statemeds,
  by.x= "STATE",
  by.y= "STATE",
  all.x= T,
  ALL.y= F
)
```

``` r
station_averages[ , temp_dist_state50 := temp- temp50]
station_averages[ , windsp_dist_state50 := wind.sp- windsp50]             
station_averages[ , atmpress_dist_state50 := atm.press- atmpress50]      
```

``` r
station_averages[ , eucdist := temp_dist_state50^2 +
                    windsp_dist_state50^2]
station_averages[ , .(
  rep50= min(eucdist))
  , by=STATE]
```

    ##     STATE        rep50
    ##  1:    AL          NaN
    ##  2:    AR 0.1736376706
    ##  3:    AZ 0.0544857836
    ##  4:    CA 0.0651314631
    ##  5:    CO          NaN
    ##  6:    CT 0.0433144478
    ##  7:    DE 0.0000000000
    ##  8:    FL 0.0009745369
    ##  9:    GA 0.0021721657
    ## 10:    IA 0.0020160519
    ## 11:    ID 0.0777385965
    ## 12:    IL 0.0077712543
    ## 13:    IN 0.0159151165
    ## 14:    KS 0.0557784421
    ## 15:    KY 0.2125405164
    ## 16:    LA          NaN
    ## 17:    MA 0.0144022476
    ## 18:    MD 0.0000000000
    ## 19:    ME 0.0943820541
    ## 20:    MI 0.0132319539
    ## 21:    MN          NaN
    ## 22:    MO 0.0416560272
    ## 23:    MS 0.0351865214
    ## 24:    MT 0.1879315944
    ## 25:    NC 0.0091213936
    ## 26:    ND 0.0348669121
    ## 27:    NE 0.0090257850
    ## 28:    NH 0.1065472295
    ## 29:    NJ 0.0000000000
    ## 30:    NM 0.0427670273
    ## 31:    NV 0.2380402433
    ## 32:    NY 0.0093580816
    ## 33:    OH 0.0634001463
    ## 34:    OK 0.0069584784
    ## 35:    OR 0.7028808653
    ## 36:    PA 0.0339881655
    ## 37:    RI 0.0668411712
    ## 38:    SC 0.0327455867
    ## 39:    SD 0.1710841308
    ## 40:    TN 0.0375202070
    ## 41:    TX          NaN
    ## 42:    UT 0.0183564932
    ## 43:    VA          NaN
    ## 44:    VT          NaN
    ## 45:    WA 0.0000000000
    ## 46:    WI 0.0081925142
    ## 47:    WV 0.0003044727
    ## 48:    WY          NaN
    ##     STATE        rep50

``` r
repstation= station_averages[ , .(
  eucdist= min(eucdist))
  , by=STATE]
```

``` r
test= merge(
  x= station_averages,
  y= repstation,
  by.x= c("eucdist","STATE"),
  by.y= c("eucdist","STATE"),
  all.x= F,
  all.y= T
)
```
