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

    ## Rows: 8,973
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-01, 2020-07-01, 2020-07-01, 2020-07-01, 2020-07-01, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19113, 19049, 19103, 19127, 19169,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Linn", "Dall…
    ## $ cases  [3m[90m<int>[39m[23m 6139, 3184, 2138, 1699, 1218, 1197, 1186, 1017, 723, 707, 701,…
    ## $ deaths [3m[90m<int>[39m[23m 178, 44, 58, 11, 82, 29, 8, 18, 3, 11, 30, 10, 2, 44, 22, 0, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
