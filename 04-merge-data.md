Merge data
================

The purpose of this document is to merge the data from all the sources
into some useful tables.

``` r
library("here")
```

    ## here() starts at /Users/runner/runners/2.262.1/work/iowa-covid-data/iowa-covid-data

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

    ## [1mRows:[22m 5,477
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

    ## [1mRows:[22m 396
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

    ## [1] "2020-05-25" "2020-05-26" "2020-05-27" "2020-05-28"

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
  filter(!is.na(fips)) %>%
  group_by(county) %>%
  mutate(
    cases = cummin(cases),
    deaths = cummin(deaths),
    tests = cummin(tests),
    recovered = cummin(recovered),
    active_cases = cases - deaths - recovered
  ) %>%
  ungroup() %>%
  print()
```

    ## [90m# A tibble: 5,544 x 8[39m
    ##    date        fips county      cases deaths tests recovered active_cases
    ##    [3m[90m<date>[39m[23m     [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m       [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m        [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-05-28 [4m1[24m[4m9[24m153 Polk         [4m3[24m920    118 [4m2[24m[4m2[24m393      [4m1[24m637         [4m2[24m165
    ## [90m 2[39m 2020-05-28 [4m1[24m[4m9[24m193 Woodbury     [4m2[24m668     31 [4m1[24m[4m0[24m737      [4m1[24m405         [4m1[24m232
    ## [90m 3[39m 2020-05-28 [4m1[24m[4m9[24m013 Black Hawk   [4m1[24m716     43  [4m8[24m833       979          694
    ## [90m 4[39m 2020-05-28 [4m1[24m[4m9[24m113 Linn          941     76  [4m8[24m688       760          105
    ## [90m 5[39m 2020-05-28 [4m1[24m[4m9[24m127 Marshall      882     15  [4m3[24m388       498          369
    ## [90m 6[39m 2020-05-28 [4m1[24m[4m9[24m049 Dallas        876     17  [4m4[24m536       557          302
    ## [90m 7[39m 2020-05-28 [4m1[24m[4m9[24m021 Buena Vista   701      0  [4m3[24m906        58          643
    ## [90m 8[39m 2020-05-28 [4m1[24m[4m9[24m103 Johnson       607      8  [4m6[24m717       383          216
    ## [90m 9[39m 2020-05-28 [4m1[24m[4m9[24m139 Muscatine     549     41  [4m2[24m925       384          124
    ## [90m10[39m 2020-05-28 [4m1[24m[4m9[24m179 Wapello       542      4  [4m2[24m225       218          320
    ## [90m# … with 5,534 more rows[39m

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv")
)
```
