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

    ## Rows: 40,370
    ## Columns: 5
    ## $ date   <date> 2021-05-11, 2021-05-11, 2021-05-11, 2021-05-11, 2021-05-11, 20…
    ## $ fips   <int> 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19049, 19155, …
    ## $ county <chr> "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", "…
    ## $ cases  <int> 57488, 20816, 19958, 15783, 15120, 14454, 13354, 11159, 11091, …
    ## $ deaths <int> 623, 334, 240, 308, 228, 83, 209, 98, 168, 48, 88, 93, 89, 74, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
