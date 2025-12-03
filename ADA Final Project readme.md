# ADA Final Project: NHANES Data Analysis  

## Project Description  
This repository includes the R Markdown file used to analyze NHANES data for the ADA final project.  
The project uses NHANES 2021–2023 exam and lab data to explore whether **total cholesterol (mg/dL)** is associated with **elevated HbA1c in people who havent been told to have diabetes (undiagnosed diabetes)** among U.S. adults.  
The code practices working with real-world complex survey data, missing data, multiple imputation, and regression modeling in R Markdown.  

## Files Included  
`ADA Final Project.Rmd`: R Markdown file used to clean, analyze, and model the NHANES data  
`README.md`: This file  

## What the Code Does  

#Chunk 1 – Setup

Sets global knitr options so code and output show up the way we want (`echo = TRUE`).

#Chunk 2 – Install packages

Installs the R packages used in the project (only really needed the first time you run the file).

#Chunk 3 – Load packages

Loads all the packages for data wrangling, survey work, imputation, modeling, tables, and graphics (e.g., `tidyverse`, `haven`, `survey`, `srvyr`, `mice`, `table1`, `DiagrammeR`, etc.).

#Chunk 4 – Read in the data

Reads the merged NHANES exam/lab file from a local path into an object called `mydata`.

#Chunk 5 – Define the analytic sample

Filters the full NHANES data to create `mydata2`, keeping:
 Adults (age ≥ 18)
 Not pregnant
 People with non-missing HbA1c
 People who report "no" diabetes.

#Chunk 6 – Create outcome and BMI categories, clean education

Creates a binary outcome `hba1c_Bin` (HbA1c ≥ 6.5 vs < 6.5).
Creates BMI categories (underweight, normal, overweight, obese) from continuous BMI.
Recodes education so “7” and “9” are treated as missing.
Saves this as `mydata_r`.

#Chunk 7 – Check missingness in key variables

Defines the main analysis variables (HbA1c, cholesterol, age, sex, race, education, BMI).
Computes how many values are missing and the percent missing for each of these variables.
Stores the result in `missing_analytic_long`.

#Chunk 8 – Pick variables for analysis and imputation

Defines:
 A core set of analytic variables.
 Extra variables that will only be used to help with imputation (auxiliary variables).
 Checks which of these actually exist in the data and stores them in `vars_keep`.

#Chunk 9 – Build the final analytic dataset (`mydata3`)

Subsets `mydata_r` down to the variables in `vars_keep`.
Renames variables to simpler names (e.g., `LBXGH` → `hba1c`, `LBXTC` → `chol`, `RIDAGEYR` → `age`, etc.).
Prints `names(mydata3)` so you can see the final variable list.

#Chunk 10 – Look at missing data patterns

Runs `md.pattern(mydata3)` to see which variables tend to be missing together and how many people fall into each missingness pattern.

#Chunk 11 – Plot missingness vs. another variable

Uses `pbox(mydata3, pos = 3)` to compare the distribution of one variable across missing vs non-missing groups of the others (a quick visual check of missingness patterns).

#Chunk 12 – Boxplot of cholesterol by HbA1c status

Makes a boxplot of total cholesterol by the binary HbA1c variable to see how cholesterol levels differ between people with and without elevated HbA1c.

#Chunk 13 – Run multiple imputation

Calls `mice()` on `mydata3` to impute missing values in the selected variables.
Sets number of imputations, iterations, method (`pmm`), and a seed.
Saves the imputation object as `mydata_imp`.

#Chunk 14 – Inspect imputed values

Checks which variables were imputed (`names(mydata_imp$imp)`).
Looks at the imputed values for cholesterol, BMI, and education to make sure they are reasonable.

#Chunk 15 – Pull one completed dataset

Uses `complete(mydata_imp, 1)` to grab the first fully imputed dataset.
Keeps only the analysis variables (outcome + predictors) and saves this as `imp1`.

#Chunk 16 – Regression model on imputed data

Fits a regression model of `hba1c_Bin` on cholesterol, age, sex, race, education, and BMI using all imputed datasets (`with(mydata_imp, glm(...))`).
Pools the estimates across imputations with `pool()`.
Uses `tidy()` to get exponentiated coefficients and confidence intervals (odds ratios).

#Chunk 17 – Regression and diagnostics on a single imputed dataset

Fits the same model on `imp1` (just one completed dataset).
Saves the fitted model as `fit_imp1` and prints the summary.

#Chunk 18 – Model diagnostics

Pulls the observed outcomes and fitted values from `fit_imp1`.
Checks multicollinearity with VIF.
Runs a Hosmer–Lemeshow goodness-of-fit test.
Draws a ROC curve and calculates AUC.
Looks at Cook’s distance and plots it to spot influential observations.

#Chunk 19 – Interaction with sex (imputed data)

Fits a model that includes an interaction term between cholesterol and sex.
Pools the results across imputations.
Uses `tidy()` to get the pooled results and then pulls out the interaction term to see if it’s significant.

#Chunk 20 – Interaction with race (imputed data)

Fits a model with an interaction between cholesterol and race.
Pools the results and uses `tidy()` to look at the interaction terms for race categories.

#Chunk 21 – Build a complete-case dataset

Creates `cc_data` by keeping only rows with no missing values in the main analytic variables.
Prints the number of rows before and after to show how many cases are dropped.

#Chunk 22 – Regression on complete cases

Fits the same regression model using only `cc_data`.
Uses `tidy()` to get exponentiated coefficients and confidence intervals for the complete-case analysis.

#Chunk 23 – Interaction with sex (complete cases)

Fits a complete-case model with a cholesterol × sex interaction term.
Uses `tidy()` and then pulls the interaction term to see if the effect differs by sex.

#Chunk 24 – Interaction with race (complete cases)

- Fits a complete-case model with a cholesterol × race interaction term.
- Uses `tidy()` and then pulls the interaction terms for race.

#Chunk 25 – Descriptive table

Makes a copy of `mydata3` for descriptive stats.
Recodes sex, race, and education into readable factor labels.
Adds variable labels.
Uses `table1()` to create a descriptive table of cholesterol, age, BMI, sex, race, and education by undiagnosed diabetes status.

#Chunk 26 – Flowchart

Uses `DiagrammeR::grViz()` to draw a simple flowchart:
 Starting NHANES sample size.
 Exclusions (age, pregnancy, missing HbA1c, self-reported diabetes).
 Final analytic sample size.
  
## Author  
 Name: Michael Tebeje  
 Course: Advanced Data Analysis (ADA)  