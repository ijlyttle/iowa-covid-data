Get NYT data
================

The purpose of this document is to get the NYT data - filtered for Iowa.

``` r
library("readr")
library("dplyr")
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library("fs")
```

``` r
dir_create("data")

dir_target <- path("data", "nyt")
dir_create(dir_target)
```

``` r
iowa_county_data <- 
  "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv" %>%
  read_csv(col_types = "Dcciii") %>%
  dplyr::filter(state == "Iowa") %>%
  select(date, fips, county, cases, deaths) %>%
  arrange(desc(date), desc(cases)) %>%
  glimpse()
```

    ## Rows: 9,873
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-10, 2020-07-10, 2020-07-10, 2020-07-10, 2020-07-10, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19103, 19113, 19049, 19127, 19163,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Johnson", "L…
    ## $ cases  [3m[90m<int>[39m[23m 7212, 3318, 2465, 1733, 1406, 1368, 1366, 1080, 1004, 920, 854…
    ## $ deaths [3m[90m<int>[39m[23m 184, 44, 59, 11, 8, 83, 31, 19, 10, 23, 5, 12, 31, 45, 3, 0, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
