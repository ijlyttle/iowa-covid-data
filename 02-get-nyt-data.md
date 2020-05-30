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

    ## Rows: 5,677
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-05-29, 2020-05-29, 2020-05-29, 2020-05-29, 2020-05-29, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19127, 19049, 19021, 19103, 19179,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Marshall", "Dallas"…
    ## $ cases  [3m[90m<int>[39m[23m 4072, 2726, 1730, 947, 892, 888, 724, 612, 556, 554, 504, 401,…
    ## $ deaths [3m[90m<int>[39m[23m 123, 34, 44, 76, 16, 20, 0, 9, 8, 41, 2, 27, 10, 18, 11, 16, 8…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
