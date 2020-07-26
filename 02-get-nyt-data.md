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

    ## Rows: 11,373
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-25, 2020-07-25, 2020-07-25, 2020-07-25, 2020-07-25, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19021, 19103, 19049, 19163, 19061,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Buena Vista", "John…
    ## $ cases  [3m[90m<int>[39m[23m 8985, 3539, 2838, 1800, 1774, 1735, 1650, 1458, 1340, 1285, 10…
    ## $ deaths [3m[90m<int>[39m[23m 191, 47, 61, 87, 12, 10, 34, 11, 26, 22, 11, 18, 31, 46, 3, 5,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
