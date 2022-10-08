Homework2
================
Chen Chen 6381370662
2022-10-07

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(tidyverse)
```

    ## ── Attaching packages
    ## ───────────────────────────────────────
    ## tidyverse 1.3.2 ──

    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.8     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.1
    ## ✔ readr   2.1.2     ✔ forcats 0.5.2
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ lubridate::as.difftime() masks base::as.difftime()
    ## ✖ lubridate::date()        masks base::date()
    ## ✖ dplyr::filter()          masks stats::filter()
    ## ✖ lubridate::intersect()   masks base::intersect()
    ## ✖ dplyr::lag()             masks stats::lag()
    ## ✖ lubridate::setdiff()     masks base::setdiff()
    ## ✖ lubridate::union()       masks base::union()

``` r
library(data.table)
```

    ## 
    ## Attaching package: 'data.table'
    ## 
    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last
    ## 
    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose
    ## 
    ## The following objects are masked from 'package:lubridate':
    ## 
    ##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
    ##     yday, year

``` r
library(dtplyr)
library(dplyr)
library(ggplot2)
```

Data Wrangling

``` r
if (!file.exists("/Users/apple/Desktop/pm566/PM566-Homework/homework 2/chs_individual.csv")){
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/01_chs/chs_individual.csv", "chs_individual.csv", method="libcurl", timeout = 60)
}
individual <- data.table::fread("/Users/apple/Desktop/pm566/PM566-Homework/homework 2/chs_individual.csv")

if (!file.exists("/Users/apple/Desktop/pm566/PM566-Homework/homework 2/chs_regional.csv")){
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/01_chs/chs_regional.csv", "chs_regional.csv", method="libcurl", timeout = 60)
}
individual <- data.table::fread("/Users/apple/Desktop/pm566/PM566-Homework/homework 2/chs_individual.csv")
regional <- data.table::fread("/Users/apple/Desktop/pm566/PM566-Homework/homework 2/chs_regional.csv")
```

``` r
data<-
  merge(
    x= individual,
    y= regional,
    by.x = "townname",
    by.y = "townname",
    all.x = TRUE,
    all.y = FALSE
  )
```

1.  After merging the data, make sure you don’t have any duplicates by
    counting the number of rows. Make sure it matches.

``` r
nrow(data)
```

    ## [1] 1200

``` r
nrow(individual)
```

    ## [1] 1200

``` r
#We find that "data" and "individual" have same number of rows. There is no duplicates.
```
