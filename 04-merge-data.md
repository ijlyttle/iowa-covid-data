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

    ## Rows: 11,173
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

    ## Rows: 6,101
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
    ## [11] "2020-06-04" "2020-06-05" "2020-06-06" "2020-06-07" "2020-06-08"
    ## [16] "2020-06-09" "2020-06-10" "2020-06-11" "2020-06-12" "2020-06-13"
    ## [21] "2020-06-14" "2020-06-15" "2020-06-16" "2020-06-17" "2020-06-18"
    ## [26] "2020-06-19" "2020-06-20" "2020-06-21" "2020-06-22" "2020-06-23"
    ## [31] "2020-06-24" "2020-06-25" "2020-06-26" "2020-06-27" "2020-06-28"
    ## [36] "2020-06-29" "2020-06-30" "2020-07-01" "2020-07-02" "2020-07-03"
    ## [41] "2020-07-04" "2020-07-05" "2020-07-06" "2020-07-07" "2020-07-08"
    ## [46] "2020-07-09" "2020-07-10" "2020-07-11" "2020-07-12" "2020-07-13"
    ## [51] "2020-07-14" "2020-07-15" "2020-07-16" "2020-07-17" "2020-07-18"
    ## [56] "2020-07-19" "2020-07-20" "2020-07-21" "2020-07-22" "2020-07-23"
    ## [61] "2020-07-24"

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

    ## # A tibble: 11,279 x 8
    ##    date        fips county      cases deaths tests recovered active_cases
    ##    <date>     <dbl> <chr>       <dbl>  <dbl> <dbl>     <dbl>        <dbl>
    ##  1 2020-07-24 19153 Polk         8822    191 74705      5782         2849
    ##  2 2020-07-24 19193 Woodbury     3517     47 19509      3168          302
    ##  3 2020-07-24 19013 Black Hawk   2785     61 21493      1994          730
    ##  4 2020-07-24 19021 Buena Vista  1769     12  7586      1677           80
    ##  5 2020-07-24 19113 Linn         1727     87 28179      1352          288
    ##  6 2020-07-24 19103 Johnson      1705     10 21593      1119          576
    ##  7 2020-07-24 19049 Dallas       1631     34 14677      1133          464
    ##  8 2020-07-24 19163 Scott        1411     11 20893       684          716
    ##  9 2020-07-24 19061 Dubuque      1311     26 18166       488          797
    ## 10 2020-07-24 19127 Marshall     1263     21  7327       989          253
    ## # … with 11,269 more rows

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv"),
  na = ""
)
```
