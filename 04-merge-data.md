Merge data
================

The purpose of this document is to merge the data from all the sources
into some useful
    tables.

``` r
library("here")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data

``` r
library("vroom")
library("fs")
library("purrr")
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
library("conflicted")
library("readr")
```

    ## Registered S3 methods overwritten by 'readr':
    ##   method           from 
    ##   format.col_spec  vroom
    ##   print.col_spec   vroom
    ##   print.collector  vroom
    ##   print.date_names vroom
    ##   print.locale     vroom
    ##   str.col_spec     vroom

``` r
library("iowa.covid")

conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer dplyr::filter over any other package

Let’s define the directories and create the target directory.

``` r
dirs <- 
  list(
    source_state = path("data", "scrape-site"),
    source_nyt = path("data", "nyt"),
    target = path("data", "merged")  
  ) %>%
  map(here)

dir_create(dirs$target)
```

Let’s read in the NYT data:

``` r
nyt_data <- vroom(path(dirs$source_nyt, "nyt-iowa.csv"))
```

    ## Rows: 5,776
    ## Columns: 5
    ## Delimiter: ","
    ## chr  [1]: county
    ## dbl  [3]: fips, cases, deaths
    ## date [1]: date
    ## 
    ## Use `spec()` to retrieve the guessed column specification
    ## Pass a specification to the `col_types` argument to quiet this message

And the state data:

``` r
state_data <- vroom(dir_ls(dirs$source_state))
```

    ## Rows: 700
    ## Columns: 7
    ## Delimiter: ","
    ## chr  [1]: county
    ## dbl  [5]: fips, tests, cases, recovered, deaths
    ## date [1]: date
    ## 
    ## Use `spec()` to retrieve the guessed column specification
    ## Pass a specification to the `col_types` argument to quiet this message

Get the dates in the state dataset and exclude those from the NYT
dataset.

``` r
dates_state <- unique(state_data$date) %>% print() 
```

    ## [1] "2020-05-25" "2020-05-26" "2020-05-27" "2020-05-28" "2020-05-29"
    ## [6] "2020-05-30" "2020-05-31"

``` r
nyt_data_abridged <- 
  nyt_data %>%
  filter(!date %in% dates_state)
```

Now we can bind the data frames together. I take into consideration:

  - discarding rows unknown counties; these eventually get counted.
  - `cases`, `deaths`, `tests`, and `recovered` must always be
    increasing. Given a current value, all earlier values can be no
    greater than this value.
  - `active_cases` is the number of `cases` less `deaths` and
    `recovered`

<!-- end list -->

``` r
merged <- 
  bind_rows(nyt_data_abridged, state_data) %>%
  arrange(desc(date), desc(cases)) %>%
  mutate(
    county = ifelse(is.na(fips), NA_character_, county)
  ) %>%
  group_by(county) %>%
  mutate(
    across(
      c(cases, deaths, tests, recovered),
      ~ifelse(!is.na(county), cummin(.x), .x)
    ),
    active_cases = cases - deaths - recovered
  ) %>%
  ungroup() %>%
  print()
```

    ## # A tibble: 5,879 x 8
    ##    date        fips county      cases deaths tests recovered active_cases
    ##    <date>     <dbl> <chr>       <dbl>  <dbl> <dbl>     <dbl>        <dbl>
    ##  1 2020-05-31 19153 Polk         4225    126 24588      1798         2301
    ##  2 2020-05-31 19193 Woodbury     2748     34 11259      1647         1067
    ##  3 2020-05-31 19013 Black Hawk   1744     44  9374       999          701
    ##  4 2020-05-31 19113 Linn          953     77  9491       771          105
    ##  5 2020-05-31 19049 Dallas        902     20  4821       579          303
    ##  6 2020-05-31 19127 Marshall      894     16  3474       530          348
    ##  7 2020-05-31 19021 Buena Vista   754      0  4234        71          683
    ##  8 2020-05-31 19103 Johnson       613      9  7122       415          189
    ##  9 2020-05-31 19179 Wapello       589      9  2560       267          313
    ## 10 2020-05-31 19139 Muscatine     557     41  3024       418           98
    ## # … with 5,869 more rows

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv"),
  na = ""
)
```
