# Class17
Peyton Chiu (PID:A18145937)

## Intro

The United States *Centers for Disease Control and Prevention* (CDC) has
been compiling reported pertussis case numbers since 1922 in
their *National Notifiable Diseases Surveillance System* (NNDSS). We can
view this data on the CDC website here:
https://www.cdc.gov/pertussis/surv-reporting/cases-by-year.htmll

> - **Q1.** With the help of the R “addin” package
>   [**datapasta**](https://milesmcbain.github.io/datapasta/) assign the
>   CDC pertussis case number data to a data frame called `cdc` and use
>   **ggplot** to make a plot of cases numbers over time.

``` r
library(ggplot2)

ggplot(cdc) +
  aes(x = year, y = cases) +
  geom_point() +
  geom_line() +
  labs(
    title = "Pertussis Cases in the United States by Year",
    x = "Year",
    y = "Reported Cases" )
```

![](Class17_files/figure-commonmark/unnamed-chunk-1-1.png)

> - **Q2.** Using the ggplot `geom_vline()` function add lines to your
>   previous plot for the 1946 introduction of the wP vaccine and the
>   1996 switch to aP vaccine (see example in the hint below). What do
>   you notice?

``` r
ggplot(cdc) +
  aes(x = year, y = cases) +
  geom_point() +
  geom_line() +
  geom_vline(xintercept = 1946, color = "blue", linetype = "dashed") +   # wP introduction
  geom_vline(xintercept = 1996, color = "red", linetype = "dashed") +    # switch to aP
  labs(
    title = "Pertussis Cases in the United States by Year",
    x = "Year",
    y = "Reported Cases"
  )
```

![](Class17_files/figure-commonmark/unnamed-chunk-2-1.png)

The total number of cases dropped after the whole cell vaccine was
introduced, before trickling up with the introduction of acellular
vaccines

> **Q3.** Describe what happened after the introduction of the aP
> vaccine? Do you have a possible explanation for the observed trend?

After switching to acellular vaccines pertussis cases began increasing
again. One possible explanation would be this version using few
antigens, including weaker pertussis toxins, to avoid strong side
effects.

## **Exploring CMI-PB data**

The new and ongoing [**CMI-PB project**](https://www.cmi-pb.org/) aims
to provide the scientific community with this very information. In
particular, CMI-PB tracks and makes freely available long-term humoral
and cellular immune response data for a large number of individuals who
received either DTwP or DTaP combination vaccines in infancy followed by
Tdap booster vaccinations. This includes complete API access to
longitudinal RNA-Seq, AB Titer, Olink, and live cell assay results
directly from their website: https://www.cmi-pb.org/

``` r
library(jsonlite)
```

    Warning: package 'jsonlite' was built under R version 4.5.2

``` r
subject <- read_json("https://www.cmi-pb.org/api/subject", simplifyVector = TRUE) 
table(subject$infancy_vac)
```


    aP wP 
    87 85 

> **Q4.** How many aP and wP infancy vaccinated subjects are in the
> dataset?

There are 87 aP and 85wP

> **Q5.** How many Male and Female subjects/patients are in the dataset?

``` r
table(subject$biological_sex)
```


    Female   Male 
       112     60 

There are 60 males and 112 females

> **Q6.** What is the breakdown of race and biological sex (e.g. number
> of Asian females, White males etc…)?

``` r
table(subject$race, subject$biological_sex)
```

                                               
                                                Female Male
      American Indian/Alaska Native                  0    1
      Asian                                         32   12
      Black or African American                      2    3
      More Than One Race                            15    4
      Native Hawaiian or Other Pacific Islander      1    1
      Unknown or Not Reported                       14    7
      White                                         48   32

Using Dates

Two of the columns of `subject` contain dates in the Year-Month-Day
format. Recall from our last mini-project that dates and times can be
annoying to work with at the best of times. However, in R we have the
excellent lubridate package, which can make life allot easier.

Converting the dates and finding the age

``` r
library(lubridate)
```

    Warning: package 'lubridate' was built under R version 4.5.2


    Attaching package: 'lubridate'

    The following objects are masked from 'package:base':

        date, intersect, setdiff, union

``` r
subject$year_of_birth <- ymd(subject$year_of_birth)
subject$date_of_boost <- ymd(subject$date_of_boost)


subject$age_today <- today() - subject$year_of_birth
subject$age_today_years <- time_length(subject$age_today, "years")
```

> **Q7.** Using this approach determine (i) the average age of wP
> individuals, (ii) the average age of aP individuals; and (iii) are
> they significantly different?

``` r
tapply(subject$age_today_years, subject$infancy_vac, mean, na.rm = TRUE)
```

          aP       wP 
    27.82553 36.57624 

``` r
t.test(age_today_years ~ infancy_vac, data = subject)
```


        Welch Two Sample t-test

    data:  age_today_years by infancy_vac
    t = -12.918, df = 104.03, p-value < 2.2e-16
    alternative hypothesis: true difference in means between group aP and group wP is not equal to 0
    95 percent confidence interval:
     -10.094058  -7.407351
    sample estimates:
    mean in group aP mean in group wP 
            27.82553         36.57624 

The average age for aP is 27.8 yrs and for wP it is 36.6 yrs. These
differences are statistically different (p value is below the 0.05
threshold)

> **Q8.** Determine the age of all individuals at time of boost?

``` r
subject$age_at_boost <- time_length(subject$date_of_boost - subject$year_of_birth,"years")

summary(subject$age_at_boost)
```

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      18.83   21.03   25.75   26.09   29.56   51.07 

The ages ranged from 18.83 to 51.07 years with the median age being
25.75 years

> **Q9.** With the help of a faceted boxplot or histogram (see below),
> do you think these two groups are significantly different?

``` r
ggplot(subject) +
  aes(x = infancy_vac, y = age_at_boost, fill = infancy_vac) +
  geom_boxplot(show.legend = FALSE) +
  xlab("Infancy vaccine type") +
  ylab("Age at boost (years)")
```

![](Class17_files/figure-commonmark/unnamed-chunk-9-1.png)

The histograms show very different age distribution supporting the idea
that they are statistically different

Opening up the specimen and ab_titer data

``` r
library(jsonlite)
library(dplyr)
```

    Warning: package 'dplyr' was built under R version 4.5.2


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(jsonlite)


specimen <- read_json("https://www.cmi-pb.org/api/specimen",
                      simplifyVector = TRUE)

titer <- read_json("https://www.cmi-pb.org/api/plasma_ab_titer",
                   simplifyVector = TRUE)
```

To know whether a given `specimen_id` comes from an aP or wP individual
we need to link (a.k.a. “join” or merge)
our `specimen` and `subject` data frames. 

> **Q9.** Complete the code to join `specimen` and `subject` tables to
> make a new merged data frame containing all specimen records along
> with their associated subject details

``` r
meta <- inner_join(specimen, subject)
```

    Joining with `by = join_by(subject_id)`

``` r
dim(meta)
```

    [1] 1503   16

``` r
head(meta)
```

      specimen_id subject_id actual_day_relative_to_boost
    1           1          1                           -3
    2           2          1                            1
    3           3          1                            3
    4           4          1                            7
    5           5          1                           11
    6           6          1                           32
      planned_day_relative_to_boost specimen_type visit infancy_vac biological_sex
    1                             0         Blood     1          wP         Female
    2                             1         Blood     2          wP         Female
    3                             3         Blood     3          wP         Female
    4                             7         Blood     4          wP         Female
    5                            14         Blood     5          wP         Female
    6                            30         Blood     6          wP         Female
                   ethnicity  race year_of_birth date_of_boost      dataset
    1 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    2 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    3 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    4 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    5 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    6 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
       age_today age_today_years age_at_boost
    1 14584 days        39.92882     30.69678
    2 14584 days        39.92882     30.69678
    3 14584 days        39.92882     30.69678
    4 14584 days        39.92882     30.69678
    5 14584 days        39.92882     30.69678
    6 14584 days        39.92882     30.69678

> **Q10.** Now using the same procedure join `meta` with `titer` data so
> we can further analyze this data in terms of time of visit aP/wP,
> male/female etc

``` r
abdata <- inner_join(titer, meta)
```

    Joining with `by = join_by(specimen_id)`

``` r
dim(abdata)
```

    [1] 52576    23

> **Q11.** How many specimens (i.e. entries in `abdata`) do we have for
> each `isotype`?

``` r
table(abdata$isotype)
```


      IgE   IgG  IgG1  IgG2  IgG3  IgG4 
     6698  5389 10117 10124 10124 10124 

> **Q12.** What are the different `$dataset` values in `abdata` and what
> do you notice about the number of rows for the most “recent” dataset?

``` r
table(abdata$dataset)
```


    2020_dataset 2021_dataset 2022_dataset 2023_dataset 
           31520         8085         7301         5670 

The most recent dataset is the smallest

# **Examine IgG Ab titer levels**

``` r
igg <- abdata %>% filter(isotype == "IgG")
head(igg)
```

      specimen_id isotype is_antigen_specific antigen        MFI MFI_normalised
    1           1     IgG                TRUE      PT   68.56614       3.736992
    2           1     IgG                TRUE     PRN  332.12718       2.602350
    3           1     IgG                TRUE     FHA 1887.12263      34.050956
    4          19     IgG                TRUE      PT   20.11607       1.096366
    5          19     IgG                TRUE     PRN  976.67419       7.652635
    6          19     IgG                TRUE     FHA   60.76626       1.096457
       unit lower_limit_of_detection subject_id actual_day_relative_to_boost
    1 IU/ML                 0.530000          1                           -3
    2 IU/ML                 6.205949          1                           -3
    3 IU/ML                 4.679535          1                           -3
    4 IU/ML                 0.530000          3                           -3
    5 IU/ML                 6.205949          3                           -3
    6 IU/ML                 4.679535          3                           -3
      planned_day_relative_to_boost specimen_type visit infancy_vac biological_sex
    1                             0         Blood     1          wP         Female
    2                             0         Blood     1          wP         Female
    3                             0         Blood     1          wP         Female
    4                             0         Blood     1          wP         Female
    5                             0         Blood     1          wP         Female
    6                             0         Blood     1          wP         Female
                   ethnicity  race year_of_birth date_of_boost      dataset
    1 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    2 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    3 Not Hispanic or Latino White    1986-01-01    2016-09-12 2020_dataset
    4                Unknown White    1983-01-01    2016-10-10 2020_dataset
    5                Unknown White    1983-01-01    2016-10-10 2020_dataset
    6                Unknown White    1983-01-01    2016-10-10 2020_dataset
       age_today age_today_years age_at_boost
    1 14584 days        39.92882     30.69678
    2 14584 days        39.92882     30.69678
    3 14584 days        39.92882     30.69678
    4 15680 days        42.92950     33.77413
    5 15680 days        42.92950     33.77413
    6 15680 days        42.92950     33.77413

> **Q13.** Complete the following code to make a summary boxplot of Ab
> titer levels (MFI) for all antigens:

``` r
ggplot(igg) +
  aes(x = MFI_normalised, y = antigen) +
  geom_boxplot() + 
  xlim(0, 75) +
  facet_wrap(vars(visit), nrow = 2)
```

    Warning: Removed 5 rows containing non-finite outside the scale range
    (`stat_boxplot()`).

![](Class17_files/figure-commonmark/unnamed-chunk-16-1.png)

> **Q14.** What antigens show differences in the level of IgG antibody
> titers recognizing them over time? Why these and not others?

The antigens that show differences in the anti body levels over time are
PT, PRN ,FHA, and Fim2/3. These antigens are present in the vaccine
meaning that the booster help spur their activation. The other antigens
presumably are not present.

``` r
ggplot(igg) +
  aes(MFI_normalised, antigen, col=infancy_vac ) +
  geom_boxplot(show.legend = FALSE) + 
  facet_wrap(vars(visit), nrow=2) +
  xlim(0,75) +
  theme_bw()
```

    Warning: Removed 5 rows containing non-finite outside the scale range
    (`stat_boxplot()`).

![](Class17_files/figure-commonmark/unnamed-chunk-17-1.png)

We can also adding infant vaccinations

``` r
igg %>% filter(visit != 8) %>%
ggplot() +
  aes(MFI_normalised, antigen, col=infancy_vac ) +
  geom_boxplot(show.legend = FALSE) + 
  xlim(0,75) +
  facet_wrap(vars(infancy_vac, visit), nrow=2)
```

    Warning: Removed 5 rows containing non-finite outside the scale range
    (`stat_boxplot()`).

![](Class17_files/figure-commonmark/unnamed-chunk-18-1.png)

> **Q15.** Filter to pull out only two specific antigens for analysis
> and create a boxplot for each. You can chose any you like. Below I
> picked a “control” antigen (**“OVA”**, that is not in our vaccines)
> and a clear antigen of interest (**“PT”**, Pertussis Toxin, one of the
> key virulence factors produced by the bacterium *B. pertussis*).

``` r
# plot for ova antigen
filter(igg, antigen == "OVA") %>%
  ggplot() +
  aes(MFI_normalised, col = infancy_vac) +
  geom_boxplot(show.legend = TRUE) +
  facet_wrap(vars(visit)) +
  theme_bw()
```

![](Class17_files/figure-commonmark/unnamed-chunk-19-1.png)

``` r
# plot for pt antigen
filter(igg, antigen == "PT") %>%
  ggplot() +
  aes(MFI_normalised, col = infancy_vac) +
  geom_boxplot(show.legend = TRUE) +
  facet_wrap(vars(visit)) +
  theme_bw()
```

![](Class17_files/figure-commonmark/unnamed-chunk-19-2.png)

> **Q16.** What do you notice about these two antigens time courses and
> the PT data in particular?

The ova shows stable antigen levels relatively compared to the PT whose
levels show notable fluctuations

> **Q17.** Do you see any clear difference in aP vs. wP responses?

On average wP responses then to be stronger with have MFI normalized
values compared to aP

``` r
abdata.21 <- abdata %>% filter(dataset == "2021_dataset")

abdata.21 %>% 
  filter(isotype == "IgG",  antigen == "PT") %>%
  ggplot() +
    aes(x=planned_day_relative_to_boost,
        y=MFI_normalised,
        col=infancy_vac,
        group=subject_id) +
    geom_point() +
    geom_line() +
    geom_vline(xintercept=0, linetype="dashed") +
    geom_vline(xintercept=14, linetype="dashed") +
  labs(title="2021 dataset IgG PT",
       subtitle = "Dashed lines indicate day 0 (pre-boost) and 14 (apparent peak levels)")
```

![](Class17_files/figure-commonmark/unnamed-chunk-20-1.png)

> **Q18.** Does this trend look similar for the 2020 dataset?

Yes wP individuals seem to show higher levels of responses compared to
aP individuals

# **Obtaining CMI-PB RNASeq data**

For RNA-Seq data the API query mechanism quickly hits the web browser
interface limit for file size. We will present alternative download
mechanisms for larger CMI-PB datasets in the next section. However, we
can still do “targeted” RNA-Seq querys via the web accessible API.

``` r
url <- "https://www.cmi-pb.org/api/v2/rnaseq?versioned_ensembl_gene_id=eq.ENSG00000211896.7"

rna <- read_json(url, simplifyVector = TRUE) 

#meta <- inner_join(specimen, subject)
ssrna <- inner_join(rna, meta)
```

    Joining with `by = join_by(specimen_id)`

> **Q19.** Make a plot of the time course of gene expression for IGHG1
> gene (i.e. a plot of `visit` vs. `tpm`).

``` r
ggplot(ssrna) +
  aes(x = visit, y = tpm, group = subject_id) +
  geom_point() +
  geom_line(alpha = 0.2)
```

![](Class17_files/figure-commonmark/unnamed-chunk-22-1.png)

> **Q20.**: What do you notice about the expression of this gene
> (i.e. when is it at it’s maximum level)?

The expression is highest around 3 to 4 visits and 8 visits where it
peaks 6000+ tpm

> **Q21.** Does this pattern in time match the trend of antibody titer
> data? If not, why not?

No. The antibody titer data typically shows a single spikes before
trending downwards with very few upward spikes after the initial big one
