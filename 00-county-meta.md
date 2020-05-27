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
library("ggplot2")
library("stringr")
```

``` r
dir_create("data")

dir_target <- path("data", "meta")
dir_create(dir_target)
```

In late April, Gov. Reynolds announced that she would partially re-open
77 counties effective May 1. The remaining 22 counties were similarly
re-opened May 15. These are the 22 “later” counties:

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

We also want to compute the approximate centers of the counties.

``` r
map_iowa_county <-
  map_data("county") %>%
  filter(region == "iowa") %>%
  group_by(county = subregion) %>%
  summarise(
    lat = (max(lat) + min(lat)) / 2,
    lon =  (max(long) + min(long)) / 2
  ) %>%
  mutate(
    abbreviation = str_to_upper(county) %>% str_sub(1, 3),
    county = str_to_title(county),
    county = ifelse(county == "Obrien", "O'Brien", county)
  ) %>%
  glimpse()
```

    ## Observations: 99
    ## Variables: 4
    ## $ county       <chr> "Adair", "Adams", "Allamakee", "Appanoose", "Audubon", "…
    ## $ lat          <dbl> 41.33318, 41.03237, 43.29556, 40.75449, 41.68554, 42.086…
    ## $ lon          <dbl> -94.47787, -94.70992, -91.33520, -92.86787, -94.90759, -…
    ## $ abbreviation <chr> "ADA", "ADA", "ALL", "APP", "AUD", "BEN", "BLA", "BOO", …

``` r
iowa_county_meta <-
  iowa_county_population %>%
  select(fips, county, population, population_group) %>%
  mutate(
    opening = 
      ifelse(county %in% county_closed, "2020-05-15", "2020-05-01") %>%
      as.Date()
  ) %>%
  left_join(
    map_iowa_county, 
    by = "county"
  ) %>%
  select(fips, county, abbreviation, lon, lat, everything()) %>%
  glimpse()
```

    ## Observations: 99
    ## Variables: 8
    ## $ fips             <dbl> 19153, 19113, 19163, 19103, 19013, 19193, 19061, 191…
    ## $ county           <chr> "Polk", "Linn", "Scott", "Johnson", "Black Hawk", "W…
    ## $ abbreviation     <chr> "POL", "LIN", "SCO", "JOH", "BLA", "WOO", "DUB", "ST…
    ## $ lon              <dbl> -93.57833, -91.58730, -90.60182, -91.58730, -92.3063…
    ## $ lat              <dbl> 41.67695, 42.08661, 41.62825, 41.64830, 42.47336, 42…
    ## $ population       <dbl> 490161, 226706, 172943, 151140, 131228, 103107, 9731…
    ## $ population_group <fct> large, large, mid-large, mid-large, mid-large, mid-l…
    ## $ opening          <date> 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 202…

``` r
write_csv(
  iowa_county_meta, 
  path(dir_target, "iowa_county_meta.csv")
)
```
