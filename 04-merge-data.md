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

    ## Rows: 5,477
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

    ## Rows: 396
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

    ## # A tibble: 5,544 x 8
    ##    date        fips county      cases deaths tests recovered active_cases
    ##    <date>     <dbl> <chr>       <dbl>  <dbl> <dbl>     <dbl>        <dbl>
    ##  1 2020-05-28 19153 Polk         3920    118 22393      1637         2165
    ##  2 2020-05-28 19193 Woodbury     2668     31 10737      1405         1232
    ##  3 2020-05-28 19013 Black Hawk   1716     43  8833       979          694
    ##  4 2020-05-28 19113 Linn          941     76  8688       760          105
    ##  5 2020-05-28 19127 Marshall      882     15  3388       498          369
    ##  6 2020-05-28 19049 Dallas        876     17  4536       557          302
    ##  7 2020-05-28 19021 Buena Vista   701      0  3906        58          643
    ##  8 2020-05-28 19103 Johnson       607      8  6717       383          216
    ##  9 2020-05-28 19139 Muscatine     549     41  2925       384          124
    ## 10 2020-05-28 19179 Wapello       542      4  2225       218          320
    ## # … with 5,534 more rows

Let’s write this out:

``` r
write_csv(
  merged,
  path(dirs$target, "merged.csv")
)
```
