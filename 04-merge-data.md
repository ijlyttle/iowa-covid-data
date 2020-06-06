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

    ## Rows: 6,374
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

    ## Rows: 1,301
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

    ##  [1] "2020-05-25" "2020-05-26" "2020-05-27" "2020-05-28" "2020-05-29"
    ##  [6] "2020-05-30" "2020-05-31" "2020-06-01" "2020-06-02" "2020-06-03"
    ## [11] "2020-06-04" "2020-06-05" "2020-06-06"

``` r
nyt_data_abridged <- 
  nyt_data %>%
  filter(!date %in% dates_state)
```

Now we can bind the data frames together. I take into consideration:

  - discarding rows unknown counties; these eventually get counted.
  - `cases`, `deaths`, `tests`, and `recovered` must always be
    increasing. Given a current value, all earlier values can be no
    greater than this value. Also keep in mind that this is true *only*
    for a county - not for the pending results (county is `NA`).
  - `active_cases` is the number of `cases` less `deaths` and
    `recovered`.

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

    ## # A tibble: 6,480 x 8
    ##    date        fips county      cases deaths tests recovered active_cases
    ##    <date>     <dbl> <chr>       <dbl>  <dbl> <dbl>     <dbl>        <dbl>
    ##  1 2020-06-06 19153 Polk         4674    142 29571      2073         2459
    ##  2 2020-06-06 19193 Woodbury     2900     37 12514      2110          753
    ##  3 2020-06-06 19013 Black Hawk   1789     49 10607      1029          711
    ##  4 2020-06-06 19021 Buena Vista  1111      2  5246       199          910
    ##  5 2020-06-06 19113 Linn          981     79 11482       794          108
    ##  6 2020-06-06 19049 Dallas        960     26  5542       623          311
    ##  7 2020-06-06 19127 Marshall      919     18  4194       549          352
    ##  8 2020-06-06 19179 Wapello       640     16  3039       355          269
    ##  9 2020-06-06 19103 Johnson       624      8  8069       447          169
    ## 10 2020-06-06 19139 Muscatine     568     41  3265       461           66
    ## # … with 6,470 more rows

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv"),
  na = ""
)
```
