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

    ## Rows: 11,873
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-30, 2020-07-30, 2020-07-30, 2020-07-30, 2020-07-30, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19103, 19021, 19049, 19163, 19061,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Johnson", "Buena Vi…
    ## $ cases  [3m[90m<int>[39m[23m 9388, 3612, 2947, 1970, 1856, 1782, 1722, 1542, 1452, 1364, 11…
    ## $ deaths [3m[90m<int>[39m[23m 201, 47, 62, 87, 14, 12, 35, 11, 28, 24, 21, 13, 31, 47, 3, 5,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
