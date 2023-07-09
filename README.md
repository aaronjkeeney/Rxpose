RXposé
================
Aaron
2023-07-09

## Rxposé: An Analysis of Performance-Enhancing Drug Use in the Sport of Weightlifting

Note : This portfolio project presents an opportunity to practice SQL
skills, as well as learning how to link a database and run multiple
languages within an Rmarkdown. There are places where it may be simpler
to use only R, but it is a valuable learning experience, regardless.

### Background

Weightlifting has been one of the sports with the highest (recorded)
prevalence of drug use. Rumors are always circling, and considerable
evidence has shown that covert as well as state-sponsored doping rings
are commonplace. Many people believe that the number of “clean” athletes
at the elite level is extremely low, possibly as low as zero.

While the analysis of the effectiveness of individual tests is
fascinating and important, the initial scope of this project is much
broader. The question at the center of this study is:

Has drug testing made an impact on the performance of athletes in the
sport?

To answer this we can compare two populations, athletes who have tested
positive for performance-enhancing drugs (PEDs) against those who have
not. This tests a central assumption about drug testing, i.e. PEDs
increase performance. If the positive-testing athletes (sometimes
referred to as “popped”) do not outperform the negative-testing
athletes, the truth of this assumptions falls must be questioned.

In order to compare all athlete, the [Sinclair
Coefficient](https://iwf.sport/weightlifting_/sinclair-coefficient/)
will be employed. This will allow us to compare performaces of athletes
across weight classes and get a better understaning of the influence of
drug testing on the sport as a whole.

We should note that the subjectivity of “talent” will always be part of
this discussion. This conversation is inextricably linked with
drug-testing, as genetics affect athletic performance, drug response,
and even our ability to screen for drug use.

Additional topics for study

- Prevalence of use by country, region, gender, etc.

- Life-time competition bans vs. temporary bans

- When in the athlete’s career a ban occurred.

## Data

One of the advantages of studying sports is the prevalence of well-kept
records. Weightlifting in particular is convenient due the the simple
measurement of performance– more weight lifted (within a weight class)
is a proportionally better performance.

For Olympic competitions, [this
dataset](https://www.kaggle.com/datasets/piterfm/olympic-games-medals-19862018)
was immensely helpful in providing performance results. A huge thank-you
to Petro for providing it, and keeping it current.

At least for now, we will restrict our analysis to Olympic competition.
While this will select for the highest-performing athletes, the Olympics
are highly visible and widely regarded. It makes sense from a public
perception standpoint to focus here.

``` r
## Packages for this analysis
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(RSQLite)
library(DBI)
library(dbplyr)
```

    ## 
    ## Attaching package: 'dbplyr'
    ## 
    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     ident, sql

``` r
getwd()
```

    ## [1] "/Users/aaronkeeney/Documents/Data Analytics Projects/Rxpose"

``` r
setwd("~/Documents/Data Analytics Projects/Rxpose/Kaggle Olympics Data")
olympics_data_raw <- read_csv("olympic_results.csv")
```

    ## Rows: 162804 Columns: 15
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (14): discipline_title, event_title, slug_game, participant_type, medal_...
    ## lgl  (1): rank_equal
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
con <- DBI::dbConnect(RSQLite::SQLite(), 
        dbname = "olympics_db.sqlite")

dbWriteTable(con, 
             "all_olympics_results", 
             olympics_data_raw, 
             overwrite = TRUE)
```

First, I sorted data in BigQuery, as it was much easier to visualize
there. Only weightlifting data was extracted, and only from 1976 and
later. This is due to two reasons. First, there was a [rule
shift](https://en.wikipedia.org/wiki/Clean_and_press) after 1972 that
greatly affected the total weight lifted. Second, anabolic steroids were
not banned until 1975.
[source](https://www.acog.org/clinical/clinical-guidance/committee-opinion/articles/2011/04/performance-enhancing-anabolic-steroid-abuse-in-women#:~:text=Anabolic%20steroids%20were%20first%20discovered,performance%20and%20enhance%20cosmetic%20appearance.)

Below is the code that was run in SQLite within the markdown file. This
filters for only weightlifting results from the correct years, and it
extracts and casts the competition year and the weight lifted as more
usable data types.

``` sql
CREATE TABLE IF NOT EXISTS weightlifting_results AS
SELECT event_title, rank_position, country_3_letter_code, athlete_full_name,
  value_type,
  CAST(value_unit AS decimal) AS total_kg,
  CAST(SUBSTR(slug_game, -4) AS INT) AS year
FROM
  all_olympics_results
WHERE
  discipline_title = "Weightlifting" 
  AND
  value_type IS NOT NULL 
  AND
  year > 1972

-- Why can I not use "year" as a variable here? Add it to the table with a join?
-- Works here, just not in BigQuery
-- CAST(SUBSTR(slug_game, 4) AS INT) >1972 for use in BigQuery
```

At this point, we need to clean the disastrous formatting that is the
event_title column. The strings are not consistent, and we need to
extract a usable number from them. Weight classes are determined by the
maximum allowable weight in each category, so we will will need to
extract the largest number present in each string. Usually, there is
only one number, but this column sometimes gives a range of maximum and
minimum weights when we only want the larger value. From the table
below, we can see that there are even some misprints in the numbers
themselves (e.g. “825” kg should be “82.5”)

``` sql
SELECT DISTINCT event_title
FROM weightlifting_results
```

<div class="knitsql-table">

| event_title   |
|:--------------|
| Men’s 61kg    |
| Women’s 55kg  |
| Men’s 67kg    |
| Men’s 81kg    |
| Women’s +87kg |
| Women’s 87kg  |
| Men’s +109kg  |
| Women’s 59kg  |
| Women’s 64kg  |
| Women’s 49kg  |

Displaying records 1 - 10

</div>

#### Separating Sex and Weight Class

First, we will add columns to designate men’s and women’s events, as
well as for weightclass.

``` sql
ALTER TABLE weightlifting_results 
  ADD sex NOT NULL default 'Men'
```

Note: this is not to say that men’s sports are the default, but it is
easier to use REGEX to select for the string “women” rather than “men.”

``` sql
ALTER TABLE weightlifting_results 
  ADD weight_class_kg NOT NULL default 0
```

``` sql
UPDATE weightlifting_results
  SET sex = CASE 
    WHEN (event_title LIKE '%women%') THEN 'F'
    ELSE 'M' END
```

``` sql
SELECT *
FROM weightlifting_results
LIMIT 10
```

<div class="knitsql-table">

| event_title | rank_position | country_3_letter_code | athlete_full_name             | value_type | total_kg | year | sex | weight_class_kg |
|:------------|:--------------|:----------------------|:------------------------------|:-----------|---------:|-----:|:----|----------------:|
| Men’s 61kg  | 4             | JPN                   | Yoichi ITOKAZU                | WEIGHT     |      292 | 2020 | M   |               0 |
| Men’s 61kg  | 12            | PER                   | Marcos Antonio ROJAS CONCHA   | WEIGHT     |      240 | 2020 | M   |               0 |
| Men’s 61kg  | 6             | ITA                   | Davide RUIU                   | WEIGHT     |      286 | 2020 | M   |               0 |
| Men’s 61kg  | 3             | KAZ                   | Igor SON                      | WEIGHT     |      294 | 2020 | M   |               0 |
| Men’s 61kg  | 9             | GER                   | Simon Josef BRANDHUBER        | WEIGHT     |      268 | 2020 | M   |               0 |
| Men’s 61kg  | 2             | INA                   | Eko Yuli IRAWAN               | WEIGHT     |      302 | 2020 | M   |               0 |
| Men’s 61kg  | 7             | GEO                   | Shota MISHVELIDZE             | WEIGHT     |      285 | 2020 | M   |               0 |
| Men’s 61kg  | 8             | DOM                   | Luis Alberto GARCIA BRITO     | WEIGHT     |      274 | 2020 | M   |               0 |
| Men’s 61kg  | 11            | MAD                   | Eric Herman ANDRIANTSITOHAINA | WEIGHT     |      264 | 2020 | M   |               0 |
| Men’s 61kg  | DNF           | TPE                   | Chan-Hung KAO                 | IRM        |       NA | 2020 | M   |               0 |

Displaying records 1 - 10

</div>

``` sql
SELECT DISTINCT event_title, sex, weight_class_kg
FROM weightlifting_results
```

<div class="knitsql-table">

| event_title   | sex | weight_class_kg |
|:--------------|:----|----------------:|
| Men’s 61kg    | M   |               0 |
| Women’s 55kg  | F   |               0 |
| Men’s 67kg    | M   |               0 |
| Men’s 81kg    | M   |               0 |
| Women’s +87kg | F   |               0 |
| Women’s 87kg  | F   |               0 |
| Men’s +109kg  | M   |               0 |
| Women’s 59kg  | F   |               0 |
| Women’s 64kg  | F   |               0 |
| Women’s 49kg  | F   |               0 |

Displaying records 1 - 10

</div>

#### Sinclair considerations

Sinclair calculations are not perfect, as different weight classes are
essentially playing different games. For example, the goal of a
superheavyweight lifter is to get as big and strong as possible to lift
maximum weight, while lighter lifters need to get as big and strong as
possible within a specified boundary. Obviously, different body types
will self-select into their most competitive classes over the course of
time. For this study, the maximum allowable weight for each category
will be used. For non-superheavy weight athletes, this approximation is
very good, as it is in their best interest to be as heavy (read
muscular, strong) as possible while remaining in their category. The
Sinclair method includes a “maximum weight” that is used for these
corrections, and that is the bodyweight used in this study for all
supers. While this may lower the Sinclair totals for superheavyweight
athletes, since this study is not comparing individual lifters, this is
an acceptable apporoximation. As a second approximation, the calculated
values for the current Olympic cycle will be used. This will greatly
simplify calcuations, and it will allow modern scaling of historical
athletes while crediting them with their bodyweight at the time.

We need to use REGEX to extract the correct numeric portions of
event_title for weight_class_kg. As mentioned above superheavyweights
will be credited with the maximum weight. SQLITE DOES NOT SUPPORT REGEX,
SO I MIGHT NEED TO EXPLORE BIGRQUERY OR ANOTHER OPTION.

``` sql
UPDATE weightlifting_results
  SET weight_class_kg = 
    CASE 
      WHEN (sex = 'M' AND event_title LIKE '%+%') THEN 1000
      WHEN (sex = 'M' AND event_title LIKE '%super%') THEN 1000
      WHEN (sex = 'F' AND event_title LIKE '%+%') THEN 800
      WHEN (sex = 'F' AND event_title LIKE '%super%') THEN 800
      ELSE 500
  END
     
--  (REGEXP(([\d.+])+(?=kg)) should be the correct REGEX expression
-- need both "super" and "+" to be accounted for
-- [WHEN (event_title LIKE '%+%') THEN 1000 ELSE (1)]
-- couldn't get the OR function to work inside LIKE function
        
```

## This was method number 1–still figuring the best way to proceed

To begin the cleaning process, I created a new table with the best
performances by each athlete. (This may be an issue, as it might
eliminate performances of athletes that rank better by Sinclair if they
moved up a category and underperformed.)

``` r
##SELECT MAX(total_kg) AS best_total_kg, athlete_full_name,
##FROM `weightliftinganalysis.Olympics_Results_86_to_20.All_weightlifting_performances_##      post_1976`
##WHERE total_kg IS NOT NULL
##GROUP BY
##  athlete_full_name
```

Then, I joined this table back to the original to have the full table
with only the best performance of each athlete. A big thank you to Bill
Karwin for his
[answer](https://stackoverflow.com/questions/121387/fetch-the-rows-which-have-the-max-value-for-a-column-for-each-distinct-value-of)
on how to do this efficiently.

``` r
##SELECT *
##FROM`weightliftinganalysis.Olympics_Results_86_to_20.All_weightlifting_performances_p## ost_1976`
##LEFT OUTER JOIN
##  `weightliftinganalysis.Olympics_Results_86_to_20.Only_nonzero_performances_post_1976`
##  ON (`weightliftinganalysis.Olympics_Results_86_to_20.Only_nonzero_performances_post_1976`.athlete_full_name = `weightliftinganalysis.Olympics_Results_86_to_20.All_weightlifting_performances_post_1976`.athlete_full_name
##    AND
##    `weightliftinganalysis.Olympics_Results_86_to_20.Only_nonzero_performances_post_1976`.best_total_kg >
##    `weightliftinganalysis.Olympics_Results_86_to_20.All_weightlifting_performances_post_1976`.total_kg)
##  WHERE
##    total_kg IS NOT NULL AND
##    best_total_kg IS NULL
```

This almost worked perfectly, but it still will repeat athletes if they
had exactly the same totals in multiple competitions. This is an easier
problem to solve in a spreadsheet, so at this point I moved the .csv
file to that software. With less than 2000 rows, I am not concerned
about performance.

### Spreadsheet cleaning and analysis

#### [This link](https://docs.google.com/spreadsheets/d/1vWlYv58PvmJE1VBEh5-dOVboG6D-HxOqWq_zTylOWC0/edit?usp=sharing) will show the spreadsheet in its current form.

As mentioned before, there are a few names which repeat, but only 14.
These can be sorted out without too much difficulty. In fact, it might
make sense to leave them in and credit the athlete with the larger
Sinclair.

The larger problem is that of weight classes. Labelled “event_title” in
this dataset, there is little to no consistency in how these data were
entered. It is necessary to have a consistent format, as weight class
and year will determine Sinclair calculations.
