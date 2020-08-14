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

    ## Rows: 13,271
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-08-13, 2020-08-13, 2020-08-13, 2020-08-13, 2020-08-13, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19103, 19049, 19021, 19163, 19061,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Johnson", "Dallas",…
    ## $ cases  [3m[90m<int>[39m[23m 10673, 3779, 3215, 2469, 2146, 1942, 1801, 1784, 1744, 1465, 1…
    ## $ deaths [3m[90m<int>[39m[23m 210, 52, 66, 89, 21, 35, 12, 15, 31, 27, 29, 15, 36, 48, 8, 3,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
