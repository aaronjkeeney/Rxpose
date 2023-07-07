RXposé
================
Aaron
2023-07-07

## Rxposé: An Analysis of Performance-Enhancing Drug Use in the Sport of Weightlifting

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
getwd()
```

    ## [1] "/Users/aaronkeeney/Documents/Data Analytics Projects/Rxpose"

``` r
setwd("~/Documents/Data Analytics Projects/Rxpose")
```

First, I sorted data in BigQuery, as it was much easier to visualize
there. Only weightlifting data was extracted, and only from 1976 and
later. This is due to two reasons. First, there was a [rule
shift](https://en.wikipedia.org/wiki/Clean_and_press) after 1972 that
greatly affected the total weight lifted. Second, anabolic steroids were
not banned until 1975.
[source](https://www.acog.org/clinical/clinical-guidance/committee-opinion/articles/2011/04/performance-enhancing-anabolic-steroid-abuse-in-women#:~:text=Anabolic%20steroids%20were%20first%20discovered,performance%20and%20enhance%20cosmetic%20appearance.)

Note: I know it is possible to run SQL code in R– The next project is
setting that up.

``` r
##SELECT event_title,rank_position, country_3_letter_code, athlete_full_name,
##  CAST(value_unit AS decimal) AS total_kg, value_type,
##  CAST(RIGHT(slug_game,4) AS INT64) AS year
##FROM
##  `weightliftinganalysis.Olympics_Results_86_to_20.Olympics Results`
##WHERE
##  discipline_title = "Weightlifting" AND
##  value_type IS NOT NULL AND
##  CAST(RIGHT(slug_game,4) AS decimal) >1972 
  
  ## Why can I not use "year" as a variable here? Add it to the table with a join?
```

Mostly, this code is intended to convert data types and to name columns
more recognizably. This data set is small enough to use a spreadsheet,
so I will finish cleaning and do initial analysis there.

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