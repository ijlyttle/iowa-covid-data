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

    ## Rows: 5,278
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-05-25, 2020-05-25, 2020-05-25, 2020-05-25, 2020-05-25, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19127, 19049, 19103, 19139, 19179,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Marshall", "Dallas"…
    ## $ cases  [3m[90m<int>[39m[23m 3795, 2635, 1686, 935, 870, 865, 603, 545, 514, 486, 395, 338,…
    ## $ deaths [3m[90m<int>[39m[23m 109, 24, 39, 75, 11, 15, 7, 39, 4, 2, 23, 9, 7, 16, 16, 0, 7, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
