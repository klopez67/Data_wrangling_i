tidy_data
================
Kimberly Lopez
2024-09-24

Load packages

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(haven)
```

``` r
pulse_df= 
  read_sas("data_import_examples/public_pulse_data.sas7bdat") |> 
  janitor::clean_names()
```

# Pivot longer

Go from wide format to long format. State columns and where they will be
stores given respective values. Use this if multiple variables contain
the same values for a type of data.

``` r
pulse_tidy_df= 
  pulse_df |>
  pivot_longer(
    cols= bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    values_to= "bdi")
pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit           bdi
    ##    <dbl> <dbl> <chr> <chr>         <dbl>
    ##  1 10003  48.0 male  bdi_score_bl      7
    ##  2 10003  48.0 male  bdi_score_01m     1
    ##  3 10003  48.0 male  bdi_score_06m     2
    ##  4 10003  48.0 male  bdi_score_12m     0
    ##  5 10015  72.5 male  bdi_score_bl      6
    ##  6 10015  72.5 male  bdi_score_01m    NA
    ##  7 10015  72.5 male  bdi_score_06m    NA
    ##  8 10015  72.5 male  bdi_score_12m    NA
    ##  9 10022  58.5 male  bdi_score_bl     14
    ## 10 10022  58.5 male  bdi_score_01m     3
    ## # ℹ 4,338 more rows

In the bdi column, there are prefix of bdi_score we can strip off the
values by:

``` r
pulse_tidy_df = 
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi")

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit   bdi
    ##    <dbl> <dbl> <chr> <chr> <dbl>
    ##  1 10003  48.0 male  bl        7
    ##  2 10003  48.0 male  01m       1
    ##  3 10003  48.0 male  06m       2
    ##  4 10003  48.0 male  12m       0
    ##  5 10015  72.5 male  bl        6
    ##  6 10015  72.5 male  01m      NA
    ##  7 10015  72.5 male  06m      NA
    ##  8 10015  72.5 male  12m      NA
    ##  9 10022  58.5 male  bl       14
    ## 10 10022  58.5 male  01m       3
    ## # ℹ 4,338 more rows

After, we should mutate the visit variable to replace values “bl” since
it is not in the format of months. Replace values “b” to 00m: - relocate
the columns so visit is in the second columns

``` r
pulse_tidy_df = 
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi")|>
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)) |>
  relocate(id,visit)

print(pulse_tidy_df, n = 12)
```

    ## # A tibble: 4,348 × 5
    ##       id visit   age sex     bdi
    ##    <dbl> <fct> <dbl> <chr> <dbl>
    ##  1 10003 00m    48.0 male      7
    ##  2 10003 01m    48.0 male      1
    ##  3 10003 06m    48.0 male      2
    ##  4 10003 12m    48.0 male      0
    ##  5 10015 00m    72.5 male      6
    ##  6 10015 01m    72.5 male     NA
    ##  7 10015 06m    72.5 male     NA
    ##  8 10015 12m    72.5 male     NA
    ##  9 10022 00m    58.5 male     14
    ## 10 10022 01m    58.5 male      3
    ## 11 10022 06m    58.5 male      8
    ## 12 10022 12m    58.5 male     NA
    ## # ℹ 4,336 more rows

To do all the coding in one shot: - janitor - pivot wide to long -
remove common prefix - mutate data & relocate

``` r
pulse_df = 
  haven::read_sas("data_import_examples/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi") |>
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)) |>
  relocate(id, visit)

print(pulse_df, n = 12)
```

    ## # A tibble: 4,348 × 5
    ##       id visit   age sex     bdi
    ##    <dbl> <fct> <dbl> <chr> <dbl>
    ##  1 10003 00m    48.0 male      7
    ##  2 10003 01m    48.0 male      1
    ##  3 10003 06m    48.0 male      2
    ##  4 10003 12m    48.0 male      0
    ##  5 10015 00m    72.5 male      6
    ##  6 10015 01m    72.5 male     NA
    ##  7 10015 06m    72.5 male     NA
    ##  8 10015 12m    72.5 male     NA
    ##  9 10022 00m    58.5 male     14
    ## 10 10022 01m    58.5 male      3
    ## 11 10022 06m    58.5 male      8
    ## 12 10022 12m    58.5 male     NA
    ## # ℹ 4,336 more rows

Doing one more example with the Litters data: - gd_weight has 2 columns
in the original dataset - use case match function to automatically take
a variable and recode/replace as a value. Must list out all the options.

``` r
litters_df= 
  read_csv("data_import_examples/FAS_litters.csv")|>
  janitor::clean_names() |>
  pivot_longer(
    gd0_weight:gd18_weight, 
    names_to = "gd_time",
    values_to = "weight"
  )|>
  mutate(
    gd_time = case_match(
      gd_time, 
      "gd0_weight" ~0,
      "gd18_weight" ~ 18, 
      
    )
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (4): Group, Litter Number, GD0 weight, GD18 weight
    ## dbl (4): GD of Birth, Pups born alive, Pups dead @ birth, Pups survive
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df
```

    ## # A tibble: 98 × 8
    ##    group litter_number gd_of_birth pups_born_alive pups_dead_birth pups_survive
    ##    <chr> <chr>               <dbl>           <dbl>           <dbl>        <dbl>
    ##  1 Con7  #85                    20               3               4            3
    ##  2 Con7  #85                    20               3               4            3
    ##  3 Con7  #1/2/95/2              19               8               0            7
    ##  4 Con7  #1/2/95/2              19               8               0            7
    ##  5 Con7  #5/5/3/83/3-3          19               6               0            5
    ##  6 Con7  #5/5/3/83/3-3          19               6               0            5
    ##  7 Con7  #5/4/2/95/2            19               5               1            4
    ##  8 Con7  #5/4/2/95/2            19               5               1            4
    ##  9 Con7  #4/2/95/3-3            20               6               0            6
    ## 10 Con7  #4/2/95/3-3            20               6               0            6
    ## # ℹ 88 more rows
    ## # ℹ 2 more variables: gd_time <dbl>, weight <chr>

Lets make up an analysis result table.

``` r
analysis_df= 
  tibble(
    group = c("treatment","control","control","treatment"),
    time = c("pre","post","pre","post"),
    mean = c(4,10,4.2,5)
    
  )
```

# Pivot Wider

Pivot Wider for readability. Helpful for post analysis results. - took
previous dataset - took column names from time variable - values are
coming from the mean variable

``` r
analysis_df |>
  pivot_wider(
    names_from = time, 
    values_from = mean
  )
```

    ## # A tibble: 2 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       5
    ## 2 control     4.2    10
