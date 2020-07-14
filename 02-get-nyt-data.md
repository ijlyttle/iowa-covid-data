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

    ## Rows: 10,173
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-13, 2020-07-13, 2020-07-13, 2020-07-13, 2020-07-13, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19103, 19049, 19113, 19127, 19163,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Johnson", "D…
    ## $ cases  [3m[90m<int>[39m[23m 7611, 3349, 2534, 1738, 1487, 1415, 1410, 1100, 1098, 1000, 87…
    ## $ deaths [3m[90m<int>[39m[23m 185, 44, 59, 11, 8, 31, 83, 19, 10, 23, 8, 13, 31, 45, 3, 0, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
