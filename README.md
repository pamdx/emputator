# Getting started

## Prerequisites

To run the tool, you need the following installed on your computer.
- A recent [R](https://cran.r-project.org/) installation (version 4.1.0 was used to build the tool)
- A recent [RStudio](https://www.rstudio.com/products/rstudio/#rstudio-desktop) installation (version 1.4.1103 was used to build the tool)
- The following R packages installed: dplyr, ggplot2, readr, tidyr, tibble, compareDF, stargazer, gridExtra, Rilostat, OECD. You can install these packages by running the R code below:  
``` R
install.packages(
  c("dplyr", "ggplot2", "readr", "tidyr", "tibble", "compareDF", "stargazer", "gridExtra", "Rilostat", "OECD")
  )
```

## Installation

1. Extract the contents of the compressed folder to the destination of your choice (e.g. on your Desktop).
2. Double-click the "emputator.Rproj" file in the main folder.
3. The tool will open in RStudio.

# How to use the imputation tool

## Loading basic packages, functions and data

First, in the main.R script, run the code block below by selecting it and pressing CTRL+ENTER. It will load the packages, functions and data necessary for the tool to run properly.
``` R
rm(list=ls()) # clear R environment

### Load packages

library(dplyr)
library(ggplot2)
library(readr)
library(tidyr)
library(tibble)
library(compareDF)
library(stargazer)
library(gridExtra)

theme_set(theme_bw()) # set simple theme for charts

# Load app functions

source("./modules/functions.R")

# Data import

source("./modules/data_import.R")
```

## Setting filtering parameters and generating a chart of existing data

The next step is to tell the tool which country, sector and period needs to be imputed. Specify those in the code block below, and then run it to generate a visualization of the existing official and estimated data. The country input must be one of the values listed in the "Name_En" column from this ![FAO country reference](https://github.com/openfigis/RefData/blob/gh-pages/country/CL_FI_COUNTRY_ITEM.csv). The sector input must be one of the following values: "Aquaculture", "Inland fishing", "Marine fishing", "Subsistence", "Unspecified", or "Processing".

``` R
# Main filtering

country_input <- "France"
OC2_input <- "Marine fishing"
start_year <- 1995
end_year <- 2020

# Subseries-related analyses

source("./modules/subseries_analysis.R")
```
A "rainbow" bar chart like the one below will be displayed in the Plots panel of RStudio.
![current_estimates](https://user-images.githubusercontent.com/59026485/144248477-c1f24364-1aed-4107-8440-fd7aed5af92f.png)

## Setting imputation parameters

Next, the user can change the code below to modify how estimates are calculated. Regardless of whether the code is modified, it needs to be run, otherwise the imputation scripts will fail.

``` R
#### Estimations parameters ####

  # Linear regression

share_valid_reg <- 0.5 # Proportion of years with official data necessary to run regression (to avoid generating estimates from too little information)
obs_threshold_linreg <- round(length(years_all) * share_valid_reg) # Do not modify
reg_type <- 1 # Regression type 1 = automatic (runs predetermined models and selects the one with best fit), 2 = manual (see below)
trend <- seq(start_year:end_year) # Do not modify
reg_dynamic <- emp_value ~ prod_value + labor_value # Specify manual regression by choosing independent variables from: trend, prod_value, labor_value, fleet_value (separated by "+")
fit_threshold_reg <- 0.8 # R2-squared threshold for the regression to be taken in consideration

  # Polynomial trends

share_valid_trend <- 0.5 # Proportion of years with official data necessary to run regression (to avoid generating estimates from too little information)
obs_threshold_trend <- round(length(years_all) * share_valid_trend) # Do not modify
fit_threshold_trend <- 0.8 # R2-squared threshold for the regression to be taken in consideration

  # Historical growth/average

histavg_threshold <- 5 # Number of previous years on which to base historical average estimates
histgrowth_threshold <- 5 # Number of previous years on which to base historical growth  estimates
```

## Choosing the imputation mode

There are two ways to perform the imputation of missing values: 
- by generating aggregated imputed values for years with no data, which are then disaggregated based on the weights of subseries for years with official data ("aggregated imputation", most convenient and suitable for most cases) 
- by imputing one subseries at a time ("subseries imputation", more suitable in cases where only some subseries need to be estimated for a given year)

### Aggregated imputation

To run the aggregated imputation, first execute the code below. 

``` R
# Aggregated imputation

source("./modules/processing_aggregated.R")
```

This will create visualizations of the results of each of the imputation method available for the time series at hand:
- charts of the covariates available for the linear regression:
![covariates](https://user-images.githubusercontent.com/59026485/144248802-f7338142-665c-4b68-a002-d0303fab7656.png)
- a chart of the best-fitting linear model:
![linear_fit](https://user-images.githubusercontent.com/59026485/144248859-b7a95541-f2f4-423e-b35b-d33b5e8368a4.png)
- a chart of the best-fitting polynomial trend:
![polynomial_fit](https://user-images.githubusercontent.com/59026485/144248912-6d749b7f-e622-4574-b98e-74f9a7ee10d1.png)
- a "rainbow" chart for each available imputation method (below is an example for the linear interpolation results):
![linearint_results](https://user-images.githubusercontent.com/59026485/144249076-dd193235-52b6-41be-814e-faefa73a6165.png)

Then, to launch the imputation prompt, run the code below. 

```
agg_imputation_type <- 1 # 1 = "Consecutive", 2 = "Year-by-year"

source("./modules/imputation_aggregated.R")
```

Note that you can choose between two types of aggregated imputations by setting the "agg_imputation_type" variable: 
- choose an imputation method for each series of consecutive years with missing data ("Consecutive", most convenient and suitable for most cases)

![agg_imp_cons](https://user-images.githubusercontent.com/59026485/144249777-dce5ca5d-6e61-42ba-8fa5-c2cbb099de37.png)

- choose an imputation method for each year with missing data ("Year-by-year", rarely used)

![agg_imp_yby](https://user-images.githubusercontent.com/59026485/144249952-bbbe337a-9975-4e96-908d-667091fb5c59.png)

### Subseries imputation

To run the subseries imputation, execute the code below. Note that if you run the subseries imputation after the aggregated imputation, the results of the aggregated imputation will be replaced by those of the subseries imputation.

``` R
# Subseries imputation

source("./modules/imputation_subseries.R")
```
A first prompt will ask you to select the subseries you want to impute. Select one subseries and click OK.

![subs_imp_1](https://user-images.githubusercontent.com/59026485/144250351-1cc9d44f-a8e1-4862-8013-6339e718ce4b.png)

A second prompt will ask you what imputation method should be applied to the subseries selected. Select one method and click OK.

![subs_imp_2](https://user-images.githubusercontent.com/59026485/144250534-425474c9-4aa1-40b5-a7af-28e018e5b235.png)

Finally, a third prompt will ask you what year should be imputed. You can select multiple year by pressing CTRL or SHIFT. Click OK to confirm your selection.

![subs_imp_3](https://user-images.githubusercontent.com/59026485/144250614-a2290124-bc06-4e24-af2b-64b207190b15.png)

A "rainbow" chart of the current state of subseries imputation will be displayed in RStudio. The first prompt will reappear in case you want to continue the process with other subseries. If you are done with the imputation, select "Stop imputation" and click OK.

## Exporting the imputed data and imputation report

To export the imputed data (CSV) and the imputation report (HTML) in the outputs folder, run the following code block in the main.R script.

``` R
# Final data export and report generation

source("./modules/final_data_export_viz.R")
```

# Description of the imputation methods

## Linear regression

[Linear regression](https://en.wikipedia.org/wiki/Linear_regression) in this tool is implemented with the lm() function from base R. 

In automatic regression mode, the following linear models are fitted to the data, and the model with the highest adjusted R-squared is selected to generate the estimates.

| Model | Specification 1                              | Specification 2                                            |
|-------|----------------------------------------------|------------------------------------------------------------|
| 1     | emp_value ~ trend + prod_value + labor_value | emp_value ~ trend + prod_value + labor_value + fleet_value |
| 2     | emp_value ~ prod_value + labor_value         | emp_value ~ prod_value + labor_value + fleet_value         |
| 3     | emp_value ~ trend + prod_value               | emp_value ~ trend + prod_value                             |
| 4     | emp_value ~ prod_value                       | emp_value ~ prod_value                                     |

where
- Specification 2 is used for imputation of fishery employment in OECD countries only (the fleet data is currently only available for OECD countries).
- Specification 2 is used in all other cases.
- "trend" is the sequence of the years composing the time series being imputed. Including a [trend variable](https://en.wikipedia.org/wiki/Linear_trend_estimation) may increase the linear fit of time series that exhibit a clear upward or downward trend over time.

In manual regression mode, the user specifies the linear model to be fitted to the data. A tilde (~) should be used to separate the dependent variable from the independent variable(s), and multiple independent variables should be separated by a plus (+) sign. The independent variable should always be "emp_value", while the independent variable(s) can be chosen among "trend", "prod_value", "labor_value" and "fleet_value".

Please note that the linear regression method is not available in subseries imputation mode. This is to avoir fitting a linear model with only a subset of employment as a dependent variable, whereas the dependent variables represent the entirety of the production, fleet or labour force.

## Polynomial trends

The polynomial trends estimates are generated by [polynomial regression](https://en.wikipedia.org/wiki/Polynomial_regression) implemented with the lm() and poly() functions from base R and the stats package, respectively. In these regressions, the employment value is the dependent variable, while the years are the independent variable. Regressions are automatically fitted for the first, second, third and fourth degree polynomials. These amount to fitting a linear, quadratic, cubic and quartic trend to the employment data.

The trend with the highest adjusted R-squared (the one that fits the data most closely) is automatically selected to generate the imputed data.

## Linear interpolation

[Linear interpolation](https://en.wikipedia.org/wiki/Linear_interpolation) estimates are generated with the following function: 

![linear interpolation function](https://wikimedia.org/api/rest_v1/media/math/render/svg/8cd1838ecabafa019029fe20e05265378d042e16)

where
- y is the employment value to estimate
- x is the year associated with the value to estimate
- x0 is the reference year (i.e. with official data) on the left side of x
- x1 is the reference year on the right side of x
- y0 is the official employment value associated with x0
- y1 is the official employment value associated with x1

## Historical average

Historical average estimates of employment are the mean of the x previous consecutive years of official data, where x is a positive integer defined by the user (by default, 5). Note that these estimates will not be generated if less than x previous consecutive years of official data are available.

## Historical growth

Historical growth estimates of employment are an extrapolation based on the [compound annual growth rate](https://en.wikipedia.org/wiki/Compound_annual_growth_rate) (CAGR) of the x previous consecutive years of official data, where x is a positive integer defined by the user (by default, 5). Note that these estimates will not be generated if less than x previous consecutive years of official data are available.

## Backward dragging

As their name suggests, backward dragged estimates impute missing employment data with the closest official value from a posterior year. This method is most commonly applied to the years at the beginning of the time series.

## Forward dragging

As their name suggests, forward dragged estimates impute missing employment data with the closest official value from an anterior year. This method is most commonly applied to the years at the end of the time series.

# Contents of the extracted folder

## Main folder
This folder includes the files that run the imputation tool:

| File            | Type           | Description                             |
|-----------------|----------------|-----------------------------------------|
| emputator.Rproj | R project file | Opens the tool in RStudio.              |
| main.R          | R script       | Main script from which the tool is run. |

## Inputs folder
The inputs folder contains the data necessary for the imputation tool:

| File           | Type        | Description                                                                                                             |
|----------------|-------------|-------------------------------------------------------------------------------------------------------------------------|
| FM_DB.rds      | R data file | Contains the up-to-date FAO-OECD employment database on which to perform the imputation. Converted from a CSV file with the inputs_update.R module.         |
| ILO_labor.rds  | R data file | Contains the ILO labor force database to be used in linear models. Retrieved from the ILO's servers with the Rilostat package.   |
| OECD_fleet.rds | R data file | Contains the OECD fleet database to be used in linear models. Retrieved from the OECD's servers with the OECD package.           |
| PROD.rds       | R data file | Contains the FAO capture and aquaculture production database to be used in linear models and productivity computations. Retrieved from FAO's servers with a custom function.|

The FM_DB.rds file should have the following structure:

| Column          | Type      | Accepted values                                                                                                                                                                                                                                                                              |
|-----------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| geographic_area | character | Values listed in the "Name_En" column from this ![FAO country reference](https://github.com/openfigis/RefData/blob/gh-pages/country/CL_FI_COUNTRY_ITEM.csv)                                                                                                                                                                                                                        |
| OC2             | character | "Aquaculture", "Inland fishing", "Marine fishing", "Subsistence", "Unspecified", "Processing"                                                                                                                                                                                                |
| OC3             | character | "Aquaculture", "Inland Waters Fishing", "Marine Coastal Fishing", "Marine Deep-Sea Fishing", "Marine Fishing, nei", "Subsistence", "Unspecified", "Processing"                                                                                                                               |
| working_time    | character | "Full time", "Part time", "Occasional", "Status Unspecified"                                                                                                                                                                                                                                 |
| sex             | character | "M", "F", "U"                                                                                                                                                                                                                                                                                |
| year            | integer   | Any year between 1950 and the current year                                                                                                                                                                                                                                                   |
| value           | integer   | Any positive integer, or blank if accompanied by an "M" or "Q" flag                                                                                                                                                                                                                          |
| flag            | character | (Blank) = Official figure, "B" = Break in time series, "E" = FAO estimate, "I" = Estimate from the reporting country, "M" = Missing value (data cannot exist, not applicable), "P" = Provisional data, "Q" = Confidential data, "T" = Data reported by non-official or semi-official sources |
| comment         | character | Blank or any text providing background on the entry                                                                                                                                                                                                                                          |

## Modules folder
The modules folder contains the R scripts that are necessary for the imputation tool to perform its computations and produce the desired outputs.

| File                    | Type            | Description                                                                                                                                                                            |
|-------------------------|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| data_import.R           | R script        | Imports data from the inputs folder into the R environment.                                                                                                                            |
| final_data_export_viz.R | R script        | Creates the HTML imputation report and CSV file of the imputed data and saves them in the outputs folder.                                                                              |
| functions.R             | R script        | Includes all the functions that the tool uses to generate imputed data, charts, tables, etc.                                                                                          |
| imputation_aggregated.R | R script        | Runs the imputation in aggregated mode.                                                                                                                                                |
| imputation_subseries.R  | R script        | Runs the imputation by subseries.                                                                                                                                                      |
| inputs_update.R         | R script        | Updates the data located in the inputs folder: online databases from FAO, the OECD and ILO are queried to retrieve the latest production, fleet and labour force data.                 |
| processing_aggregated.R | R script        | Generates all the objects necessary to run the imputation in aggregated mode: tables with imputed data and their associated "rainbow" bar charts.                                      |
| report.Rmd              | R Markdown file | Generates the HTML imputation report.                                                                                                                                                  |
| subseries_analysis.R    | R script        | Performs a series of basic computation from the data: visualizes the existing estimates, identifies years with missing data, computes the weight of each subseries for each year, etc. |

## Outputs folder
The outputs folder contains the imputation results by country and sector. For each country/sector processed by the tool, an HTML report and a CSV file of the imputed data are saved in this folder.

| File            | Type           | Description                             |
|-----------------|----------------|-----------------------------------------|
| [country]_[sector]_report.html | HTML document | Summary of the imputation process and results.              |
| [country]_[sector]_imputed.csv          | CSV file       | Imputed time series |
