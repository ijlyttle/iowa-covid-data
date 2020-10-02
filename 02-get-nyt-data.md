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

    ## Rows: 18,170
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-10-01, 2020-10-01, 2020-10-01, 2020-10-01, 2020-10-01, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19103, 19013, 19113, 19061, 19169, 19163, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Johnson", "Black Hawk", "Linn", "Dubuque"…
    ## $ cases  [3m[90m<int>[39m[23m 16304, 5766, 5220, 4590, 4268, 3539, 3525, 3193, 2888, 2269, 2…
    ## $ deaths [3m[90m<int>[39m[23m 266, 69, 27, 92, 115, 43, 17, 29, 38, 40, 12, 35, 4, 14, 57, 2…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
