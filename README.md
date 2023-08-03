RXposé
================
Aaron
2023-08-03

## Rxposé: An Analysis of Performance-Enhancing Drug Use in the Sport of Weightlifting

Note : This portfolio project presents an opportunity to practice SQL
skills, as well as learning how to link a database and run multiple
languages within an Rmarkdown. There are places where it may be simpler
to use only R, but it is a valuable learning experience, regardless.

## Background

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

A quick primer on the sport of weightlifting: as sports go, it is
relatively simple. Whoever lifts the most weight wins. Athletes are
separated by assigned sex (male/female) and weight class. Weight class
is a way of separating body types, as being larger and having more
muscle allows people to typically lift more weight. Usually, the
athletes are split into weight classes that are about 7kg apart. This
means that all the athletes competing against one another weigh within
7kg (15 lbs) of each other. This allows broader participation in the
sport, since it allows people who are not naturally large to compete.
The standard modern convention is to identify a weight class by the
maximum allowable bodyweight in that weight class. For example, the M
102kg category means that any man weighing less than 102kg is eligible
to compete. (Note: athletes may always compete in a heavier category
than their bodyweight, but it is almost never advantageous to do so, as
the other competitors will be larger and stronger.)

In order to compare all athlete, the [Sinclair
Coefficient](https://iwf.sport/weightlifting_/sinclair-coefficient/)
will be employed. This will allow us to compare performances of athletes
across weight classes and get a better understanding of the influence of
drug testing on the sport as a whole.

We should note that the subjectivity of “talent” will always be part of
this discussion. This conversation is inextricably linked with
drug-testing, as genetics affect athletic performance, drug response,
and even our ability to screen for drug use.

Additional topics for potential future study:

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

First, the data is loaded into a SQL databsae using RSQLite. There are
limitaions with this package, but it is a good opportunity to practice
queries.

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

Only weightlifting data was extracted for analyis, and only from 1976
and later. This is due to two reasons. First, there was a [rule
shift](https://en.wikipedia.org/wiki/Clean_and_press) after 1972 that
greatly affected the total weight lifted. Second, anabolic steroids were
not banned until 1975.
[source](https://www.acog.org/clinical/clinical-guidance/committee-opinion/articles/2011/04/performance-enhancing-anabolic-steroid-abuse-in-women#:~:text=Anabolic%20steroids%20were%20first%20discovered,performance%20and%20enhance%20cosmetic%20appearance.)

Below is the code that was run in SQLite within the markdown file. This
filters for only weightlifting results from the correct years, and it
extracts and casts the competition year and the weight lifted as more
usable data types. It also removes any null or 0 values from the
total_kg column, as those values will not be useful for comparison.

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
  total_kg IS NOT NULL
  AND
  total_kg >0
  AND
  year > 1972

-- Why can I not use "year" as a variable here? Add it to the table with a join?
-- Works here, just not in BigQuery
-- CAST(SUBSTR(slug_game, 4) AS INT) >1972 for use in BigQuery
```

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
UPDATE weightlifting_results
  SET sex = CASE 
    WHEN (event_title LIKE '%women%') THEN 'F'
    ELSE 'M' END
```

``` sql
-- just a check for the sex column
SELECT DISTINCT sex, country_3_letter_code 
FROM weightlifting_results
LIMIT 20
```

At this point, we need to clean the disastrous formatting that is the
event_title column. The strings are not consistent, and we need to
extract a usable number from them. Weight classes are determined by the
maximum allowable weight in each category, so we will will need to
extract the largest number present in each string. Usually, there is
only one number, but this column sometimes gives a range of maximum and
minimum weights when we only want the larger value. From the table
below, we can see that there are even some misprints in the numbers
themselves (e.g. “825” kg should be “82.5”).

This step of cleaning required r, as SQLite does not support REGEX. Data
were queried, cleaned, then reuploaded to replace the original table.

``` r
weightlifting_results <- 
  dbGetQuery(con, "SELECT * FROM weightlifting_results")


weightlifting_results_after_regex_extraction <-
weightlifting_results %>%
  mutate(weight_class_kg = as.numeric(str_extract(event_title, "([0-9.])+(?=kg)")
  ))

unique(sort(weightlifting_results_after_regex_extraction$weight_class_kg)) ## check for necessary changes
```

    ##  [1]  48.0  49.0  52.0  53.0  54.0  55.0  56.0  58.0  59.0  60.0  61.0  62.0
    ## [13]  63.0  64.0  67.0  67.5  69.0  70.0  75.0  76.0  77.0  81.0  83.0  85.0
    ## [25]  87.0  90.0  91.0  94.0  99.0 100.0 105.0 108.0 109.0 110.0 825.0

``` r
## Since 825kg is a nonsensical number in weightlifting, we can universally replace 825 with 82.5.

weightlifting_results_after_regex_extraction[weightlifting_results_after_regex_extraction == 825] <- 82.5
unique(sort(weightlifting_results_after_regex_extraction$weight_class_kg))
```

    ##  [1]  48.0  49.0  52.0  53.0  54.0  55.0  56.0  58.0  59.0  60.0  61.0  62.0
    ## [13]  63.0  64.0  67.0  67.5  69.0  70.0  75.0  76.0  77.0  81.0  82.5  83.0
    ## [25]  85.0  87.0  90.0  91.0  94.0  99.0 100.0 105.0 108.0 109.0 110.0

This code conflates the weight classes for the heaviest and
second-heaviest categoies, but that will be sorted out with SQL. This
resulting dataframe can be uploaded to our database.

``` r
dbWriteTable(con, 
             "weightlifting_results", 
             weightlifting_results_after_regex_extraction, 
             overwrite = TRUE)
```

``` sql
-- test to make sure the new table exists and has correct data
SELECT *
FROM weightlifting_results
LIMIT
20
```

<div class="knitsql-table">

| event_title | rank_position | country_3_letter_code | athlete_full_name             | value_type | total_kg | year | sex | weight_class_kg |
|:------------|:--------------|:----------------------|:------------------------------|:-----------|---------:|-----:|:----|----------------:|
| Men’s 61kg  | 4             | JPN                   | Yoichi ITOKAZU                | WEIGHT     |      292 | 2020 | M   |              61 |
| Men’s 61kg  | 12            | PER                   | Marcos Antonio ROJAS CONCHA   | WEIGHT     |      240 | 2020 | M   |              61 |
| Men’s 61kg  | 6             | ITA                   | Davide RUIU                   | WEIGHT     |      286 | 2020 | M   |              61 |
| Men’s 61kg  | 3             | KAZ                   | Igor SON                      | WEIGHT     |      294 | 2020 | M   |              61 |
| Men’s 61kg  | 9             | GER                   | Simon Josef BRANDHUBER        | WEIGHT     |      268 | 2020 | M   |              61 |
| Men’s 61kg  | 2             | INA                   | Eko Yuli IRAWAN               | WEIGHT     |      302 | 2020 | M   |              61 |
| Men’s 61kg  | 7             | GEO                   | Shota MISHVELIDZE             | WEIGHT     |      285 | 2020 | M   |              61 |
| Men’s 61kg  | 8             | DOM                   | Luis Alberto GARCIA BRITO     | WEIGHT     |      274 | 2020 | M   |              61 |
| Men’s 61kg  | 11            | MAD                   | Eric Herman ANDRIANTSITOHAINA | WEIGHT     |      264 | 2020 | M   |              61 |
| Men’s 61kg  | 10            | PNG                   | Morea BARU                    | WEIGHT     |      265 | 2020 | M   |              61 |

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

For this analysis, credited bodyweights for male and female
superheavyweights were taken from the [IWF
website](https://iwf.sport/wp-content/uploads/downloads/2023/05/2021-Sinclair_Coefficients.pdf).

``` sql
UPDATE weightlifting_results
  SET weight_class_kg = 
    CASE 
      WHEN (sex = 'M' AND event_title LIKE '%+%') THEN 193.609
      WHEN (sex = 'M' AND event_title LIKE '%super%') THEN 193.609
      WHEN (sex = 'F' AND event_title LIKE '%+%') THEN 153.757
      WHEN (sex = 'F' AND event_title LIKE '%super%') THEN 153.757
      ELSE weight_class_kg
  END
  
-- need both "super" and "+" to be accounted for
-- [WHEN (event_title LIKE '%+%') THEN 1000 ELSE (1)]
-- couldn't get the OR function to work inside LIKE function
        
```

``` sql
SELECT DISTINCT weight_class_kg, event_title
FROM weightlifting_results
ORDER BY weight_class_kg DESC
```

<div class="knitsql-table">

| weight_class_kg | event_title                 |
|----------------:|:----------------------------|
|         193.609 | Men’s +109kg                |
|         193.609 | +105kg men                  |
|         193.609 | 105kg superheavyweight men  |
|         193.609 | 108kg super heavyweight men |
|         193.609 | 110kg super heavyweight men |
|         153.757 | Women’s +87kg               |
|         153.757 | +75kg women                 |
|         110.000 | 100 110kg heavyweight men   |
|         110.000 | 91 110kg heavyweight men    |
|         109.000 | Men’s 109kg                 |

Displaying records 1 - 10

</div>

#### Adding Sinclair to the table

Finally, we get to add the primary value for analysis to this table.

``` sql
ALTER TABLE weightlifting_results 
  ADD sinclair NOT NULL default 0
```

``` sql
UPDATE weightlifting_results
  SET sinclair = CASE
    WHEN (sex = 'F')
    THEN  total_kg* POWER(10, 0.787004341*POWER(LOG10(weight_class_kg/153.757),2))
    ELSE  total_kg* POWER(10, 0.722762521*POWER(LOG10(weight_class_kg/193.609),2))
    END
```

``` sql
SELECT DISTINCT weight_class_kg, sinclair, athlete_full_name
FROM weightlifting_results
WHERE sex = 'F'
ORDER BY sinclair DESC
LIMIT 30
```

<div class="knitsql-table">

| weight_class_kg | sinclair | athlete_full_name |
|----------------:|---------:|:------------------|
|          63.000 | 343.9311 | Wei DENG          |
|          69.000 | 342.4793 | Chunhong LIU      |
|          58.000 | 340.4270 | Xueying LI        |
|          48.000 | 333.7315 | Nurcan TAYLAN     |
|         153.757 | 333.0000 | Lulu ZHOU         |
|          58.000 | 332.1239 | Sukanya SRISURAT  |
|         153.757 | 332.0000 | Tatiana KASHIRINA |
|          53.000 | 331.5665 | Xia YANG          |
|          58.000 | 328.6643 | Yanqing CHEN      |
|          49.000 | 328.3477 | Zhihui HOU        |

Displaying records 1 - 10

</div>

While not the correct Sincalir for each individual athlete, this will
serve well enough for the purposes of this study. This list is
absolutley not intended to rank athletes’ performances. I would like to
apologize especially to Lasha Talakhadze, who as the best
superheavyweight in history appears at \#28 on this ranking. Using
historical Sinclair calculations, he appears at \#2 behind Naim
Süleymanoglu, but I still have faith that Lasha will hit the legendary
500kg total before he retires.

It is interesting to note that two female superheavyweights appear in
the top 10 in this ranking. I will not analyze that too much here, but
there is likely a huge amount of influence on this calcultion by
representation in the sport. The number of male athletes (historically)
is far greater, and it would be wonderful to see how more female
representation would change the sport.

As the last step in data cleaning and preparation, we need to add a
table of lifters who have served drug bans. A list (if not a complete
one) exists at [this
link](https://en.wikipedia.org/wiki/Category:Doping_cases_in_weightlifting).
After [stripping the html](https://www.striphtml.com/), I added the list
to my
[spreadsheet](https://docs.google.com/spreadsheets/d/1vWlYv58PvmJE1VBEh5-dOVboG6D-HxOqWq_zTylOWC0/edit?usp=sharing).

Creating the new table.

``` r
setwd("~/Documents/Data Analytics Projects/Rxpose/Kaggle Olympics Data")
weightlifters_with_doping_violations_raw <- read_csv("weightlifters_with_doping_violations.csv")
```

    ## Rows: 189 Columns: 1
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): weightlifters_with_doping_violations
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
str(weightlifters_with_doping_violations_raw)
```

    ## spc_tbl_ [189 × 1] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ weightlifters_with_doping_violations: chr [1:189] "Khadzhimurat Akkaev" "Henadzi Aliashchuk" "Saeid Alihosseini" "Chika Amalaha" ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   weightlifters_with_doping_violations = col_character()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

This table will also require some in-depth cleaning. The table includes
parentheses (left over from the online source), and there are many
spellings of names with non-English characters. Matching names and
successfully executing a join with the previous table will require
REGEX, so this portion of cleaning will necessitate R.

``` r
## This code removes the parentheses from the strings in the athlete's names and creates a new column for the cleaned names

weightlifters_with_doping_violations_no_parentheses <-
weightlifters_with_doping_violations_raw %>%
    mutate(weightlifters_with_doping_violations_no_parentheses = gsub("\\(.*$","",weightlifters_with_doping_violations_raw$weightlifters_with_doping_violations))
## Note to self on REGEX-- the double backslash is important due to "\" and "(" both being escape operators.


# Creates a table with only one column
weightlifters_with_doping_violations_no_parentheses_v_2 <- weightlifters_with_doping_violations_no_parentheses[2]
```

The final step in analysis is to join the table of weightlifters with
violations with the table of all lifters to separate the two
populations. In the table
weightlifters_with_doping_violations_no_parentheses, there are many
letters considered “special characters” in the English alphabet. There
also appear to be extraneous capital letters, possibly from odd
formatting from other alphabets. For simplicity, we can match special
characters to the English character on which they are based. For
example, “î”,“ï”, “í”, and “ì” would all match “i.” We check the
encoding and get some bad news.

``` r
# Encoding(weightlifters_with_doping_violations_no_parentheses_v_2$weightlifters_with_doping_violations_no_parentheses)

weightlifters_with_doping_violations_reduced <- iconv(weightlifters_with_doping_violations_no_parentheses_v_2$weightlifters_with_doping_violations_no_parentheses,from="UTF-8",to="ASCII//TRANSLIT")

df_weightlifters_with_doping_violations_reduced <- data.frame(weightlifters_with_doping_violations_reduced)
colnames(df_weightlifters_with_doping_violations_reduced)[1] = "athlete_full_name"

df_weightlifters_with_doping_violations_reduced_lower <- data.frame( tolower(df_weightlifters_with_doping_violations_reduced$athlete_full_name))
colnames(df_weightlifters_with_doping_violations_reduced_lower)[1] = "athlete_full_name"

df_weightlifters_with_doping_violations_reduced_lower <- data.frame(gsub("[^[:alnum:\\,]", "", df_weightlifters_with_doping_violations_reduced_lower$athlete_full_name))
colnames(df_weightlifters_with_doping_violations_reduced_lower)[1] = "athlete_full_name"
view(df_weightlifters_with_doping_violations_reduced_lower)

#df_weightlifters_with_doping_violations_reduced <-
#  str_replace_all(df_weightlifters_with_doping_violations_reduced$athlete_full_name,'^a-zA-Z0-\\', '')

# Lots of code below that I don't want to delete yet in case I need it later...
#Encoding(weightlifters_with_doping_violations_reduced)

#weightlifters_with_doping_violations_no_parentheses_v_2$weightlifters_with_doping_violations_no_parentheses = stri_trans_general(str = weightlifters_with_doping_violations_no_parentheses_v_2$weightlifters_with_doping_violations_no_parentheses, id = "Latin-ASCII")
#Encoding(weightlifters_with_doping_violations_no_parentheses_v_2$weightlifters_with_doping_violations_no_parentheses)
#view(weightlifters_with_doping_violations_no_parentheses_v_2)
#library(stringi)
#weightlifters_with_doping_violations_no_parentheses_v_2[, "weightlifters_with_doping_violations_no_parentheses" = stri_trans_general(str = weightlifters_with_doping_violations_no_parentheses_v_2, id = "Latin-ASCII")]
```

After clearing, knitting, and revisiting, this code worked. After some
research, a “fuzzy join” looks promising, so that package is installed.
A join will then separate the positive-testing lifters from the
non-positive-testing lifters. The fuzzyjoin package in R has wonderful
functinoality for Jaccard distance, so this step will be done in R.
Then, two separate tables can be added to the database for each
population of athletes.

``` sql

SELECT * 
FROM weightlifting_results
ORDER BY sinclair DESC
LIMIT 30
```

<div class="knitsql-table">

| event_title                          | rank_position | country_3_letter_code | athlete_full_name   | value_type | total_kg | year | sex | weight_class_kg | sinclair |
|:-------------------------------------|:--------------|:----------------------|:--------------------|:-----------|---------:|-----:|:----|----------------:|---------:|
| 56 - 60kg (featherweight) men        | 1             | TUR                   | Naim Süleymanoğlu   | WEIGHT     |    342.5 | 1988 | M   |            60.0 | 526.9247 |
| 100 110kg heavyweight men            | 1             | URS                   | Yuri ZAKHAREVICH    | WEIGHT     |    455.0 | 1988 | M   |           110.0 | 503.0187 |
| 75 825kg lightheavyweight men        | 1             | URS                   | Yuri Vardanyan      | WEIGHT     |    400.0 | 1980 | M   |            82.5 | 502.6418 |
| 69kg men                             | 1             | BUL                   | Galabin BOEVSKI     | WEIGHT     |    357.5 | 2000 | M   |            69.0 | 499.3291 |
| -56kg (bantamweight) men             | 1             | CHN                   | Qingquan LONG       | WEIGHT     |    307.0 | 2016 | M   |            56.0 | 497.6358 |
| 67.5-75kg middleweight men           | 1             | BUL                   | Borislav GIDIKOV    | WEIGHT     |    375.0 | 1988 | M   |            75.0 | 497.3190 |
| 82.5 - 90kg (middle-heavyweight) men | 1             | EUN                   | Akakios KAKIASVILIS | WEIGHT     |    412.5 | 1992 | M   |            90.0 | 495.9271 |
| 82.5 - 90kg (middle-heavyweight) men | 2             | EUN                   | Serguei SYRTSOV     | WEIGHT     |    412.5 | 1992 | M   |            90.0 | 495.9271 |
| 82.5 - 90kg (middle-heavyweight) men | 1             | URS                   | Anatoli KHRAPATY    | WEIGHT     |    412.5 | 1988 | M   |            90.0 | 495.9271 |
| 77kg men                             | 1             | KAZ                   | Nidzhat Rakhimov    | WEIGHT     |    379.0 | 2016 | M   |            77.0 | 494.9174 |

Displaying records 1 - 10

</div>

``` r
library(fuzzyjoin)
library(stringdist)
```

    ## 
    ## Attaching package: 'stringdist'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     extract

``` r
all_weightlifting_results_for_join <- dbGetQuery(con, "SELECT * FROM weightlifting_results")

all_weightlifting_results_for_join$athlete_full_name <- tolower(all_weightlifting_results_for_join$athlete_full_name)

#all_weightlifting_results_for_join$athlete_full_name <- str_replace_all(all_weightlifting_results_for_join$athlete_full_name, " ","")

all_weightlifting_results_for_join$athlete_full_name <- gsub("[^[:alnum:\\,]", "", all_weightlifting_results_for_join$athlete_full_name)
view(all_weightlifting_results_for_join)

# The code below exhausted the vector memory when I used a fuzzy join. The regular inner join returns some useful data, but it is incomplete.

positive_testing_lifters_exact_matches <- inner_join(df_weightlifters_with_doping_violations_reduced_lower, all_weightlifting_results_for_join, by = "athlete_full_name")
view(positive_testing_lifters_exact_matches)

weightlifting_results_without_exact_matches <- anti_join(all_weightlifting_results_for_join, df_weightlifters_with_doping_violations_reduced_lower, by = "athlete_full_name")
view(weightlifting_results_without_exact_matches)

positive_testing_lifters_without_exact_matches <- anti_join(df_weightlifters_with_doping_violations_reduced_lower, positive_testing_lifters_exact_matches, by = "athlete_full_name")
view(positive_testing_lifters_without_exact_matches)
#positive_testing_lifters_closest_match <- fuzzy_inner_join(df_weightlifters_with_doping_violations_reduced_lower, all_weightlifting_results_for_join, by = "athlete_full_name", match_fun = stringdist::stringdistmatrix, method = "jaccard")

#view(positive_testing_lifters_closest_match)
```

There are 69 exact name matches in the results. We can now run some form
of approximate match function to classify the rest.

``` r
test_join <- stringdist_inner_join(positive_testing_lifters_without_exact_matches, weightlifting_results_without_exact_matches, by = "athlete_full_name", method = "cosine", max_dist = 0.05)
view(test_join)

#repeated_names <- inner_join(test_join, positive_testing_lifters_exact_matches, by = c('athlete_full_name.x' = 'athlete_full_name'))
#view(repeated_names)

#names_from_only_stringdist <- anti_join(test_join, positive_testing_lifters_exact_matches, by = c('athlete_full_name.x' = 'athlete_full_name'))
#view(names_from_only_stringdist)
```

Cosine distances and Jaccard distances give slightly different results,
but with the inexact matching, we are able to add between 10 and 20
other names. It would be good to check to see if the original dataset
includes some athletes who had medals stripped (Klokov is a famous
example). This approximate matching has done a good job of catching
missing characters as well as matching Asian names that are sometimes
transcribed into English in different orders.

``` sql
SELECT athlete_full_name, rank_position, sinclair
FROM weightlifting_results
WHERE athlete_full_name LIKE '%klo%'
```

<div class="knitsql-table">

| athlete_full_name | rank_position | sinclair |
|:------------------|:--------------|:---------|

0 records

</div>

Klokov is missing, which indicates that stripped medals are not part of
this dataset. Those results may be harder to find, but they will be
important to add in future iterations of the project.

## Analysis

##### This was method number 1–This work was improved upon via other methods, but it will be remain here for reference at least until the first iteration of the project is complete.

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

(Code used for editing that shouldn’t be deleted yet:)

``` r
## This code removes names that match in the hope that enough memory would be freed.
#doping_names_without_exact_match <- anti_join(df_weightlifters_with_doping_violations_reduced_lower, positive_testing_lifters_exact_matches, by="athlete_full_name")

#doping_names_without_exact_match <- gsub("[^[:alnum:],]", "", doping_names_without_exact_match)

#doping_names_without_exact_match <- str_split(doping_names_without_exact_match_letters_only, ",")
#str(doping_names_without_exact_match)

#doping_names_without_exact_match <- data.frame(doping_names_without_exact_match)
#doping_names_without_exact_match[doping_names_without_exact_match == "chenadzialiashchuk"] = "henadzialiashchuk"
#colnames(doping_names_without_exact_match)[1] = "athlete_full_name"
#view(doping_names_without_exact_match)

#fuzzy_inner_join(doping_names_without_exact_match, all_weightlifting_results_for_join, by = "athlete_full_name", match_fun = stringdist::stringdistmatrix, method = "jaccard")
```

``` sql
DROP TABLE IF EXISTS weightlifting_results
```
