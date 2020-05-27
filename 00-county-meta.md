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
    lat = mean(max(lat), min(lat)),
    lon =  mean(max(long), min(long))
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
    ## $ lat          <dbl> 41.50506, 41.16129, 43.51041, 40.90346, 41.86603, 42.301…
    ## $ lon          <dbl> -94.24583, -94.48647, -91.06018, -92.63009, -94.70992, -…
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
    ## $ lon              <dbl> -93.33482, -91.35239, -90.32107, -91.34666, -92.0628…
    ## $ lat              <dbl> 41.86030, 42.30148, 41.79727, 41.87175, 42.64525, 42…
    ## $ population       <dbl> 490161, 226706, 172943, 151140, 131228, 103107, 9731…
    ## $ population_group <fct> large, large, mid-large, mid-large, mid-large, mid-l…
    ## $ opening          <date> 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 202…

``` r
write_csv(
  iowa_county_meta, 
  path(dir_target, "iowa_county_meta.csv")
)
```
