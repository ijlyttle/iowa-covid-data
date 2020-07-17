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

    ## Rows: 10,473
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-16, 2020-07-16, 2020-07-16, 2020-07-16, 2020-07-16, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19103, 19113, 19049, 19163, 19127,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Johnson", "L…
    ## $ cases  [3m[90m<int>[39m[23m 8001, 3411, 2632, 1745, 1575, 1494, 1483, 1226, 1152, 1122, 91…
    ## $ deaths [3m[90m<int>[39m[23m 188, 46, 60, 12, 9, 85, 33, 10, 20, 23, 8, 13, 31, 45, 3, 0, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
