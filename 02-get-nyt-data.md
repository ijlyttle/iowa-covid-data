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

    ## Rows: 7,274
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-06-14, 2020-06-14, 2020-06-14, 2020-06-14, 2020-06-14, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19021, 19049, 19113, 19127, 19179, 19103,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Dallas", "Li…
    ## $ cases  [3m[90m<int>[39m[23m 5169, 3004, 1834, 1588, 1026, 1018, 941, 675, 638, 618, 574, 5…
    ## $ deaths [3m[90m<int>[39m[23m 161, 40, 53, 6, 26, 80, 18, 26, 8, 2, 43, 11, 29, 10, 21, 11, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
