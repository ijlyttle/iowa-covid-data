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

    ## Rows: 13,770
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-08-18, 2020-08-18, 2020-08-18, 2020-08-18, 2020-08-18, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19103, 19049, 19163, 19061, 19021,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Johnson", "Dallas",…
    ## $ cases  [3m[90m<int>[39m[23m 11101, 3838, 3343, 2596, 2255, 2002, 1874, 1823, 1808, 1522, 1…
    ## $ deaths [3m[90m<int>[39m[23m 212, 54, 67, 89, 22, 35, 17, 33, 12, 28, 30, 16, 38, 48, 8, 3,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
