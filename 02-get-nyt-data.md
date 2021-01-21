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

    ## Rows: 29,270
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2021-01-20, 2021-01-20, 2021-01-20, 2021-01-20, 2021-01-20, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19155, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", …
    ## $ cases  [3m[90m<int>[39m[23m 46016, 17864, 15566, 13875, 13040, 12160, 11434, 9047, 8965, 8…
    ## $ deaths [3m[90m<int>[39m[23m 457, 278, 166, 238, 177, 51, 153, 113, 72, 35, 72, 71, 57, 63,…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
