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

    ## Rows: 38,770
    ## Columns: 5
    ## $ date   <date> 2021-04-25, 2021-04-25, 2021-04-25, 2021-04-25, 2021-04-25, 20…
    ## $ fips   <int> 19153, 19113, 19163, 19013, 19193, 19103, 19061, 19049, 19155, …
    ## $ county <chr> "Polk", "Linn", "Scott", "Black Hawk", "Woodbury", "Johnson", "…
    ## $ cases  <int> 56574, 20483, 19376, 15645, 15002, 14226, 13232, 10999, 10892, …
    ## $ deaths <int> 612, 332, 237, 306, 224, 83, 205, 97, 164, 47, 87, 92, 87, 73, …

``` r
write_csv(
  iowa_county_data,
  path(dir_target, "nyt-iowa.csv")
)
```
