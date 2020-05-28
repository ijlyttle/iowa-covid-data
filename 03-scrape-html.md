Scrape Iowa site
================

``` r
library("rvest")
```

    ## Loading required package: xml2

``` r
library("tibble")
library("here")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data

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

    ## /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data/data/download-site/access-2020-05-25.html
    ## /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data/data/download-site/access-2020-05-26.html
    ## /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data/data/download-site/access-2020-05-27.html
    ## /Users/sesa19001/Documents/repos/public/graphics-group/iowa-covid-data/data/download-site/access-2020-05-28.html

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

    ## # A tibble: 99 x 6
    ##    date       county      tests cases recovered deaths
    ##    <date>     <chr>       <dbl> <dbl>     <dbl>  <dbl>
    ##  1 2020-05-28 Polk        22393  3920      1637    118
    ##  2 2020-05-28 Woodbury    10737  2668      1405     31
    ##  3 2020-05-28 Black Hawk   8833  1716       979     43
    ##  4 2020-05-28 Linn         8688   941       760     76
    ##  5 2020-05-28 Marshall     3388   882       498     15
    ##  6 2020-05-28 Dallas       4536   876       557     17
    ##  7 2020-05-28 Buena Vista  3906   701        58      0
    ##  8 2020-05-28 Johnson      6717   607       383      8
    ##  9 2020-05-28 Muscatine    2925   549       384     41
    ## 10 2020-05-28 Wapello      2225   542       218      4
    ## # … with 89 more rows

``` r
walk(files_needed, write_file, dir_target)
```
