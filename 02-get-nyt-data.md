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

    ## Rows: 5,179
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-05-24, 2020-05-24, 2020-05-24, 2020-05-24, 2020-05-24, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19127, 19049, 19103, 19139, 19179,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Marshall", "Dallas"…
    ## $ cases  [3m[90m<int>[39m[23m 3744, 2594, 1680, 932, 867, 853, 599, 544, 511, 484, 392, 334,…
    ## $ deaths [3m[90m<int>[39m[23m 108, 24, 39, 75, 11, 14, 7, 39, 4, 2, 23, 7, 9, 16, 16, 0, 6, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
