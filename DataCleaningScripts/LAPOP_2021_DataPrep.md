AmericasBarometer 2021
================

## Data information

All data and resources were downloaded from
<http://datasets.americasbarometer.org/database/> on May 7, 2023.

``` r
library(here) #easy relative paths
```

``` r
library(tidyverse) #data manipulation
library(haven) #data import
library(tidylog) #informative logging messages
library(osfr) # be sure to have PAT saved in Renviron as OSF_PAT
```

## Import data and create derived variables

``` r
stata_files <- osf_retrieve_node("https://osf.io/z5c3m/") %>%
  osf_ls_files(path="LAPOP_2021", n_max=40, pattern=".dta")

read_stata_unlabeled <- function(osf_tbl_i){
  filedet <- osf_tbl_i %>%
    osf_download(conflicts="overwrite", path=here("osf_dl"))
  
  tibin <- filedet %>%
    pull(local_path) %>%
    read_stata() %>%
    zap_labels() %>%
    zap_label()
  
  unlink(pull(filedet, "local_path"))
  
  return(tibin)
}

lapop_in <- stata_files %>% 
  split(1:nrow(stata_files)) %>%
  map_df(read_stata_unlabeled)

# https://www.vanderbilt.edu/lapop/ab2021/AB2021-Core-Questionnaire-v17.5-Eng-210514-W-v2.pdf 
lapop <- lapop_in %>%
  select(pais, strata, upm, weight1500, strata, core_a_core_b,
         q2, q1tb, covid2at, a4, idio2, idio2cov, it1, jc13,
         m1, mil10a, mil10e, ccch1, ccch3, ccus1, ccus3,
         edr, ocup4a, q14, q11n, q12c, q12bn,
         starts_with("covidedu1"), gi0n,
         r15, r18n, r18
         ) 
```

    ## select: dropped 483 variables (idnum, uniq_id, year, wave, nationality, …)

## Save data

``` r
summary(lapop)
```

    ##       pais           strata               upm              weight1500       core_a_core_b            q2              q1tb          covid2at    
    ##  Min.   : 1.00   Min.   :1.000e+08   Min.   :1.001e+07   Min.   :0.004136   Length:64352       Min.   : 16.00   Min.   :1.000   Min.   :1.000  
    ##  1st Qu.: 6.00   1st Qu.:6.000e+08   1st Qu.:6.153e+07   1st Qu.:0.251556   Class :character   1st Qu.: 27.00   1st Qu.:1.000   1st Qu.:1.000  
    ##  Median :11.00   Median :1.100e+09   Median :1.202e+08   Median :0.417251   Mode  :character   Median : 36.00   Median :2.000   Median :2.000  
    ##  Mean   :13.03   Mean   :1.303e+09   Mean   :1.666e+08   Mean   :0.512805                      Mean   : 38.86   Mean   :1.521   Mean   :2.076  
    ##  3rd Qu.:17.00   3rd Qu.:1.700e+09   3rd Qu.:2.105e+08   3rd Qu.:0.674477                      3rd Qu.: 49.00   3rd Qu.:2.000   3rd Qu.:3.000  
    ##  Max.   :41.00   Max.   :4.100e+09   Max.   :1.135e+09   Max.   :7.024495                      Max.   :121.00   Max.   :3.000   Max.   :4.000  
    ##                                                                                                NA's   :90       NA's   :90      NA's   :6686   
    ##        a4             idio2          idio2cov          it1             jc13             m1            mil10a          mil10e          ccch1      
    ##  Min.   :  1.00   Min.   :1.000   Min.   :1.000   Min.   :1.000   Min.   :1.00    Min.   :1.00    Min.   :1.00    Min.   :1.00    Min.   :1.00   
    ##  1st Qu.:  3.00   1st Qu.:2.000   1st Qu.:1.000   1st Qu.:2.000   1st Qu.:1.00    1st Qu.:2.00    1st Qu.:2.00    1st Qu.:2.00    1st Qu.:1.00   
    ##  Median : 22.00   Median :3.000   Median :1.000   Median :2.000   Median :2.00    Median :3.00    Median :3.00    Median :2.00    Median :1.00   
    ##  Mean   : 36.73   Mean   :2.439   Mean   :1.242   Mean   :2.275   Mean   :1.62    Mean   :2.98    Mean   :2.72    Mean   :2.39    Mean   :1.78   
    ##  3rd Qu.: 71.00   3rd Qu.:3.000   3rd Qu.:1.000   3rd Qu.:3.000   3rd Qu.:2.00    3rd Qu.:4.00    3rd Qu.:3.00    3rd Qu.:3.00    3rd Qu.:2.00   
    ##  Max.   :865.00   Max.   :3.000   Max.   :2.000   Max.   :4.000   Max.   :2.00    Max.   :5.00    Max.   :4.00    Max.   :4.00    Max.   :4.00   
    ##  NA's   :4965     NA's   :2766    NA's   :31580   NA's   :3631    NA's   :50827   NA's   :33238   NA's   :49939   NA's   :44021   NA's   :50535  
    ##      ccch3           ccus1           ccus3            edr            ocup4a           q14             q11n            q12c            q12bn       
    ##  Min.   :1.00    Min.   :1.00    Min.   :1.00    Min.   :0.000   Min.   :1.000   Min.   :1.0     Min.   :1.000   Min.   : 1.000   Min.   : 0.000  
    ##  1st Qu.:1.00    1st Qu.:1.00    1st Qu.:1.00    1st Qu.:2.000   1st Qu.:1.000   1st Qu.:1.0     1st Qu.:1.000   1st Qu.: 3.000   1st Qu.: 0.000  
    ##  Median :2.00    Median :1.00    Median :2.00    Median :2.000   Median :1.000   Median :2.0     Median :2.000   Median : 4.000   Median : 1.000  
    ##  Mean   :1.82    Mean   :1.58    Mean   :1.76    Mean   :2.192   Mean   :2.627   Mean   :1.6     Mean   :2.214   Mean   : 4.036   Mean   : 1.001  
    ##  3rd Qu.:2.00    3rd Qu.:2.00    3rd Qu.:2.00    3rd Qu.:3.000   3rd Qu.:4.000   3rd Qu.:2.0     3rd Qu.:3.000   3rd Qu.: 5.000   3rd Qu.: 2.000  
    ##  Max.   :3.00    Max.   :4.00    Max.   :3.00    Max.   :3.000   Max.   :7.000   Max.   :2.0     Max.   :7.000   Max.   :20.000   Max.   :16.000  
    ##  NA's   :51961   NA's   :50028   NA's   :51226   NA's   :4114    NA's   :29505   NA's   :44130   NA's   :31198   NA's   :29144    NA's   :29449   
    ##   covidedu1_1     covidedu1_2     covidedu1_3     covidedu1_4     covidedu1_5         gi0n            r15             r18n            r18       
    ##  Min.   :0.00    Min.   :0.00    Min.   :0.00    Min.   :0.00    Min.   :0.00    Min.   :1.000   Min.   :0.000   Min.   :0.000   Min.   :0.000  
    ##  1st Qu.:0.00    1st Qu.:0.00    1st Qu.:0.00    1st Qu.:0.00    1st Qu.:0.00    1st Qu.:1.000   1st Qu.:0.000   1st Qu.:0.000   1st Qu.:1.000  
    ##  Median :0.00    Median :0.00    Median :1.00    Median :0.00    Median :0.00    Median :1.000   Median :1.000   Median :1.000   Median :1.000  
    ##  Mean   :0.17    Mean   :0.07    Mean   :0.62    Mean   :0.12    Mean   :0.08    Mean   :1.646   Mean   :0.513   Mean   :0.537   Mean   :0.815  
    ##  3rd Qu.:0.00    3rd Qu.:0.00    3rd Qu.:1.00    3rd Qu.:0.00    3rd Qu.:0.00    3rd Qu.:2.000   3rd Qu.:1.000   3rd Qu.:1.000   3rd Qu.:1.000  
    ##  Max.   :1.00    Max.   :1.00    Max.   :1.00    Max.   :1.00    Max.   :1.00    Max.   :5.000   Max.   :1.000   Max.   :1.000   Max.   :1.000  
    ##  NA's   :51297   NA's   :51297   NA's   :51297   NA's   :51297   NA's   :51297   NA's   :1240    NA's   :4118    NA's   :4386    NA's   :4249

``` r
dir.create(here("osf_dl", "LAPOP_2021"))
```

    ## Warning in dir.create(here("osf_dl", "LAPOP_2021")): 'C:\Users\steph\Documents\GitHub\tidy-survey-book\osf_dl\LAPOP_2021' already exists

``` r
lapop_temp_loc <- here("osf_dl", "LAPOP_2021", "lapop_2021.rds")

write_rds(lapop, lapop_temp_loc)

# target_dir <- osf_retrieve_node("https://osf.io/gzbkn/?view_only=8ca80573293b4e12b7f934a0f742b957") 

target_dir <- osf_retrieve_node("https://osf.io/z5c3m/")

osf_upload(target_dir, path=here("osf_dl", "LAPOP_2021"), conflicts="overwrite")
```

    ## Searching for conflicting files on OSF

    ## Retrieving 24 of 24 available items:

    ## ..retrieved 10 items

    ## ..retrieved 20 items

    ## ..retrieved 24 items

    ## ..done

    ## Updating 1 existing file(s) on OSF

    ## # A tibble: 1 × 3
    ##   name       id                       meta            
    ##   <chr>      <chr>                    <list>          
    ## 1 LAPOP_2021 647ce3443c3a380884a04379 <named list [3]>

``` r
unlink(lapop_temp_loc)
```
