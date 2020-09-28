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

    ## Rows: 17,770
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-09-27, 2020-09-27, 2020-09-27, 2020-09-27, 2020-09-27, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19103, 19013, 19113, 19169, 19061, 19163, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Johnson", "Black Hawk", "Linn", "Story", …
    ## $ cases  [3m[90m<int>[39m[23m 15962, 5509, 5147, 4500, 4078, 3453, 3283, 3020, 2798, 2141, 1…
    ## $ deaths [3m[90m<int>[39m[23m 262, 64, 27, 90, 112, 17, 41, 28, 38, 39, 12, 34, 3, 57, 14, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
