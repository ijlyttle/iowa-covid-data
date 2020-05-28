Scrape Iowa site
================

``` r
library("rvest")
```

    ## Loading required package: xml2

``` r
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

    ## here() starts at /Users/runner/runners/2.262.1/work/iowa-covid-data/iowa-covid-data

``` r
library("fs")
library("stringr")
library("purrr")
```

    ## 
    ## Attaching package: 'purrr'

    ## The following object is masked from 'package:rvest':
    ## 
    ##     pluck

``` r
dir_source_rel <- path("data", "download-site")
dir_target_rel <- path("data", "scrape-site")

dir_source <- here(dir_source_rel)
dir_target <- here(dir_target_rel)

dir_create(dir_target)
```

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

``` r
extract_data <- function(html, date) {
  
  content <-
    html %>%
    rvest::html_nodes("td") %>%
    purrr::map_chr(rvest::html_text)
  
  ind_counties <- 
    which(content %in% iowa.covid::iowa_county_population$county)
  
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
    )
  
  iowa_data
}
```

Let’s find out what data we have already scraped.

We want a function, given a directory, returns a named vector of
candidate files.

``` r
get_date_files <- function(dir) {
  
  regex_date <- ".*access-(\\d{4}-\\d{2}-\\d{2})\\.(csv|html)$"
  
  files <- dir_ls(dir, regexp = regex_date)
  dates <- str_replace(files, regex_date, "\\1")
  
  names(files) <- dates
  
  files
} 
```

``` r
files_source <- get_date_files(dir_source)
files_target <- get_date_files(dir_target)

dates_needed <- 
  names(files_source)[!names(files_source) %in% names(files_target)]

files_needed <- files_source[dates_needed]
files_needed
```

    ## /Users/runner/runners/2.262.1/work/iowa-covid-data/iowa-covid-data/data/download-site/access-2020-05-28.html

Now, we need a function, given a filepath to an html file, and a target
directory, scrape the html file and write a CSV file in the target
directory.

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

``` r
html <- read_html(path(dir_source, "access-2020-05-28.html"))

date <- extract_date(html)
```

``` r
extract_data(html, date)
```

    ## [90m# A tibble: 99 x 6[39m
    ##    date       county      tests cases recovered deaths
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m       [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-05-28 Polk        [4m2[24m[4m2[24m393  [4m3[24m920      [4m1[24m637    118
    ## [90m 2[39m 2020-05-28 Woodbury    [4m1[24m[4m0[24m737  [4m2[24m668      [4m1[24m405     31
    ## [90m 3[39m 2020-05-28 Black Hawk   [4m8[24m833  [4m1[24m716       979     43
    ## [90m 4[39m 2020-05-28 Linn         [4m8[24m688   941       760     76
    ## [90m 5[39m 2020-05-28 Marshall     [4m3[24m388   882       498     15
    ## [90m 6[39m 2020-05-28 Dallas       [4m4[24m536   876       557     17
    ## [90m 7[39m 2020-05-28 Buena Vista  [4m3[24m906   701        58      0
    ## [90m 8[39m 2020-05-28 Johnson      [4m6[24m717   607       383      8
    ## [90m 9[39m 2020-05-28 Muscatine    [4m2[24m925   549       384     41
    ## [90m10[39m 2020-05-28 Wapello      [4m2[24m225   542       218      4
    ## [90m# … with 89 more rows[39m

``` r
walk(files_needed, write_file, dir_target)
```