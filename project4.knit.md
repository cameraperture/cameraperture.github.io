---
title: "Project 4: SQL"
format: html
date: November 20, 2024"
---


# Project Description

For this project, I will be using SQL to query Smith College's Wideband Acoustic Immittance (WAI) Database, which contains WAI ear measurements from a multitude of scientific publications. The WAI Database is available at <https://www.science.smith.edu/wai-database/>. My goal is twofold. First, I aim to use SQL and ggplot to replicate Figure 1 from Susan Voss' 2020 study, "An online wideband acoustic immittance (WAI) database and corresponding website", available at <https://pmc.ncbi.nlm.nih.gov/articles/PMC7093226/>. Second, I aim to produce a plot showing race differences in frequency versus mean absorption for one specific study in the WAI database (done in 2010 study by Voss et al.).

## Data Familiarization


::: {.cell}

```{.r .cell-code}
library(mdsr)
library(dbplyr)
library(DBI)
```
:::

::: {.cell}

```{.r .cell-code}
library(RMariaDB)
library(tidyverse)

con_wai <- dbConnect(
  MariaDB(), host = "scidb.smith.edu",
  user = "waiuser", password = "smith_waiDB", 
  dbname = "wai"

)

Measurements <- tbl(con_wai, "Measurements")
PI_Info <- tbl(con_wai, "PI_Info")
Subjects <- tbl(con_wai, "Subjects")
# collect(Measurements)
```
:::

::: {.cell}

```{.sql .cell-code}
SHOW TABLES;
```


<div class="knitsql-table">


Table: 7 records

|Tables_in_wai        |
|:--------------------|
|Codebook             |
|Measurements         |
|Measurements_pre2020 |
|PI_Info              |
|PI_Info_OLD          |
|Subjects             |
|Subjects_pre2020     |

</div>
:::


This displays all of the tables that are in the WAI database. Measurements and Subjects will be of particular interest for what I aim to do.

Measurements contains information about each specific study, notably the frequency and absorbance.


::: {.cell}

```{.sql .cell-code}
SELECT *
FROM Measurements
LIMIT 0, 5;
```


<div class="knitsql-table">


Table: 5 records

|Identifier | SubjectNumber| Session|Ear  |Instrument | Age|AgeCategory |EarStatus | TPP| AreaCanal| PressureCanal|SweepDirection | Frequency| Absorbance|      Zmag|      Zang|
|:----------|-------------:|-------:|:----|:----------|---:|:-----------|:---------|---:|---------:|-------------:|:--------------|---------:|----------:|---------:|---------:|
|Abur_2014  |             1|       1|Left |HearID     |  20|Adult       |Normal    |  -5|  4.42e-05|             0|Ambient        |   210.938|  0.0333379| 113780000| -0.233504|
|Abur_2014  |             1|       1|Left |HearID     |  20|Adult       |Normal    |  -5|  4.42e-05|             0|Ambient        |   234.375|  0.0315705| 103585000| -0.235778|
|Abur_2014  |             1|       1|Left |HearID     |  20|Adult       |Normal    |  -5|  4.42e-05|             0|Ambient        |   257.812|  0.0405751|  92951696| -0.233482|
|Abur_2014  |             1|       1|Left |HearID     |  20|Adult       |Normal    |  -5|  4.42e-05|             0|Ambient        |   281.250|  0.0438399|  86058000| -0.233421|
|Abur_2014  |             1|       1|Left |HearID     |  20|Adult       |Normal    |  -5|  4.42e-05|             0|Ambient        |   304.688|  0.0486400|  79492800| -0.232931|

</div>
:::


Subjects contains information about the participants in each study, including their age, sex, race and ethnicity.


::: {.cell}

```{.sql .cell-code}
SELECT *
FROM Subjects
LIMIT 0, 5;
```


<div class="knitsql-table">


Table: 5 records

|Identifier | SubjectNumber| SessionTotal| AgeFirstMeasurement|AgeCategoryFirstMeasurement |Sex    |Race    |Ethnicity |LeftEarStatusFirstMeasurement |RightEarStatusFirstMeasurement |SubjectNotes                               |
|:----------|-------------:|------------:|-------------------:|:---------------------------|:------|:-------|:---------|:-----------------------------|:------------------------------|:------------------------------------------|
|Abur_2014  |             1|            7|                  20|Adult                       |Female |Unknown |Unknown   |Normal                        |Normal                         |                                           |
|Abur_2014  |             3|            8|                  19|Adult                       |Female |Unknown |Unknown   |Normal                        |Normal                         |Session 5 not included do to acoustic leak |
|Abur_2014  |             4|            7|                  21|Adult                       |Female |Unknown |Unknown   |Normal                        |Normal                         |                                           |
|Abur_2014  |             6|            8|                  21|Adult                       |Female |Unknown |Unknown   |Normal                        |Normal                         |                                           |
|Abur_2014  |             7|            5|                  20|Adult                       |Female |Unknown |Unknown   |Normal                        |Normal                         |                                           |

</div>
:::


PI_Info contains information about the authors, year, journal and title of each study in the database. This will be useful for including the primary author and date of publication in the replicated figure legend.

## Replicating the Figure from Voss, 2020


::: {.cell output.var='graph'}

```{.sql .cell-code}
SELECT Identifier, Frequency, LOG10(Frequency) AS log_frequency,  AVG(Absorbance) AS mean_absorbance 
FROM Measurements
WHERE Identifier IN ("Abur_2014", "Feeney_2017", "Groon_2015" ,"Lewis_2015", "Liu_2008", "Rosowski_2012", "Shahnaz_2006", "Shaver_2013" , "Sun_2016", "Voss_1994", "Voss_2010", "Werner_2010") AND Frequency > 200 AND Frequency < 8000
GROUP BY Identifier, Frequency;  
```
:::

::: {.cell}

```{.r .cell-code}
graph |>
ggplot(aes (x=Frequency, y = mean_absorbance, 
   color =  Identifier,
   group = Identifier)) +
  geom_line() +
  scale_x_log10() +
  labs(title = "Mean absorbance from each publication in WAI database",
      x = "Frequency (Hz)",
      y = "Mean Absorbance",
      color = "Study") +
  xlim(200, 8000) +
  ylim(0, 1) +
  theme_minimal()
```

::: {.cell-output-display}
![](project4_files/figure-html/unnamed-chunk-7-1.png){width=672}
:::
:::


This plot is not an identical replication of the figure in Voss' 2020 study, but it is closely similar. It displays frequency versus mean absorbance measurements for the 12 studies included in the WAI database (as of July 2019). The number of unique ears as well as the equipment used in each study is missing from the legend above. This information is included in the figure I am trying to replicate, so it needs to be displayed here.


::: {.cell output.var='plot2'}

```{.sql .cell-code}
SELECT p.Identifier, p.Year, p.AuthorsShortList, Frequency,
LOG10(Frequency) AS log_frequency, AVG(Absorbance) AS mean_absorbance,
COUNT(DISTINCT SubjectNumber, Ear) AS ear_u,
CONCAT(AuthorsShortList, " (" , year, ") ", "N=", COUNT(DISTINCT SubjectNumber, Ear), "; ", Instrument) AS legend
FROM PI_Info AS p
LEFT JOIN Measurements AS m ON m.Identifier = p.Identifier
WHERE p.Identifier IN ("Abur_2014", "Feeney_2017", "Groon_2015" ,"Lewis_2015", "Liu_2008", "Rosowski_2012", "Shahnaz_2006", "Shaver_2013" , "Sun_2016", "Voss_1994", "Voss_2010", "Werner_2010") AND Frequency > 200 AND Frequency < 8000
GROUP BY Identifier, Instrument, Frequency;
```
:::

::: {.cell}

```{.r .cell-code}
plot2 |>
ggplot(aes(x = Frequency, y = mean_absorbance,
  color = legend,
  group = legend)) +
  geom_line() +
  scale_x_log10() +
  labs(title = "Mean absorbance from each publication in WAI database",
      x = "Frequency (Hz)",
      y = "Mean Absorbance",
      color = "Study, No of individual ears, Equipment") +
  xlim(200, 8000) +
  ylim(0, 1) +
  theme_minimal() +
  theme(legend.text = element_text(size = 6),
        legend.title = element_text(size = 8))
```

::: {.cell-output-display}
![](project4_files/figure-html/unnamed-chunk-9-1.png){width=672}
:::
:::


This plot is identical to the previous one, except that the legend now includes the number of unique ears and the equipment used in the study.

## Race Differences in Mean Absorbance


::: {.cell output.var='groups_graph'}

```{.sql .cell-code}
SELECT s.Race, m.Frequency, AVG(m.Absorbance) AS mean_absorbance 
FROM Subjects AS s 
RIGHT JOIN Measurements AS m ON s.SubjectNumber = m.SubjectNumber
WHERE m.Identifier = "Voss_2010" AND m.Frequency > 200 AND m.Frequency < 8000
GROUP BY s.Race, m.Frequency;
```
:::

::: {.cell}

```{.r .cell-code}
head(groups_graph)
```

::: {.cell-output .cell-output-stdout}

```
              Race Frequency mean_absorbance
1 African American  210.9375      0.06189557
2 African American  234.3750      0.03557816
3 African American  257.8125      0.06311273
4 African American  281.2500      0.05740714
5 African American  304.6875      0.06617098
6 African American  328.1250      0.07566153
```


:::
:::

::: {.cell}

```{.r .cell-code}
groups_graph |>
  ggplot(aes(x = Frequency, y = mean_absorbance)) +
           geom_line(aes(color = Race)) +
  scale_x_log10() +
  labs(title = "Differences in Mean Absorbance by Race",
       subtitle = "Voss et al., 2010",
      x = "Frequency (Hz)",
      y = "Mean Absorbance",
      color = "Race") +
  xlim(200, 6000) +
  ylim(0, 1) +
  theme_minimal()
```

::: {.cell-output-display}
![](project4_files/figure-html/unnamed-chunk-12-1.png){width=672}
:::
:::


I chose to look at differences in mean absorbance for each frequency band by race in one specific study conducted by Voss et al. in 2010. I chose this study because of its large number of participants (1984), which should help to make trends in the data more clear. Each line represents a different race included in the study. It can be seen from the graph that mean absorbance follows very similar trends for White, Caucasian and Asian participants. Meanwhile, the mean absorbances for African American, Black and Mixed participants are noticeably different from those of the other races. Notably, African American participants tend to have a higher mean absorbance for the same frequencies than the other races included in the study, especially for frequencies between 1000 Hz and 5000 Hz. Mixed participants tend to have lower mean absorbances than the other races in the study for frequencies between 2000 Hz and 4000 Hz.

## Conclusion

In this project I used SQL queries to obtain the necessary information from the relevant tables in the WAI database (Subjects, Measurements and PI_Info). These SQL queries enabled me to save the data as R objects, from which I could produce graphs using ggplot. I was able to closely replicate Figure 1 from Voss' 2020 study, and I also produced a graph that showed clear race differences in mean absorbance for Voss et al.'s 2010 study.

