RXposé
================
Aaron
2023-07-06

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
more recognizably. This dataset is small enough to use a spreadsheet, so
I probably will to start.
