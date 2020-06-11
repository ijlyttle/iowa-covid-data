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

    ## Rows: 6,874
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-06-10, 2020-06-10, 2020-06-10, 2020-06-10, 2020-06-10, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19113, 19049, 19127, 19179, 19103,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Linn", "Dall…
    ## $ cases  [3m[90m<int>[39m[23m 4924, 2960, 1817, 1359, 1007, 974, 933, 660, 625, 589, 569, 43…
    ## $ deaths [3m[90m<int>[39m[23m 155, 38, 53, 4, 80, 26, 18, 23, 8, 2, 43, 10, 29, 10, 21, 11, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
