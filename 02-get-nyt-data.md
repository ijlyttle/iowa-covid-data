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

    ## Rows: 20,270
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-10-22, 2020-10-22, 2020-10-22, 2020-10-22, 2020-10-22, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19103, 19013, 19113, 19061, 19163, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Johnson", "Black Hawk", "Linn", "Dubuque"…
    ## $ cases  [3m[90m<int>[39m[23m 18460, 7001, 5735, 5397, 5279, 4936, 4280, 3915, 3376, 3093, 2…
    ## $ deaths [3m[90m<int>[39m[23m 285, 94, 30, 98, 129, 55, 37, 18, 44, 44, 16, 12, 36, 14, 27, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
