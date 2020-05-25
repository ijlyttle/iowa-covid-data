Population data
================

The purpose of this document is to write out the 2019 estimate for the
population of Iowa counties.

The source of this data is the [Iowa Community Indicators
Program](https://www.icip.iastate.edu/tables/population/counties-estimates).

``` r
library("fs")
library("magrittr")
library("iowa.covid")
library("tibble")
library("readr")
```

``` r
dir_create("data")

dir_target <- path("data", "population")
dir_create(dir_target)
```

``` r
iowa_county_population %>%
  glimpse()
```

    ## Rows: 99
    ## Columns: 6
    ## $ fips                  [3m[90m<dbl>[39m[23m 19153, 19113, 19163, 19103, 19013, 19193, 19061…
    ## $ county                [3m[90m<chr>[39m[23m "Polk", "Linn", "Scott", "Johnson", "Black Hawk…
    ## $ population            [3m[90m<dbl>[39m[23m 490161, 226706, 172943, 151140, 131228, 103107,…
    ## $ cumulative_population [3m[90m<dbl>[39m[23m 3155070, 2664909, 2438203, 2265260, 2114120, 19…
    ## $ quantile_population   [3m[90m<dbl>[39m[23m 1.0000000, 0.8446434, 0.7727889, 0.7179746, 0.6…
    ## $ population_group      [3m[90m<fct>[39m[23m large, large, mid-large, mid-large, mid-large, …

``` r
write_csv(
  iowa_county_population, 
  path(dir_target, "iowa_county_population.csv")
)
```
