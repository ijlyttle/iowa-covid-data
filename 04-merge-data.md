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

    ## [1mRows:[22m 8,373
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

    ## [1mRows:[22m 3,201
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

    ##  [1] "2020-05-25" "2020-05-26" "2020-05-27" "2020-05-28" "2020-05-29"
    ##  [6] "2020-05-30" "2020-05-31" "2020-06-01" "2020-06-02" "2020-06-03"
    ## [11] "2020-06-04" "2020-06-05" "2020-06-06" "2020-06-07" "2020-06-08"
    ## [16] "2020-06-09" "2020-06-10" "2020-06-11" "2020-06-12" "2020-06-13"
    ## [21] "2020-06-14" "2020-06-15" "2020-06-16" "2020-06-17" "2020-06-18"
    ## [26] "2020-06-19" "2020-06-20" "2020-06-21" "2020-06-23" "2020-06-24"
    ## [31] "2020-06-25" "2020-06-26"

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

    ## [90m# A tibble: 8,479 x 8[39m
    ##    date        fips county        cases deaths tests recovered active_cases
    ##    [3m[90m<date>[39m[23m     [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m         [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m        [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-06-26 [4m1[24m[4m9[24m153 Polk           [4m5[24m758    174 [4m4[24m[4m5[24m523      [4m2[24m810         [4m2[24m774
    ## [90m 2[39m 2020-06-26 [4m1[24m[4m9[24m193 Woodbury       [4m3[24m112     43 [4m1[24m[4m5[24m418      [4m2[24m651          418
    ## [90m 3[39m 2020-06-26 [4m1[24m[4m9[24m013 Black Hawk     [4m1[24m980     57 [4m1[24m[4m3[24m923      [4m1[24m194          729
    ## [90m 4[39m 2020-06-26 [4m1[24m[4m9[24m021 Buena Vista    [4m1[24m678     11  [4m6[24m816       586         [4m1[24m081
    ## [90m 5[39m 2020-06-26 [4m1[24m[4m9[24m113 Linn           [4m1[24m156     81 [4m1[24m[4m7[24m628       893          182
    ## [90m 6[39m 2020-06-26 [4m1[24m[4m9[24m049 Dallas         [4m1[24m120     29  [4m8[24m905       706          385
    ## [90m 7[39m 2020-06-26 [4m1[24m[4m9[24m127 Marshall        998     18  [4m5[24m343       616          364
    ## [90m 8[39m 2020-06-26 [4m1[24m[4m9[24m103 Johnson         945      8 [4m1[24m[4m3[24m087       524          413
    ## [90m 9[39m 2020-06-26 [4m1[24m[4m9[24m179 Wapello         692     28  [4m3[24m983       609           55
    ## [90m10[39m 2020-06-26 [4m1[24m[4m9[24m155 Pottawattamie   656     11  [4m8[24m296       469          176
    ## [90m# … with 8,469 more rows[39m

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv"),
  na = ""
)
```
