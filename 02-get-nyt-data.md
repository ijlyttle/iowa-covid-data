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

    ## Observations: 6,374
    ## Variables: 5
    ## $ date   <date> 2020-06-05, 2020-06-05, 2020-06-05, 2020-06-05, 2020-06-05, 2…
    ## $ fips   <int> 19153, 19193, 19013, 19021, 19113, 19049, 19127, 19179, 19103,…
    ## $ county <chr> "Polk", "Woodbury", "Black Hawk", "Buena Vista", "Linn", "Dall…
    ## $ cases  <int> 4650, 2887, 1788, 1091, 978, 958, 918, 640, 622, 567, 557, 412…
    ## $ deaths <int> 142, 37, 49, 2, 79, 26, 18, 15, 8, 41, 2, 29, 10, 21, 11, 10, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
