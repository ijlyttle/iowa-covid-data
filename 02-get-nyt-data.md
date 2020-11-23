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

    ## Rows: 23,370
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-11-22, 2020-11-22, 2020-11-22, 2020-11-22, 2020-11-22, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", …
    ## $ cases  [3m[90m<int>[39m[23m 31270, 13252, 10220, 10209, 9870, 9008, 8664, 6378, 5955, 5746…
    ## $ deaths [3m[90m<int>[39m[23m 322, 158, 75, 126, 111, 35, 90, 19, 55, 66, 25, 28, 41, 41, 37…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
