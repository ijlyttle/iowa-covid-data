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
    lon =  mean(max(long) + min(long))
  ) %>%
  mutate(
    abbreviation = str_to_upper(county) %>% str_sub(1, 3),
    county = str_to_title(county),
    county = ifelse(county == "Obrien", "O'Brien", county)
  ) %>%
  glimpse()
```

    ## Rows: 99
    ## Columns: 4
    ## $ county       [3m[90m<chr>[39m[23m "Adair", "Adams", "Allamakee", "Appanoose", "Audubon", "…
    ## $ lat          [3m[90m<dbl>[39m[23m 82.66635, 82.06475, 86.59111, 81.50898, 83.37109, 84.173…
    ## $ lon          [3m[90m<dbl>[39m[23m -188.9557, -189.4198, -182.6704, -185.7357, -189.8152, -…
    ## $ abbreviation [3m[90m<chr>[39m[23m "ADA", "ADA", "ALL", "APP", "AUD", "BEN", "BLA", "BOO", …

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

    ## Rows: 99
    ## Columns: 8
    ## $ fips             [3m[90m<dbl>[39m[23m 19153, 19113, 19163, 19103, 19013, 19193, 19061, 191…
    ## $ county           [3m[90m<chr>[39m[23m "Polk", "Linn", "Scott", "Johnson", "Black Hawk", "W…
    ## $ abbreviation     [3m[90m<chr>[39m[23m "POL", "LIN", "SCO", "JOH", "BLA", "WOO", "DUB", "ST…
    ## $ lon              [3m[90m<dbl>[39m[23m -187.1567, -183.1746, -181.2036, -183.1746, -184.612…
    ## $ lat              [3m[90m<dbl>[39m[23m 83.35390, 84.17323, 83.25650, 83.29660, 84.94672, 84…
    ## $ population       [3m[90m<dbl>[39m[23m 490161, 226706, 172943, 151140, 131228, 103107, 9731…
    ## $ population_group [3m[90m<fct>[39m[23m large, large, mid-large, mid-large, mid-large, mid-l…
    ## $ opening          [3m[90m<date>[39m[23m 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 202…

``` r
write_csv(
  iowa_county_meta, 
  path(dir_target, "iowa_county_meta.csv")
)
```
