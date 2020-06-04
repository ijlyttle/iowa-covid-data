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

    ## Rows: 6,174
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-06-03, 2020-06-03, 2020-06-03, 2020-06-03, 2020-06-03, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19193, 19013, 19113, 19021, 19049, 19127, 19103, 19179,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Woodbury", "Black Hawk", "Linn", "Buena Vista", "Dall…
    ## $ cases  [3m[90m<int>[39m[23m 4418, 2819, 1759, 967, 951, 929, 901, 616, 615, 564, 543, 408,…
    ## $ deaths [3m[90m<int>[39m[23m 134, 37, 46, 79, 1, 24, 18, 9, 14, 41, 2, 28, 10, 21, 11, 10, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
