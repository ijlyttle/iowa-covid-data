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

    ## Rows: 9,173
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-07-03, 2020-07-03, 2020-07-03, 2020-07-03, 2020-07-03, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19049, 19113, 19103, 19127, 19169,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Dallas", "Li…
    ## $ cases  [3m[90m<int>[39m[23m 6366, 3207, 2235, 1709, 1249, 1246, 1243, 1041, 752, 748, 719,…
    ## $ deaths [3m[90m<int>[39m[23m 179, 44, 58, 11, 29, 82, 8, 19, 3, 10, 11, 30, 2, 22, 44, 0, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
