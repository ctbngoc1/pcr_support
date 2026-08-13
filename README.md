# Predicting Hospital Length of Stay Using PCR

## Overview

This project investigates the use of Principal Component Regression (PCR) to model and predict hospital length of stay for critically ill patients as part of a university course assignment. By applying dimensionality reduction through Principal Component Analysis (PCA), the study aims to address multicollinearity among predictors and improve model stability and generalization. The code was developed and executed using RStudio.

## Data

This project uses the SUPPORT2 dataset, which contains 9105 observations and 47 variables, including clinical, demographic, and physiological information on critically ill patients admitted to five medical centers in the United States. The dataset is publicly available at: <https://www.kaggle.com/datasets/joebeachcapital/support2>

The outcome variable of interest is **slos**, which represents the number of days from study enrollment until hospital discharge. All remaining variables were treated as potential predictors in the modeling process.

Predictor variables derived from existing prognostic models (*aps, sps, surv2m, surv6m, scoma*) were excluded since they aren't raw clinical measurement. Additionally, the *adls* variablewas also removed for containing information almost similar to the *adlsc* variable. Categorical variables (*sex, dzgroup, dzclass, income, race, ca, dnr, sfdm2, death, hospdead, diabetes, dementia*) were converted into factor variables.

The dataset contains 27 variables with missing values, which were handled as follows:

- Variables with approximately 50% missing values (*glucose, bun, urine, adlp*) were excluded.

- Variables with minimal missingness (*meanbp, hrt, resp, temp, sod*) were assumed Missing Completely At Random (MCAR) and imputed using mean imputation.

- Selected laboratory variables (*alb, pafi, bili, crea, wblc*) were imputed with clinically reasonable reference values.

- Categorical MCAR variables (*race, dnr)* were imputed using mode imputation; Numeric MCAR variable *dnrday* was imputed using mean imputation.

- Remaining variables with Missing At Random (MAR) patterns were imputed using Multiple Imputation by Chained Equations (MICE) with the Classification And Regression Trees (CART) method.

Outliers were examined but retained, as extreme values may reflect valid clinical conditions. The completed dataset was split into training (80%) and validation (20%) sets.

According to the Variance Inflation Factor (VIF) index, substantial multicollinearity exists between some predictors. Given the large number of variables (37 predictors) and the presence of multicollinearity, PCA was employed to reduce the dimensionality of the data and also to address the multicollinearity between variables. Moreover, since the means and variances of the variables differ significantly, PCA was performed on the correlation matrix instead of the covariance matrix, which is equivalent to standardizing each of the predictor variables. All factor variables were converted into numeric variables for PCA.

## Methods

PCA transforms the original predictors into a smaller set of *principal components (PCs)* in order to eliminate multicollinearity and reduce model complexity. The proportion of variance explained by each component and the cumulative explained variance were examined to guide component selection. *Principal Component Regression (PCR)* was then used to model the relationship between *slos* and the selected subset of principal components.

Two PCR models were considered: a model using the first 30 PCs, and a model in which PCs were selected via bidirectional stepwise selection using the Bayesian Information Criterion (BIC). The optimal model was selected based on performance on the training set.

Regression model assumptions were assessed using residual diagnostic plots, the Anderson-Darling normality test, the Breusch-Pagan test, the One Sample t-test, and the Durbin-Watson autocorrelation test. When violations were detected, a Box-Cox transformation of the response variable was considered and applied to improve model adequacy.

To examine the relationship between variables, the final PCR model was transformed back into a regression model expressed in terms of the original predictors. Afterwards, the predictors in the validation set were standardized using the same centering and scaling parameters derived from the training set, and the corresponding PCs were computed. The final PCR model was then used for predictions on the validation set.

## Results

From the original 36 predictor variables, PCA produced 36 PCs, with the first 30 PCs explaining 96.98% of the total sample variance. As expected, the Tolerance and VIF index confirmed the absence of multicollinearity among the PCs. The stepwise selection method produced a model with 31 predictor variables, including PCs from 1 to 36, excluding PCs number 15, 26, 33, 34, and 35. The PCR model using the first 30 PCs achieved an $R^{2}$ of 0.765, whereas the stepwise selection PCR model achieved a higher $R^{2}$ of 0.851 and was therefore chosen as the better model.

Assumption diagnostics indicated that the PCR model violated the linearity, normality and homoscedasticity assumption. Although the Box-Cox transformation did improve linearity, it didn't improve normality and homoscedasticity. The transformed PCR model was nonetheless retained for inference and prediction in this project. However, the inferential results should be interpreted with caution. Alternative modeling approaches may be more appropriate for prediction and drawing statistical inferences from the support2 dataset.

The selected PCR model explained a substantial proportion of variability in hospital length of stay, achieving an $R^{2}$ of 0.784. Longer hospital stays were associated with older age, longer time since surgery, higher disease severity, greater numbers of comorbidities and diagnoses, and increased utilization of hospital resources such as intensive care, procedures, and charges. Clinical and laboratory indicators of poorer health status - including elevated heart rate, respiratory rate, blood urea nitrogen, creatinine, and bilirubin - were linked to increased length of stay, while higher albumin levels were associated with shorter hospitalizations.

On the validation set, the PCR model managed achieved an RMSE of 11.602 and an MAE of 5.100, which is reasonable considering that *slos* has a wide range of values (from 3 to 223). Additionally, the model attained standardized RMSE of 0.053 and standardized MAE of 0.023, which indicates good predictive performance on the validation set.
