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
    lat = mean(max(lat) + min(lat)),
    long =  mean(max(long) + min(long))
  ) %>%
  mutate(
    county = str_to_title(county),
    county = ifelse(county == "Obrien", "O'Brien", county)
  ) %>%
  glimpse()
```

    ## Observations: 99
    ## Variables: 3
    ## $ county <chr> "Adair", "Adams", "Allamakee", "Appanoose", "Audubon", "Benton…
    ## $ lat    <dbl> 82.66635, 82.06475, 86.59111, 81.50898, 83.37109, 84.17323, 84…
    ## $ long   <dbl> -188.9557, -189.4198, -182.6704, -185.7357, -189.8152, -184.10…

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
  select(fips, county, lat, long, everything()) %>%
  glimpse()
```

    ## Observations: 99
    ## Variables: 7
    ## $ fips             <dbl> 19153, 19113, 19163, 19103, 19013, 19193, 19061, 191…
    ## $ county           <chr> "Polk", "Linn", "Scott", "Johnson", "Black Hawk", "W…
    ## $ lat              <dbl> 83.35390, 84.17323, 83.25650, 83.29660, 84.94672, 84…
    ## $ long             <dbl> -187.1567, -183.1746, -181.2036, -183.1746, -184.612…
    ## $ population       <dbl> 490161, 226706, 172943, 151140, 131228, 103107, 9731…
    ## $ population_group <fct> large, large, mid-large, mid-large, mid-large, mid-l…
    ## $ opening          <date> 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 202…

``` r
write_csv(
  iowa_county_meta, 
  path(dir_target, "iowa_county_meta.csv")
)
```
