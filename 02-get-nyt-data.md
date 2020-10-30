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

    ## Rows: 20,970
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-10-29, 2020-10-29, 2020-10-29, 2020-10-29, 2020-10-29, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19113, 19103, 19013, 19061, 19163, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Linn", "Johnson", "Black Hawk", "Dubuque"…
    ## $ cases  [3m[90m<int>[39m[23m 19766, 7450, 6069, 6025, 5982, 5511, 4899, 4118, 3578, 3385, 2…
    ## $ deaths [3m[90m<int>[39m[23m 289, 99, 133, 31, 102, 58, 43, 18, 47, 45, 17, 12, 36, 15, 31,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
