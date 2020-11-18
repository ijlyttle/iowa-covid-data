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

    ## Rows: 22,870
    ## Columns: 5
    ## $ date   [3m[90m<date>[39m[23m 2020-11-17, 2020-11-17, 2020-11-17, 2020-11-17, 2020-11-17, 2…
    ## $ fips   [3m[90m<int>[39m[23m 19153, 19113, 19013, 19193, 19163, 19103, 19061, 19169, 19049,…
    ## $ county [3m[90m<chr>[39m[23m "Polk", "Linn", "Black Hawk", "Woodbury", "Scott", "Johnson", …
    ## $ cases  [3m[90m<int>[39m[23m 28712, 12158, 9414, 9360, 8964, 8433, 8068, 5734, 5485, 5080, …
    ## $ deaths [3m[90m<int>[39m[23m 310, 150, 122, 109, 64, 35, 86, 19, 54, 58, 22, 39, 27, 35, 13…

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
