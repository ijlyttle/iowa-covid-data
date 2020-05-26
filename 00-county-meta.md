County metadata
================

The purpose of this document is to write out the 2019 estimate for the
population of Iowa counties, along with other metadata.

The source of this data is the [Iowa Community Indicators
Program](https://www.icip.iastate.edu/tables/population/counties-estimates).

``` r
library("fs")
library("magrittr")
library("iowa.covid")
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
library("readr")
```

``` r
dir_create("data")

dir_target <- path("data", "meta")
dir_create(dir_target)
```

``` r
county_closed <- 
  c(
    "Allamakee", 
    "Benton", 
    "Black Hawk", 
    "Bremer", 
    "Dallas", 
    "Des Moines", 
    "Dubuque", 
    "Fayette", 
    "Henry", 
    "Iowa", 
    "Jasper", 
    "Johnson", 
    "Linn", 
    "Louisa", 
    "Marshall", 
    "Muscatine", 
    "Polk", 
    "Poweshiek", 
    "Scott", 
    "Tama", 
    "Washington", 
    "Woodbury"
  )
```

``` r
iowa_county_meta <-
  iowa_county_population %>%
  select(fips, county, population, population_group) %>%
  mutate(
    opening = 
      ifelse(county %in% county_closed, "2020-05-15", "2020-05-01") %>%
      as.Date()
  ) %>%
  glimpse()
```

    ## Rows: 99
    ## Columns: 5
    ## $ fips             [3m[90m<dbl>[39m[23m 19153, 19113, 19163, 19103, 19013, 19193, 19061, 191…
    ## $ county           [3m[90m<chr>[39m[23m "Polk", "Linn", "Scott", "Johnson", "Black Hawk", "W…
    ## $ population       [3m[90m<dbl>[39m[23m 490161, 226706, 172943, 151140, 131228, 103107, 9731…
    ## $ population_group [3m[90m<fct>[39m[23m large, large, mid-large, mid-large, mid-large, mid-l…
    ## $ opening          [3m[90m<date>[39m[23m 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 202…

``` r
write_csv(
  iowa_county_meta, 
  path(dir_target, "iowa_county_meta.csv")
)
```
