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

    ## Rows: 41,270
    ## Columns: 5
    ## $ date   <date> 2021-05-20, 2021-05-20, 2021-05-20, 2021-05-20, 2021-05-20, 20…
    ## $ fips   <int> 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19049, 19155, …
    ## $ county <chr> "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", "…
    ## $ cases  <int> 57837, 20957, 20141, 15856, 15170, 14521, 13399, 11219, 11159, …
    ## $ deaths <int> 630, 335, 243, 308, 228, 84, 209, 98, 169, 48, 90, 93, 89, 74, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
