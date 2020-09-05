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

    ## Rows: 15,470
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-09-04, 2020-09-04, 2020-09-04, 2020-09-04, 2020-09-04, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19103, 19193, 19013, 19113, 19169, 19049, 19163, 19061,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Johnson", "Woodbury", "Black Hawk", "Linn", "Story", …
    ## $ cases  [3m[90m<int>[39m[23m 13859, 4421, 4266, 3950, 3114, 2818, 2456, 2334, 2139, 1853, 1…
    ## $ deaths [3m[90m<int>[39m[23m 240, 26, 56, 78, 94, 16, 38, 22, 36, 12, 34, 32, 54, 11, 52, 3…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
