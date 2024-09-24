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
it is not in the format of months. Replace values “b” to 00m:

- relocate the columns so visit is in the second columns

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

To do all the coding in one shot:

- janitor
- pivot wide to long
- remove common prefix
- mutate data & relocate

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

Pivot Wider for readability. Helpful for post analysis results.

- took previous dataset
- took column names from time variable
- values are coming from the mean variable
- use knitr::kable() for r built in readability

``` r
analysis_df |>
  pivot_wider(
    names_from = time, 
    values_from = mean
  )|>
  knitr::kable()
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |    5 |
| control   | 4.2 |   10 |

# Binding Rows

Load data, to create a tidy dataframe that has all the variables from 3
different tables.

Binding Tables

- read excels with ranges that correspond to each table in the data set
- ensure each table corresponds with their movie name using mutate()

``` r
fellowship_ring = 
  readxl::read_excel("data_import_examples/LotR_Words.xlsx", range = "B3:D6") |>
  mutate(movie = "fellowship_ring")

two_towers = 
  readxl::read_excel("data_import_examples/LotR_Words.xlsx", range = "F3:H6") |>
  mutate(movie = "two_towers")

return_king = 
  readxl::read_excel("data_import_examples/LotR_Words.xlsx", range = "J3:L6") |>
  mutate(movie = "return_king")
```

Use Bind rows to stack the tables together

- also clean up using janitor
- pivot to longer

``` r
lotr_df= 
  bind_rows(fellowship_ring, two_towers,return_king)|> 
  janitor:: clean_names()|>
  pivot_longer(
    female:male,
    names_to = "sex", 
    values_to = "words"
    ) |>
  relocate(movie)|>
  mutate(race = str_to_lower(race))
```

# Joining Datasets

There are four major ways join dataframes x and y: - Inner: keeps data
that appear in both x and y - Left: keeps data that appear in x - Right:
keeps data that appear in y - Full: keeps data that appear in either x
or y

Join FAS datasets

- Import “litters” dataset
- mutates so that we know what weight gained
- seperate the variable
- use seperate to seperate column

``` r
litters_df= 
  read_csv("data_import_examples/FAS_litters.csv", na = c("NA","","."))|>
  janitor::clean_names()|>
  mutate( 
    wt_gain = gd18_weight - gd0_weight,
   )|>
  separate(
    group, into = c("dose","day_of_treatment"), sep=3 
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df
```

    ## # A tibble: 49 × 10
    ##    dose  day_of_treatment litter_number   gd0_weight gd18_weight gd_of_birth
    ##    <chr> <chr>            <chr>                <dbl>       <dbl>       <dbl>
    ##  1 Con   7                #85                   19.7        34.7          20
    ##  2 Con   7                #1/2/95/2             27          42            19
    ##  3 Con   7                #5/5/3/83/3-3         26          41.4          19
    ##  4 Con   7                #5/4/2/95/2           28.5        44.1          19
    ##  5 Con   7                #4/2/95/3-3           NA          NA            20
    ##  6 Con   7                #2/2/95/3-2           NA          NA            20
    ##  7 Con   7                #1/5/3/83/3-3/2       NA          NA            20
    ##  8 Con   8                #3/83/3-3             NA          NA            20
    ##  9 Con   8                #2/95/3               NA          NA            20
    ## 10 Con   8                #3/5/2/2/95           28.5        NA            20
    ## # ℹ 39 more rows
    ## # ℹ 4 more variables: pups_born_alive <dbl>, pups_dead_birth <dbl>,
    ## #   pups_survive <dbl>, wt_gain <dbl>

Import FAS “pups” dataset

``` r
pups_df = 
  read_csv("data_import_examples/FAS_pups.csv",na = c("NA", "", ".")) |>
  janitor::clean_names() |>
  mutate(
    sex = 
      case_match(
        sex, 
        1 ~ "male", 
        2 ~ "female"),
    sex = as.factor(sex))
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df
```

    ## # A tibble: 313 × 6
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #85           male        4      13        7      11
    ##  2 #85           male        4      13        7      12
    ##  3 #1/2/95/2     male        5      13        7       9
    ##  4 #1/2/95/2     male        5      13        8      10
    ##  5 #5/5/3/83/3-3 male        5      13        8      10
    ##  6 #5/5/3/83/3-3 male        5      14        6       9
    ##  7 #5/4/2/95/2   male       NA      14        5       9
    ##  8 #4/2/95/3-3   male        4      13        6       8
    ##  9 #4/2/95/3-3   male        4      13        7       9
    ## 10 #2/2/95/3-2   male        4      NA        8      10
    ## # ℹ 303 more rows

Join the datasets - matching by variable litter number - using left
joining into the pups data frame. - rearrange so litter_number, dose,
day of treatment follow order

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number")|>
  relocate (litter_number, dose, day_of_treatment)

fas_df
```

    ## # A tibble: 313 × 15
    ##    litter_number dose  day_of_treatment sex   pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <chr> <chr>            <fct>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #85           Con   7                male        4      13        7      11
    ##  2 #85           Con   7                male        4      13        7      12
    ##  3 #1/2/95/2     Con   7                male        5      13        7       9
    ##  4 #1/2/95/2     Con   7                male        5      13        8      10
    ##  5 #5/5/3/83/3-3 Con   7                male        5      13        8      10
    ##  6 #5/5/3/83/3-3 Con   7                male        5      14        6       9
    ##  7 #5/4/2/95/2   Con   7                male       NA      14        5       9
    ##  8 #4/2/95/3-3   Con   7                male        4      13        6       8
    ##  9 #4/2/95/3-3   Con   7                male        4      13        7       9
    ## 10 #2/2/95/3-2   Con   7                male        4      NA        8      10
    ## # ℹ 303 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>

# Other Functions on Names

- left_join(x, y)
- inner_join (, )
- if there are multiple of the same matched data, it will give warnings
- anti_join(, )
