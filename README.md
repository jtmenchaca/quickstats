
<!-- README.md is generated from README.Rmd. Please edit that file -->

# quickcompare

<!-- badges: start -->
<!-- badges: end -->

The goal of quickcompare is to provide a tool to quickly compare column
values by groups with the appropriate statistical tests and output the
results in a publication-ready table.

## Installation

You can install the development version of quickcompare from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("jtmenchaca/quickcompare")
```

## Create a comparison

``` r
library(quickcompare)
library(dplyr, quietly = T, warn.conflicts = F)
library(palmerpenguins, quietly = T, warn.conflicts = F)

data(penguins, package = "palmerpenguins")
penguins_data = penguins |>
  filter(species != "Gentoo") # Simplifying example for only 2 groups

summary = penguins_data |> 
  compare_cols_by_group(
    group_col = "species", 
    binary_or_cat_cols = c("island", "sex", "year"), 
    continuous_cols = c("bill_length_mm", "bill_depth_mm", 
                        "flipper_length_mm", "body_mass_g")
  )
```

By default, the binary/categorical columns are compared across groups
using the Chi square test or the Fisher’s Exact test, depending on the
count of results. If there are no observations for a given subgroup, no
p-value is provided (consider comparing across sub-groups that all have
at least one observation in a separate analysis).

``` r
penguins_data |> 
  compare_cols_by_group(
    group_col = "species", 
    binary_or_cat_cols = c("island", "sex", "year")
  )
#> # A tibble: 13 × 5
#>    Characteristic   `Species - Adelie` `Species - Chinstrap` `p-value` statist…¹
#>    <chr>            <chr>              <chr>                 <chr>     <chr>    
#>  1 "Total Count, n" "152"              "68"                  ""        ""       
#>  2 "Island, n (%)"  ""                 ""                    ""        ""       
#>  3 "     Biscoe"    "44 (28.9)"        "0 (0)"               ""        "n/a"    
#>  4 "     Dream"     "56 (36.8)"        "68 (100)"            "<.01"    "Chi-squ…
#>  5 "     Torgersen" "52 (34.2)"        "0 (0)"               ""        "n/a"    
#>  6 "Sex, n (%)"     ""                 ""                    ""        ""       
#>  7 "     female"    "73 (48)"          "34 (50)"             ".9"      "Chi-squ…
#>  8 "     male"      "73 (48)"          "34 (50)"             ".9"      "Chi-squ…
#>  9 "     Missing"   "6 (3.9)"          "0 (0)"               ""        "n/a"    
#> 10 "Year, n (%)"    ""                 ""                    ""        ""       
#> 11 "     2007"      "50 (32.9)"        "26 (38.2)"           ".54"     "Chi-squ…
#> 12 "     2008"      "50 (32.9)"        "18 (26.5)"           ".43"     "Chi-squ…
#> 13 "     2009"      "52 (34.2)"        "24 (35.3)"           "1"       "Chi-squ…
#> # … with abbreviated variable name ¹​statistical_test
```

Continuous columns are compared between two groups by either 1) the
Student’s t-test if the column has a normal distribution or 2) the
Wilcoxon signed-rank test if the distribution is non-normal. The
Shapiro–Wilk test is used to evaluate normality.

When comparing between three groups, the ANOVA test is used to compare
continuous columns.

If a continuous variable has a normal distribution, the mean and
standard deviation are provided. If non-normal, the median and
interquartile range are provided.

``` r
penguins_data |> 
  compare_cols_by_group(
    group_col = "species", 
    continuous_cols = c("bill_length_mm", "bill_depth_mm", 
                        "flipper_length_mm", "body_mass_g")
  )
```

## Save your results to Excel

Neatly translate your formatted results to an XLSX file with the helper
function `save_comparison_to_xlsx`.

``` r
penguins_data |> 
  compare_cols_by_group(
    group_col = "species", 
    binary_or_cat_cols = c("island", "sex", "year"), 
    continuous_cols = c("bill_length_mm", "bill_depth_mm", 
                        "flipper_length_mm", "body_mass_g")
  ) |>
   save_comparison_to_xlsx(
     file_name = "Comparison.xlsx",
     title = "Table 1: Characteristics of Penguins by Species"
  )
```

It should leave you with a tidy XLSX file with something that looks like
the following:

![A tidy XLSX table](man/figures/README-example-xlsx.png)

## Extra options

Use the `remove_group_col_NA` and `cols_to_remove_NA` to remove rows
with missing values in specific columns.

Sometimes, you want to use a subset of your dataset to compare across
groups for your binary/categorical data. To specify the column you would
like to use as a subset, use the `binary_or_cat_cols_subpop` argument.

If the `binary_or_cat_cols_subpop` argument is provided, it must be the
same length as the `binary_or_cat_cols` argument where each value in
`binary_or_cat_cols` aligns by index to the value in the
`binary_or_cat_cols_subgroup`. Columns that should use the general
population should have a value of the empty string `""`.

For now, any columns specified in `binary_or_cat_cols_subpop` should be
binary columns, where a value of `1` will be used to identify the
appropriate subpopulation.

*J.T. Menchaca*
