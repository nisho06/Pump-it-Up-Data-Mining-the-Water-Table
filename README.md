# Pump it Up Challenge

Github Link: https://github.com/Skogul1997/Pump-it-Up-Data-Mining-the-Water-Table

Competition Link: https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/

This repo contains the code used for submissions made on the competition Pump it Up: Data Mining the Water Table, hosted on DrivenData.

**Accuracy of best model  : 0.8220**

**Final Rank              : 740(As of 17th September 2021)**

Following are the steps followed to tackle this problem.

***

## 1) Exploratory Data Analysis
Data was explored initally using the *Pandas Profiling* library. The html file generated is attached. 

Then data was plotted and further explored. The following were the key findings;
- 6 columns contained missing values, out of which the scheme_name column contained 50% null values.
- There was a class imbalance in the dataset with the class "functional needs repair" only having 4317 points(7% of the dataset)
- There were several grouped columns(columns that had a hierarchical structure)
  - scheme_management, management and management_group
  - region, region_code, lga, ward, district_code, longitude, latitude [all focussing on location of the datapoint]
  - quantity and quantity_group
  - source, source_type and source_class
  - water_quality and quality_group
  - payment and pay
  - extraction_type, extrcation_type_group and extraction_type_class
  - waterpoint_type and waterpoint_type_group
- The recorded_by column only contained one value *GeoData Consultants Ltd* throughout.
- Installer column contains alot of spelling mistakes.
- num_private column doesn't have metadata(we don't know that the column is)
- amount_tsh column has alot of zero values(70%), this can't be true because if the total static pressure head is zero then there is no need for a pump.
- population column had zero value, this cannot be true because if there is 0 population in an area, then there is no need for a pump.

***

## 2) Pre-processing and Feature Engineering
The pre-processing steps can be abtracted in the following categories.
- Dropping grouped columns(due to data redundancy) The following columns were dropped.
  - scheme_management, management_group, region_code, lga, district_code, ward, subvillage, quantity_group, source_type, source_class, quality_group, payment_type, extraction_type, extraction_type_class, waterpoint_type_group
- Handling Missing Values and 0 values
  - In the longitude column, the 0 values were replace by the means of the region_code.
  - The null values of permit and public_meeting were filled with the mode.
  - Overall null values in categorical column was replaces by mode and null values in numerical column was replaced by the mean.
  - Some categorical columns the null values were replaces by the term "Unknown" and we saw accuracy improving due to this.
- Encoding was done using CatBoost.
- Z-score normalization was performed for numerical columns(but the accuracy dropped due to this therefore droppped it).

New features were created. 
- contruction_year was transformed to decade, indicating the decade of construction.
- The installer column was changed, values with count more than 71 were taken and the others were rename as "other".
- The funder column was also transformed in a similar fashion.
- Features were dropped due to redundancy,
  - There were several features that when grouped gave the same output, therefore these features were dropped.


***

## 3) Model and Fine-tuning
H2O and CatBoost were tried out.

- H2O was tried out. With the following parameters,
  - Train set was divided in to train, valid and test.
  - The dataframes were converted to H2O Frames.
  - All categorical features were converted to type "enum".
  - AutoML from H2O was used to fit the data with cross-validation.
- CatBoost was tried with the following parameters,
  - Train set was divided in to train, valid and test.
  - Categorical features were specified.
  - Model was trained for 5000 iterations.
  - Finally used the best model(best modelk chosen based on accuracy).
- Note: XGBoost and other sklearn models were tried out but did not give good accuracies.
  
Accordingly CatBoost gave better performance than H2O models. Therefore it was used.

***

## 4) Post-processing

The CatBoost model was used to generate SHAP values, which then were used to study feature importance. Accordingly quantity, extraction_type and payment proved to be very important whereas public_meeting, water_quality and permit turned out to be least important.


**Some Points to Consider**
- CatBoost classifier gave better results on CPU rather than on GPU. This issue can be found on the official repository of CatBoost [here](https://github.com/catboost/catboost/issues/241).
- When columns/ features were removed due to redundancy and the SHAP values, the accuracy seems to drop considerably. Akthough the features seem redundant, they do matter in the model built.
- The dataset itself is not clean, major issue being the amount of zeros. Zeros in some columns was considered as missing values. For example, when considering the longitude column, Tanzania doesn't have a point with a 0 value for longitude, therefore this point has to be outside of Tanzania, but this can't be true. There were similar issues in other columns such as latitude, amount_tsh etc.
- There was an issue with class imbalance in the dataset. In order to counter this class weights were added to the CatBoost Classifier, but the accuracy dropped due to this.

***

## Proof of submission

DrivenData username: moracse_170307M

![submission](https://github.com/Skogul1997/Pump-it-Up-Data-Mining-the-Water-Table/blob/main/proof.PNG)
