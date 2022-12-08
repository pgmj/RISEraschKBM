# RISEkbmRasch
R package for Rasch Measurement Theory based psychometric analysis. Intended for use with [Quarto](https://quarto.org) for documentation and presentation of analysis process and results. This package uses other packages for the Rasch analyses, such as [eRm](https://cran.r-project.org/web/packages/eRm/), [mirt](https://cran.r-project.org/web/packages/mirt/) and [psychotree](https://cran.r-project.org/web/packages/psychotree/), and aims to simplify the steps in Rasch analysis to provide tables and figures with functions that have few options.

## Installation

First, install the [`devtools`](https://devtools.r-lib.org/) package:
```r
install.packages('devtools')
```

Then install the package and its dependencies: 
```r
devtools::install_github("pgmj/RISEkbmRasch", dependencies = TRUE)
```

And, while not necessary, it is highly recommended to install Quarto (and update your Rstudio installation if needed):
- https://quarto.org/docs/get-started/
- https://posit.co/download/rstudio-desktop/

### Upgrading
```r
detach("package:RISEkbmRasch", unload = TRUE) # not needed if you haven't loaded the package in your current session
devtools::install_github("pgmj/RISEkbmRasch")
```

## Usage

Most functions in this package are relatively simple wrappers that create outputs such as tables and figures to make the Rasch analysis process quick and visual. There is a [companion Quarto template file](https://github.com/pgmj/RISEkbmRasch/tree/main/Quarto) that shows suggested ways to use this package.

There are two basic data structure requirements:

- you need to create a dataframe object named `itemlabels` that consists of two variables/columns:
  - the **first one** named `itemnr`, containing variable names exactly as they are named in the dataframe containing data (for example q1, q2, q3, etc)
  - the **second one** named `item`, containing either the questionnaire item or a description of it (or description of a task)
- the data you want to analyze needs to be in a dataframe with participants as rows and items as columns/variables
  - the lowest response category needs to be zero (0). Recode as needed, the Quarto template file contains code for this.
  - during data import, you will need to separate any demographic variables into vectors, for analysis of differential item functioning (DIF), and then remove them from the dataframe with item data. **The dataframe with item data can only contain item data for the analysis functions to work** (no ID variable or other demographic variables).

For most Rasch-related functions in the package, there are separate functions for polytomous data (more than two response options for each item) and dichotomous data, except `RItargeting()` which defaults to polytomous data and has the option `dich = TRUE` for dichotomous data. For instance, `RIitemfitPCM()` for the Partial Credit Model and `RIitemfitRM()` for the dichotomous Rasch Model. The Rating Scale Model (RSM) for polytomous data has not been implemented in any of the functions.

More details and examples of use will be added. You can find two Quarto use cases with Rasch analyses in the [project subfolder Quarto](https://github.com/pgmj/RISEkbmRasch/tree/main/Quarto). You will need to copy all the text from the .qmd template file and paste it into a new file, since GitHub does not allow any simple way to just download a file(?). Since Quarto/qmd files are just text files, this is hopefully not too inconvenient.

Examples of output from the Quarto files are [also available](https://github.com/pgmj/RISEkbmRasch/tree/main/Quarto/output). Quarto can output multiple types of documents based on the same script, which is illustrated using the RaschR1.qmd file to output both a HTML scrollable file with side menu, and a revealjs presentation (also using HTML). The HTML files need to be downloaded and opened with a web browser.

### Notes on known issues

There are currently no checks on whether data input in functions are correct. This means that you need to make sure to follow the instructions above, or you may have unexpected outputs or difficult to interpret error messages. Start by using the functions for descriptive analysis and look closely at the output, which usually reveals mistakes in data coding or demographic variables left in the item dataset.

If there is too much missingness in your data, some functions may have issues or take a lot of time to run. In the Quarto template file there is a script for choosing how many responses a participant needs to have to be included in the analysis. You can experiment with this if you run in to trouble. Currently, the `RIloadLoc()` function does not work with any missing data (due to the PCA function), and the workaround for now is to run this command with `na.omit()` around the dataframe (ie. `RIloadLoc(na.omit(df))`. Other reasons for functions taking longer time to run is having a lot of items (30+), and/or if you have a lot of response categories that are disordered (commonly happens with more than 4-5 response categories, especially if they are unlabeled in the questionnaire).

The `RIitemfitPCM2()` function that makes use of multiple random subsamples to avoid inflated infit/outfit ZSTD values and runs on multiple CPU's will fail if there is a lot of missing data or very few responses in some categories. Increasing the sample size and/or decreasing the number of parallel CPUs can help, or just revert to the function `RIitemfitPCM()` that only uses one CPU.

### If you don't use the Quarto template

Since this is work in progress, including structuring the package properly, you will need these lines of code in your .R-file (they are also included in the Quarto template) for all RI* functions to work:

```r
library(ggrepel)
library(car)
library(ggrepel)
library(grateful)
library(kableExtra)
library(readxl)
library(tidyverse)
library(eRm)
library(mirt)
library(psych)
library(ggplot2)
library(psychotree)
library(matrixStats)
library(reshape)
library(knitr)
library(cowplot)
library(formattable)
library(glue)
library(RISEkbmRasch)

### some commands exist in multiple packages, here we define preferred ones that are frequently used
select <- dplyr::select
count <- dplyr::count
recode <- car::recode
rename <- dplyr::rename

### set up color palette based on RISE guidelines
RISEprimGreen <- "#009ca6"
RISEprimRed <- "#e83c63"
RISEprimYellow <- "#ffe500"
RISEprimGreenMid <- "#8dc8c7"
RISEprimRedMid <- "#f5a9ab"
RISEprimYellowMid <- "#ffee8d"
RISEprimGreenLight <- "#ebf5f0"
RISEprimRedLight <- "#fde8df"
RISEprimYellowLight <- "#fff7dd"
RISEcompPurple <- "#482d55"
RISEcompGreenDark <- "#0e4e65"
RISEgrey1 <- "#f0f0f0"
RISEgrey2 <- "#c8c8c8"
RISEgrey3 <- "#828282"
RISEgrey4 <- "#555555"

# set some colors used later
cutoff_line <- RISEprimRed
dot_color <- "black"
backg_color <- RISEprimGreenLight

# set fontsize for all tables
r.fontsize <- 15

### pre-set our chosen cut-off values for some commonly used indices:
### these are used to create highlight colors in tables
msq_min <- 0.7
msq_max <- 1.3
zstd_min <- -2
zstd_max <- 2
loc_dep <- 0.2 # above average residual correlation
dif_dif <- 0.5 # logits difference between groups in average item location (DIF)

### zstd is inflated with large samples (N > 500). Reduce sample size to jz and 
### run analysis yz random samples to get average ZSTD
jz = 300 # number to include in dataset
yz = 10 # number of random samples
```


## Author

[Magnus Johansson](https://www.ri.se/en/person/magnus-p-johansson) is a licensed psychologist with a PhD in behavior analysis, working at [RISE Research Institutes of Sweden](https://ri.se/en).
- Twitter: [@pgmjoh](https://twitter.com/pgmjoh)
- ORCID: [0000-0003-1669-592X](https://orcid.org/0000-0003-1669-592X)

## License

This work is licensed under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).
