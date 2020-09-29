Merge data
================

The purpose of this document is to merge the data from all the sources
into some useful tables.

``` r
library("here")
```

    ## here() starts at /Users/runner/work/iowa-covid-data/iowa-covid-data

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

    ## [conflicted] Will prefer [34mdplyr::filter[39m over any other package

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

    ## [1mRows:[22m 17,870
    ## [1mColumns:[22m 5
    ## [1mDelimiter:[22m ","
    ## [31mchr[39m  [1]: county
    ## [32mdbl[39m  [3]: fips, cases, deaths
    ## [34mdate[39m [1]: date
    ## 
    ## [90mUse `spec()` to retrieve the guessed column specification[39m
    ## [90mPass a specification to the `col_types` argument to quiet this message[39m

And the state data:

``` r
state_data <- vroom(dir_ls(dirs$source_state))
```

    ## [1mRows:[22m 12,701
    ## [1mColumns:[22m 7
    ## [1mDelimiter:[22m ","
    ## [31mchr[39m  [1]: county
    ## [32mdbl[39m  [5]: fips, tests, cases, recovered, deaths
    ## [34mdate[39m [1]: date
    ## 
    ## [90mUse `spec()` to retrieve the guessed column specification[39m
    ## [90mPass a specification to the `col_types` argument to quiet this message[39m

Get the dates in the state dataset and exclude those from the NYT
dataset.

``` r
dates_state <- unique(state_data$date) %>% print() 
```

    ##   [1] "2020-05-25" "2020-05-26" "2020-05-27" "2020-05-28" "2020-05-29"
    ##   [6] "2020-05-30" "2020-05-31" "2020-06-01" "2020-06-02" "2020-06-03"
    ##  [11] "2020-06-04" "2020-06-05" "2020-06-06" "2020-06-07" "2020-06-08"
    ##  [16] "2020-06-09" "2020-06-10" "2020-06-11" "2020-06-12" "2020-06-13"
    ##  [21] "2020-06-14" "2020-06-15" "2020-06-16" "2020-06-17" "2020-06-18"
    ##  [26] "2020-06-19" "2020-06-20" "2020-06-21" "2020-06-22" "2020-06-23"
    ##  [31] "2020-06-24" "2020-06-25" "2020-06-26" "2020-06-27" "2020-06-28"
    ##  [36] "2020-06-29" "2020-06-30" "2020-07-01" "2020-07-02" "2020-07-03"
    ##  [41] "2020-07-04" "2020-07-05" "2020-07-06" "2020-07-07" "2020-07-08"
    ##  [46] "2020-07-09" "2020-07-10" "2020-07-11" "2020-07-12" "2020-07-13"
    ##  [51] "2020-07-14" "2020-07-15" "2020-07-16" "2020-07-17" "2020-07-18"
    ##  [56] "2020-07-19" "2020-07-20" "2020-07-21" "2020-07-22" "2020-07-23"
    ##  [61] "2020-07-24" "2020-07-25" "2020-07-26" "2020-07-27" "2020-07-28"
    ##  [66] "2020-07-29" "2020-07-30" "2020-07-31" "2020-08-01" "2020-08-02"
    ##  [71] "2020-08-03" "2020-08-04" "2020-08-05" "2020-08-06" "2020-08-07"
    ##  [76] "2020-08-08" "2020-08-09" "2020-08-10" "2020-08-11" "2020-08-12"
    ##  [81] "2020-08-13" "2020-08-14" "2020-08-15" "2020-08-16" "2020-08-17"
    ##  [86] "2020-08-18" "2020-08-19" "2020-08-20" "2020-08-21" "2020-08-22"
    ##  [91] "2020-08-23" "2020-08-24" "2020-08-25" "2020-08-26" "2020-08-27"
    ##  [96] "2020-08-28" "2020-08-29" "2020-08-30" "2020-08-31" "2020-09-01"
    ## [101] "2020-09-02" "2020-09-03" "2020-09-04" "2020-09-05" "2020-09-06"
    ## [106] "2020-09-07" "2020-09-08" "2020-09-09" "2020-09-10" "2020-09-11"
    ## [111] "2020-09-12" "2020-09-13" "2020-09-14" "2020-09-15" "2020-09-16"
    ## [116] "2020-09-17" "2020-09-18" "2020-09-19" "2020-09-20" "2020-09-21"
    ## [121] "2020-09-22" "2020-09-23" "2020-09-24" "2020-09-25" "2020-09-26"
    ## [126] "2020-09-27" "2020-09-28"

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

    ## [90m# A tibble: 17,879 x 8[39m
    ##    date        fips county        cases deaths  tests recovered active_cases
    ##    [3m[90m<date>[39m[23m     [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m         [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m        [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-09-28 [4m1[24m[4m9[24m153 Polk          [4m1[24m[4m5[24m966    262 [4m1[24m[4m3[24m[4m1[24m631     [4m1[24m[4m3[24m259         [4m2[24m445
    ## [90m 2[39m 2020-09-28 [4m1[24m[4m9[24m193 Woodbury       [4m5[24m516     65  [4m3[24m[4m1[24m182      [4m4[24m115         [4m1[24m336
    ## [90m 3[39m 2020-09-28 [4m1[24m[4m9[24m103 Johnson        [4m5[24m157     27  [4m3[24m[4m9[24m532      [4m3[24m943         [4m1[24m187
    ## [90m 4[39m 2020-09-28 [4m1[24m[4m9[24m013 Black Hawk     [4m4[24m505     90  [4m3[24m[4m5[24m060      [4m3[24m734          681
    ## [90m 5[39m 2020-09-28 [4m1[24m[4m9[24m113 Linn           [4m4[24m083    112  [4m5[24m[4m0[24m791      [4m3[24m099          872
    ## [90m 6[39m 2020-09-28 [4m1[24m[4m9[24m169 Story          [4m3[24m457     17  [4m2[24m[4m9[24m900      [4m2[24m441          999
    ## [90m 7[39m 2020-09-28 [4m1[24m[4m9[24m061 Dubuque        [4m3[24m307     41  [4m2[24m[4m9[24m860      [4m2[24m015         [4m1[24m251
    ## [90m 8[39m 2020-09-28 [4m1[24m[4m9[24m163 Scott          [4m3[24m029     28  [4m3[24m[4m7[24m261      [4m2[24m227          774
    ## [90m 9[39m 2020-09-28 [4m1[24m[4m9[24m049 Dallas         [4m2[24m802     38  [4m2[24m[4m5[24m674      [4m2[24m346          418
    ## [90m10[39m 2020-09-28 [4m1[24m[4m9[24m155 Pottawattamie  [4m2[24m141     39  [4m2[24m[4m1[24m299      [4m1[24m736          366
    ## [90m# … with 17,869 more rows[39m

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv"),
  na = ""
)
```
