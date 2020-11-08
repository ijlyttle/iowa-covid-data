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

    ## Rows: 21,870
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-11-07, 2020-11-07, 2020-11-07, 2020-11-07, 2020-11-07, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19113, 19193, 19013, 19103, 19061, 19163, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Linn", "Woodbury", "Black Hawk", "Johnson", "Dubuque"…
    ## $ cases  [3m[90m<int>[39m[23m 22928, 8416, 8296, 7455, 6878, 6662, 6288, 4761, 4303, 4002, 2…
    ## $ deaths [3m[90m<int>[39m[23m 297, 140, 102, 108, 33, 69, 51, 18, 51, 49, 20, 37, 13, 18, 34…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
