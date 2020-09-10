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

    ## Rows: 15,970
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-09-09, 2020-09-09, 2020-09-09, 2020-09-09, 2020-09-09, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19103, 19193, 19013, 19113, 19169, 19049, 19163, 19061,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Johnson", "Woodbury", "Black Hawk", "Linn", "Story", …
    ## $ cases  [3m[90m<int>[39m[23m 14219, 4672, 4378, 4077, 3330, 2992, 2508, 2409, 2263, 1863, 1…
    ## $ deaths [3m[90m<int>[39m[23m 248, 26, 58, 81, 98, 16, 38, 24, 37, 12, 35, 32, 56, 13, 52, 3…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
