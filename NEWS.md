## Changelog

### 0.1.12

New functions:

- `RImissing()` to generate a figure that shows missing data per item, arranged by amount of missing data.
- `RIdifTable2()` enables DIF interaction analysis between two variables, using the `psychotree` package.

Fixes:

- Changes to use dplyr 1.1.0 syntax for `mutate(across(), ~ function(.x))`, mostly used for rounding numbers in tables/dataframes.
- `RItargeting(data, dich = TRUE)` - bug fix when analyzing dichotomous data - now the first item is also included in item threshold sections.

### 0.1.8.4

New functions:

- `RIestThetas()` and `RIestThetas2()` for estimating person locations/scores. See [example use in vignette]( https://pgmj.github.io/raschrvignette/RaschRvign.html#estimating-interval-level-person-scores).