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

    ## Rows: 22,970
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-11-18, 2020-11-18, 2020-11-18, 2020-11-18, 2020-11-18, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19113, 19013, 19193, 19163, 19103, 19061, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Linn", "Black Hawk", "Woodbury", "Scott", "Johnson", …
    ## $ cases  [3m[90m<int>[39m[23m 29223, 12407, 9616, 9488, 9208, 8555, 8200, 5905, 5577, 5234, …
    ## $ deaths [3m[90m<int>[39m[23m 315, 150, 124, 109, 66, 35, 88, 19, 54, 61, 22, 27, 40, 37, 37…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
