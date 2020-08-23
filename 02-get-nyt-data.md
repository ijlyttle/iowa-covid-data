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

    ## Rows: 14,170
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-08-22, 2020-08-22, 2020-08-22, 2020-08-22, 2020-08-22, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19103, 19049, 19163, 19061, 19021,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Johnson", "Dallas",…
    ## $ cases  [3m[90m<int>[39m[23m 11653, 3933, 3501, 2729, 2454, 2100, 1992, 1899, 1814, 1565, 1…
    ## $ deaths [3m[90m<int>[39m[23m 214, 54, 69, 90, 24, 36, 18, 35, 12, 28, 16, 31, 43, 48, 3, 3,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
