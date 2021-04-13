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

    ## Rows: 37,470
    ## Columns: 5
    ## $ date   <date> 2021-04-12, 2021-04-12, 2021-04-12, 2021-04-12, 2021-04-12, 20…
    ## $ fips   <int> 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19049, 19155, …
    ## $ county <chr> "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", "…
    ## $ cases  <int> 55552, 20170, 18763, 15468, 14792, 13876, 13035, 10827, 10627, …
    ## $ deaths <int> 598, 329, 234, 306, 219, 80, 202, 96, 160, 47, 86, 89, 86, 91, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
