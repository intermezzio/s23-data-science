COVID-19
================
Andrew Mascillaro
2023-01-24

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Due Date](#due-date)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute the mean and standard deviation for
      `cases_per100k` and
      `deaths_per100k`.](#q6-compute-the-mean-and-standard-deviation-for-cases_per100k-and-deaths_per100k)
    - [**q7** Find the top 10 counties in terms of `cases_per100k`, and
      the top 10 in terms of `deaths_per100k`. Report the population of
      each county along with the per-100,000 counts. Compare the counts
      against the mean values you found in q6. Note any
      observations.](#q7-find-the-top-10-counties-in-terms-of-cases_per100k-and-the-top-10-in-terms-of-deaths_per100k-report-the-population-of-each-county-along-with-the-per-100000-counts-compare-the-counts-against-the-mean-values-you-found-in-q6-note-any-observations)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category    | Needs Improvement                                                                                                | Satisfactory                                                                                                               |
|-------------|------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Effort      | Some task **q**’s left unattempted                                                                               | All task **q**’s attempted                                                                                                 |
| Observed    | Did not document observations, or observations incorrect                                                         | Documented correct observations based on analysis                                                                          |
| Supported   | Some observations not clearly supported by analysis                                                              | All observations clearly supported by analysis (table, graph, etc.)                                                        |
| Assessed    | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support      |
| Specified   | Uses the phrase “more data are necessary” without clarification                                                  | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability                                 | Code sufficiently close to the [style guide](https://style.tidyverse.org/)                                                 |

## Due Date

<!-- ------------------------- -->

All the deliverables stated in the rubrics above are due **at midnight**
before the day of the class discussion of the challenge. See the
[Syllabus](https://docs.google.com/document/d/1qeP6DUS8Djq_A0HMllMqsSqX3a9dbcx1/edit?usp=sharing&ouid=110386251748498665069&rtpof=true&sd=true)
for more information.

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.0      ✔ purrr   1.0.1 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.5.0 
    ## ✔ readr   2.1.3      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(stringr)
library(GGally)
```

    ## Registered S3 method overwritten by 'GGally':
    ##   method from   
    ##   +.gg   ggplot2

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
filename <- "./data/ACSDT5Y2018.B01003-Data.csv"

## Load the data
df_pop <- read_csv(filename, skip = 1,
                   col_select = c(id = "Geography",
                                  county_name = "Geographic Area Name",
                                  population = "Estimate!!Total",
                                  error = "Margin of Error!!Total"),
                   col_types = "ccdd", na = "*****") %>% 
  slice(-1)
```

    ## New names:
    ## • `` -> `...7`

``` r
df_pop
```

    ## # A tibble: 3,220 × 4
    ##    id             county_name              population error
    ##    <chr>          <chr>                         <dbl> <dbl>
    ##  1 0500000US01001 Autauga County, Alabama       55200    NA
    ##  2 0500000US01003 Baldwin County, Alabama      208107    NA
    ##  3 0500000US01005 Barbour County, Alabama       25782    NA
    ##  4 0500000US01007 Bibb County, Alabama          22527    NA
    ##  5 0500000US01009 Blount County, Alabama        57645    NA
    ##  6 0500000US01011 Bullock County, Alabama       10352    NA
    ##  7 0500000US01013 Butler County, Alabama        20025    NA
    ##  8 0500000US01015 Calhoun County, Alabama      115098    NA
    ##  9 0500000US01017 Chambers County, Alabama      33826    NA
    ## 10 0500000US01019 Cherokee County, Alabama      25853    NA
    ## # … with 3,210 more rows

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties-2020.csv"
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 884737 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop %>% glimpse
```

    ## Rows: 3,220
    ## Columns: 4
    ## $ id          <chr> "0500000US01001", "0500000US01003", "0500000US01005", "050…
    ## $ county_name <chr> "Autauga County, Alabama", "Baldwin County, Alabama", "Bar…
    ## $ population  <dbl> 55200, 208107, 25782, 22527, 57645, 10352, 20025, 115098, …
    ## $ error       <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
df_covid %>% glimpse
```

    ## Rows: 884,737
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
df_q3 <- df_pop %>% 
  mutate(
    fips = str_sub(id, -5),
    # subregion = county_name %>% 
    #   str_extract(., "[\\w|\\s]+,") %>% 
    #   str_remove(., " County,") %>% 
    #   str_to_lower(),
    # state = county_name %>% 
    #   str_extract(., ", [\\w|\\s]+") %>% 
    #   str_remove(., ", ") %>% 
    #   str_to_lower(),
    
  )
df_q3 %>% head()
```

    ## # A tibble: 6 × 5
    ##   id             county_name             population error fips 
    ##   <chr>          <chr>                        <dbl> <dbl> <chr>
    ## 1 0500000US01001 Autauga County, Alabama      55200    NA 01001
    ## 2 0500000US01003 Baldwin County, Alabama     208107    NA 01003
    ## 3 0500000US01005 Barbour County, Alabama      25782    NA 01005
    ## 4 0500000US01007 Bibb County, Alabama         22527    NA 01007
    ## 5 0500000US01009 Blount County, Alabama       57645    NA 01009
    ## 6 0500000US01011 Bullock County, Alabama      10352    NA 01011

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(county_name, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
df_q4 <- df_q3 %>% 
  left_join(df_covid, by = "fips")
```

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  mutate(
    region = state %>% str_to_lower(),
    subregion = county %>% str_to_lower()
  ) %>% 
  select(
    date,
    county,
    state,
    region,
    subregion,
    fips,
    cases,
    deaths,
    population
  )
```

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <- df_data %>% 
  mutate(
    cases_per100k = 100000 * cases / population,
    deaths_per100k = 100000 * deaths / population
  )
```

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127) < 1e-3
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- ------------------------- -->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute the mean and standard deviation for `cases_per100k` and `deaths_per100k`.

``` r
df_summative <- df_normalized %>% 
  group_by(date) %>% 
  drop_na(cases_per100k, deaths_per100k) %>% 
  summarize(
    mean_cases_per100k = mean(cases_per100k),
    sd_cases_per100k = sd(cases_per100k),
    mean_deaths_per100k = mean(deaths_per100k),
    sd_deaths_per100k = sd(deaths_per100k)
  )
df_summative %>% slice(300)
```

    ## # A tibble: 1 × 5
    ##   date       mean_cases_per100k sd_cases_per100k mean_deaths_per100k sd_deaths…¹
    ##   <date>                  <dbl>            <dbl>               <dbl>       <dbl>
    ## 1 2020-11-15              3684.            2066.                66.8        65.3
    ## # … with abbreviated variable name ¹​sd_deaths_per100k

*Note*:

- It doesn’t make sense to average cases across different dates for this
  dataset, so mean and standard deviation are calculated for each day
  individually.

### **q7** Find the top 10 counties in terms of `cases_per100k`, and the top 10 in terms of `deaths_per100k`. Report the population of each county along with the per-100,000 counts. Compare the counts against the mean values you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well
sorted_cases <- df_normalized %>% 
  group_by(fips) %>% 
  summarize(max_cases_per100k = max(cases_per100k)) %>% 
  arrange(desc(max_cases_per100k))
top_cases <- sorted_cases %>% 
  slice(1:10)
top_cases
```

    ## # A tibble: 10 × 2
    ##    fips  max_cases_per100k
    ##    <chr>             <dbl>
    ##  1 08025            29254.
    ##  2 47169            22553.
    ##  3 46041            21855.
    ##  4 20137            21491.
    ##  5 46009            21194.
    ##  6 05079            20591.
    ##  7 13053            20535.
    ##  8 46017            19971.
    ##  9 47095            18708.
    ## 10 19021            18337.

``` r
## TASK: Find the top 10 deaths_per100k counties; report populations as well
sorted_deaths <- df_normalized %>% 
  group_by(fips) %>% 
  summarize(max_deaths_per100k = max(deaths_per100k)) %>% 
  arrange(desc(max_deaths_per100k))
top_deaths <- sorted_deaths %>% 
  slice(1:10)
top_deaths
```

    ## # A tibble: 10 × 2
    ##    fips  max_deaths_per100k
    ##    <chr>              <dbl>
    ##  1 20063               764.
    ##  2 46073               739.
    ##  3 38021               644.
    ##  4 46053               619.
    ##  5 46125               593.
    ##  6 38031               578.
    ##  7 55051               577.
    ##  8 46057               567.
    ##  9 51595               558.
    ## 10 13141               551.

**Observations**:

- The counties with the maximum cases per 100k have about 10x more cases
  than the average county.
- The counties with the maximum deaths per 100k have more than 10x more
  deaths than the average county.

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

``` r
df_normalized %>%
  semi_join(top_cases, by = "fips") %>% 
  ggplot(aes(x = date, y = cases_per100k, color = fct_reorder2(county, date, cases_per100k))) +
  scale_color_discrete(name = "County") +
  geom_line()
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
sorted_cases %>%
  inner_join(
    sorted_deaths,
    by = "fips"
  ) %>% 
  left_join(df_q3, by = "fips") %>% 
  ggplot(aes(x = max_deaths_per100k, y = max_cases_per100k, color = population)) +
  scale_color_continuous(type = "viridis", trans = "log") +
  geom_point() +
  geom_smooth() +
  scale_x_continuous(trans = "log") +
  scale_y_continuous(trans = "log")
```

    ## Warning: Transformation introduced infinite values in continuous x-axis
    ## Transformation introduced infinite values in continuous x-axis

    ## `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 176 rows containing non-finite values (`stat_smooth()`).

    ## Warning: The following aesthetics were dropped during statistical transformation: colour
    ## ℹ This can happen when ggplot fails to infer the correct grouping structure in
    ##   the data.
    ## ℹ Did you forget to specify a `group` aesthetic or to convert a numerical
    ##   variable into a factor?

    ## Warning: Removed 87 rows containing missing values (`geom_point()`).

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
sorted_deaths %>%
  left_join(df_q3, by = "fips") %>% 
  ggplot(aes(x = max_deaths_per100k)) +
  scale_color_continuous(type = "viridis", trans = "log") +
  geom_histogram() +
  scale_x_continuous(trans = "log")
```

    ## Warning: Transformation introduced infinite values in continuous x-axis

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 176 rows containing non-finite values (`stat_bin()`).

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
sorted_cases %>%
  left_join(df_q3, by = "fips") %>% 
  ggplot(aes(x = max_cases_per100k)) +
  scale_color_continuous(type = "viridis", trans = "log") +
  geom_histogram() +
  scale_x_continuous(trans = "log")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 9 rows containing non-finite values (`stat_bin()`).

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
us_counties <- map_data("county")
sorted_deaths %>%
  left_join(df_q3, by = "fips") %>% 
  ggplot(aes(x = max_deaths_per100k)) +
  scale_color_continuous(type = "viridis", trans = "log") +
  geom_histogram() +
  scale_x_continuous(trans = "log")
```

    ## Warning: Transformation introduced infinite values in continuous x-axis

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 176 rows containing non-finite values (`stat_bin()`).

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
cases_and_deaths <- sorted_cases %>%
  inner_join(
    sorted_deaths,
    by = "fips"
  ) %>% 
  left_join(df_q3, by = "fips") %>%
  mutate(
    subregion = county_name %>%
      str_extract(., "[\\w|\\s]+,") %>%
      str_remove(., " County,") %>%
      str_remove(., " Parish,") %>%
      str_remove(., ",") %>%
      str_to_lower(),
    region = county_name %>%
      str_extract(., ", [\\w|\\s]+") %>%
      str_remove(., ", ") %>%
      str_to_lower()
  )
mapped_cases_and_deaths <- cases_and_deaths %>% 
  right_join(
    us_counties,
    by = c("subregion","region")
  )
```

``` r
mapped_cases_and_deaths %>% 
  group_by(subregion, region, fips)
```

    ## # A tibble: 87,949 × 13
    ## # Groups:   subregion, region, fips [3,076]
    ##    fips  max_ca…¹ max_d…² id    count…³ popul…⁴ error subre…⁵ region  long   lat
    ##    <chr>    <dbl>   <dbl> <chr> <chr>     <dbl> <dbl> <chr>   <chr>  <dbl> <dbl>
    ##  1 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.1
    ##  2 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.5
    ##  3 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.5
    ##  4 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.5
    ##  5 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.5
    ##  6 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.3
    ##  7 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.3
    ##  8 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.2
    ##  9 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.2
    ## 10 08025   29254.    213. 0500… Crowle…    5630    NA crowley color… -104.  38.1
    ## # … with 87,939 more rows, 2 more variables: group <dbl>, order <int>, and
    ## #   abbreviated variable names ¹​max_cases_per100k, ²​max_deaths_per100k,
    ## #   ³​county_name, ⁴​population, ⁵​subregion

``` r
summative_cases_and_deaths <- mapped_cases_and_deaths %>% 
  group_by(subregion, region, fips) %>% 
  summarise(
    county_pop = mean(population),
    county_max_cases_per100k = max(max_cases_per100k),
    county_max_deaths_per100k = max(max_deaths_per100k)
  ) %>%
  ungroup() %>% 
  # left_join(
  #   mapped_cases_and_deaths,
  #   by = "fips",
  # ) %>% 
  left_join(
    us_counties,
    by = c("subregion","region")
  )
```

    ## `summarise()` has grouped output by 'subregion', 'region'. You can override
    ## using the `.groups` argument.

``` r
# sanity checking that the population aggregation is
# roughly similar to the actual USA population of
# 330 million
# assertthat::assert_that(
#   between(
#     sum(summative_cases_and_deaths$county_pop),
#     250000000,
#     400000000
#   )
# )

facetable_cases_pop_deaths <- summative_cases_and_deaths %>% 
  pivot_longer(
    cols = starts_with("county_"),
    names_to = "metric",
    values_to = "people"
  )
facetable_cases_pop_deaths
```

    ## # A tibble: 263,847 × 9
    ##    subregion region         fips   long   lat group order metric          people
    ##    <chr>     <chr>          <chr> <dbl> <dbl> <dbl> <int> <chr>            <dbl>
    ##  1 abbeville south carolina 45001 -82.2  34.4  2285 67181 county_pop      24657 
    ##  2 abbeville south carolina 45001 -82.2  34.4  2285 67181 county_max_cas…  5171.
    ##  3 abbeville south carolina 45001 -82.2  34.4  2285 67181 county_max_dea…   101.
    ##  4 abbeville south carolina 45001 -82.3  34.4  2285 67182 county_pop      24657 
    ##  5 abbeville south carolina 45001 -82.3  34.4  2285 67182 county_max_cas…  5171.
    ##  6 abbeville south carolina 45001 -82.3  34.4  2285 67182 county_max_dea…   101.
    ##  7 abbeville south carolina 45001 -82.3  34.3  2285 67183 county_pop      24657 
    ##  8 abbeville south carolina 45001 -82.3  34.3  2285 67183 county_max_cas…  5171.
    ##  9 abbeville south carolina 45001 -82.3  34.3  2285 67183 county_max_dea…   101.
    ## 10 abbeville south carolina 45001 -82.3  34.3  2285 67184 county_pop      24657 
    ## # … with 263,837 more rows

``` r
theme_maps <- theme(legend.position="right",
    axis.line=element_blank(),
    axis.text=element_blank(),
    axis.ticks=element_blank(),
    axis.title=element_blank(),
    panel.background=element_blank(),
    panel.border=element_blank(),
    panel.grid=element_blank()
  )
```

``` r
facetable_cases_pop_deaths %>% 
  ggplot(aes(x = long, y = lat, group = group, fill = people)) +
  geom_polygon(color = "gray90", size = 0.1) +
  coord_map(projection = "albers", lat0 = 39, lat1 = 45) +
  scale_fill_continuous(type = "viridis", trans = "log") +
  theme_maps +
  facet_wrap(~metric, ncol = 1)
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.

    ## Warning: Transformation introduced infinite values in discrete y-axis

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
summative_cases_and_deaths %>% 
  ggplot(aes(x = long, y = lat, group = group, fill = county_pop,
             label)) +
  geom_polygon(color = "gray90", size = 0.1) +
  coord_map(projection = "albers", lat0 = 39, lat1 = 45) +
  scale_fill_continuous(type = "viridis", trans = "log", name = "Population") +
  theme_maps
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

``` r
summative_cases_and_deaths %>% 
  ggpairs(
    # aes(size = .0000001),
    columns = 4:6,
    columnLabels = c("Population", "Cases", "Deaths")
  ) + 
  scale_x_continuous(n.breaks = 3, trans = "log") +
  scale_y_continuous(n.breaks = 4, trans = "log") +
  
  theme_gray()
```

    ## Warning in ggally_statistic(data = data, mapping = mapping, na.rm = na.rm, :
    ## Removed 1807 rows containing missing values

    ## Scale for x is already present.
    ## Adding another scale for x, which will replace the existing scale.

    ## Warning in ggally_statistic(data = data, mapping = mapping, na.rm = na.rm, :
    ## Removed 1807 rows containing missing values

    ## Scale for x is already present.
    ## Adding another scale for x, which will replace the existing scale.

    ## Warning in ggally_statistic(data = data, mapping = mapping, na.rm = na.rm, :
    ## Removed 1807 rows containing missing values

    ## Scale for x is already present.
    ## Adding another scale for x, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.
    ## Scale for y is already present.
    ## Adding another scale for y, which will replace the existing scale.

    ## Warning: Removed 1703 rows containing non-finite values (`stat_density()`).

    ## Warning: Removed 1807 rows containing missing values (`geom_point()`).

    ## Warning: Removed 1807 rows containing non-finite values (`stat_density()`).

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 1807 rows containing missing values (`geom_point()`).

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 1807 rows containing missing values (`geom_point()`).

    ## Warning: Transformation introduced infinite values in continuous x-axis

    ## Warning: Removed 3552 rows containing non-finite values (`stat_density()`).

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Part of this data sifting was figuring out how to parse county data from
Louisiana, which often would show up as blank because it has parishes
instead. To learn more about the data at hand, I filtered all Louisiana
“county” names and checked them here so I could update the parser in the
previous merge.

``` r
cases_and_deaths %>% 
  filter(
    region == "louisiana"
  ) %>%
  distinct(subregion)
```

    ## # A tibble: 64 × 1
    ##    subregion     
    ##    <chr>         
    ##  1 east carroll  
    ##  2 east feliciana
    ##  3 madison       
    ##  4 franklin      
    ##  5 allen         
    ##  6 caldwell      
    ##  7 ouachita      
    ##  8 bienville     
    ##  9 jackson       
    ## 10 pointe coupee 
    ## # … with 54 more rows

``` r
us_counties %>% 
  filter(
    region == "louisiana"
  ) %>%
  distinct(subregion)
```

    ##              subregion
    ## 1               acadia
    ## 2                allen
    ## 3            ascension
    ## 4           assumption
    ## 5            avoyelles
    ## 6           beauregard
    ## 7            bienville
    ## 8              bossier
    ## 9                caddo
    ## 10           calcasieu
    ## 11            caldwell
    ## 12             cameron
    ## 13           catahoula
    ## 14           claiborne
    ## 15           concordia
    ## 16             de soto
    ## 17    east baton rouge
    ## 18        east carroll
    ## 19      east feliciana
    ## 20          evangeline
    ## 21            franklin
    ## 22               grant
    ## 23              iberia
    ## 24           iberville
    ## 25             jackson
    ## 26           jefferson
    ## 27     jefferson davis
    ## 28           lafayette
    ## 29           lafourche
    ## 30            la salle
    ## 31             lincoln
    ## 32          livingston
    ## 33             madison
    ## 34           morehouse
    ## 35        natchitoches
    ## 36             orleans
    ## 37            ouachita
    ## 38         plaquemines
    ## 39       pointe coupee
    ## 40             rapides
    ## 41           red river
    ## 42            richland
    ## 43              sabine
    ## 44          st bernard
    ## 45          st charles
    ## 46           st helena
    ## 47            st james
    ## 48 st john the baptist
    ## 49           st landry
    ## 50           st martin
    ## 51             st mary
    ## 52          st tammany
    ## 53          tangipahoa
    ## 54              tensas
    ## 55          terrebonne
    ## 56               union
    ## 57           vermilion
    ## 58              vernon
    ## 59          washington
    ## 60             webster
    ## 61    west baton rouge
    ## 62        west carroll
    ## 63      west feliciana
    ## 64                winn

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

``` r
## NOTE: No need to change this; just an example
df_normalized %>%
  filter(state == "Massachusetts") %>%

  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line() +
  scale_y_log10(labels = scales::label_number_si()) +
  scale_color_discrete(name = "County") +
  theme_minimal() +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)"
  )
```

    ## Warning: `label_number_si()` was deprecated in scales 1.2.0.
    ## ℹ Please use the `scale_cut` argument of `label_number()` instead.

![](c06-covid19-assignment_files/figure-gfm/ma-example-1.png)<!-- -->

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
