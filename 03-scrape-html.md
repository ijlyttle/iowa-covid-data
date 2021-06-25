Scrape Iowa site
================

The purpose of this document is to scrape the downloaded pages into csv
files.

``` r
library("rvest")
library("tibble")
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
library("here")
```

    ## here() starts at /Users/runner/work/iowa-covid-data/iowa-covid-data

``` r
library("fs")
library("stringr")
library("purrr")
library("iowa.covid")
```

First, we define the source and target directories.

``` r
dir_source_rel <- path("data", "download-site")
dir_target_rel <- path("data", "scrape-site")

dir_source <- here(dir_source_rel)
dir_target <- here(dir_target_rel)

dir_create(dir_target)
```

Next we define some local functions.

First, a function that, given the HTML for the state’s page, returns the
date tag embedded in the page.

``` r
extract_date <- function(html) {
  
  text <- html_text(html)
  
  str_date_year <- str_extract(text, "\\d{1,2}/\\d{1,2}/\\d{4}")[[1]]
  str_date_no_year <- str_extract(text, '\\d{1,2}/\\d{1,2}(?= )')[[1]]

  str_date <- NA_character_
  
  if (!is.na(str_date_year)) {
    str_date <- str_date_year
  }
  
  if (!is.na(str_date_no_year)) {
    str_date <- glue::glue("{str_date_no_year}/2020")
  }
  
  date <- readr::parse_date(str_date, format = "%m/%d/%Y")
  
  date
}
```

Next, a function that, given the HTML, returns a data-frame with:

  - `date`
  - `county`
  - `tests`
  - `cases`
  - `recovered`
  - `deaths`

All values are cumulative.

``` r
extract_data <- function(html, date) {

  content <-
    html %>%
    rvest::html_nodes("td") %>%
    purrr::map_chr(rvest::html_text)
  
  ind_counties <- 
    which(
      content %in% iowa.covid::iowa_county_population$county |
      str_detect(content, regex("^pending", ignore_case = TRUE))  
    )
  
  iowa_data <- 
    tibble::tibble(
      date = date,
      county = content[ind_counties],
      tests = as.numeric(content[ind_counties + 1]),
      cases = as.numeric(content[ind_counties + 2]),
      recovered = as.numeric(content[ind_counties + 3]),
      deaths = as.numeric(content[ind_counties + 4])
    ) %>%
    dplyr::mutate(
      tests = ifelse(is.na(tests), 0, tests),
      cases = ifelse(is.na(cases), 0, cases),
      recovered = ifelse(is.na(recovered), 0, recovered),
      deaths = ifelse(is.na(deaths), 0, deaths),
    ) %>% 
    dplyr::left_join(
      iowa.covid::iowa_county_population %>% dplyr::select(fips, county),
      by = "county"
    ) %>%
    dplyr::select(date, fips, county, everything())
    
  
  iowa_data
}
```

A function that, given a directory, returns a named vector of candidate
files:

``` r
get_date_files <- function(dir) {
  
  regex_date <- ".*access-(\\d{4}-\\d{2}-\\d{2})\\.(csv|html)$"
  
  files <- dir_ls(dir, regexp = regex_date)
  dates <- str_replace(files, regex_date, "\\1")
  
  names(files) <- dates
  
  files
} 
```

Let’s determine the HTML files we need to scrape; these are the files
that do not have corresponding CSV files in the target directory.

``` r
files_source <- get_date_files(dir_source)
files_target <- 
  get_date_files(dir_target) %>%
  head(-1) # always parse the most-recent file

dates_needed <- 
  names(files_source)[!names(files_source) %in% names(files_target)]

files_needed <- files_source[dates_needed]
files_needed
```

    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2020-11-07.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-01-12.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-01-18.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-03-06.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-03-29.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-06-03.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-06-24.html
    ## /Users/runner/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2021-06-25.html

Finally, we need a function, given a filepath to an html file, and a
target directory, scrape the html file and write a CSV file in the
target directory.

``` r
write_file <- function(file_html, dir_target) {
  
  html <- xml2::read_html(file_html)
  date <- extract_date(html)
  data <- extract_data(html, date)
  
  readr::write_csv(
    data, 
    path = fs::path(dir_target, glue::glue("access-{date}.csv"))
  )
}
```

Test some of our functions:

``` r
html <- read_html(path(dir_source, "access-2020-06-22.html"))

date <- extract_date(html)
```

``` r
data <- extract_data(html, date)
print(data)
```

    ## # A tibble: 100 x 7
    ##    date        fips county      tests cases recovered deaths
    ##    <date>     <dbl> <chr>       <dbl> <dbl>     <dbl>  <dbl>
    ##  1 2020-06-22 19153 Polk        42147  5510      2669    167
    ##  2 2020-06-22 19193 Woodbury    14893  3069      2587     42
    ##  3 2020-06-22 19013 Black Hawk  13042  1896      1160     56
    ##  4 2020-06-22 19021 Buena Vista  6655  1667       476     10
    ##  5 2020-06-22 19113 Linn        16349  1097       857     80
    ##  6 2020-06-22 19049 Dallas       8207  1075       689     29
    ##  7 2020-06-22 19127 Marshall     5100   974       608     18
    ##  8 2020-06-22 19103 Johnson     11612   770       497      8
    ##  9 2020-06-22 19179 Wapello      3815   689       555     27
    ## 10 2020-06-22 19047 Crawford     2651   640       376      2
    ## # … with 90 more rows

Finally, create the new CSV files.

``` r
walk(files_needed, write_file, dir_target)
```

    ## Warning: The `path` argument of `write_csv()` is deprecated as of readr 1.4.0.
    ## Please use the `file` argument instead.
