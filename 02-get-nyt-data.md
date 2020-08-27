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

    ## Rows: 14,570
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-08-26, 2020-08-26, 2020-08-26, 2020-08-26, 2020-08-26, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19103, 19113, 19049, 19163, 19061, 19169,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Johnson", "Linn", "Dallas",…
    ## $ cases  [3m[90m<int>[39m[23m 12087, 4009, 3580, 2944, 2830, 2168, 2061, 1938, 1870, 1822, 1…
    ## $ deaths [3m[90m<int>[39m[23m 221, 54, 72, 26, 92, 37, 19, 36, 16, 12, 29, 34, 46, 49, 3, 3,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
