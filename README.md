# ACME: King County Home Analysis
*Author:* [Robert Harrow](mailto:rharrow928@gmail.com)

## Overview

This project analyzes King County House Sales dataset for the benefit of ACME, a real estate investment firm entering the King County market. This analysis helps ACME decide on a renovation strategy once it has acquired a property in King County. Through the use of linear regression and hypothesis testing, we're able to see whether renovations help home price, and if so which specific renovations have the biggest impact on price.

## Business Problem

This analysis focuses on answering the following business questions:

1. Will renovations have a meaningful impact on home price?
2. If so, which renovations will have the most outsized impact on price?
3. How much should ACME spend on renovations to still be profitable? 

## Data

Data for this analysis came from two sources:

* [King County House Sales dataset from Kaggle](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction)
* [Zipcode-to-city data from ciclt.net](https://www.ciclt.net/sn/clt/capitolimpact/gw_ziplist.aspx?FIPS=53033) 

The data contains records on 21,420 homes in King County, Washington. The zipcode data maps zip codes to the municipality.

The King County data came with a dictionary of features:


| **Column**        | Description                                                                      | Unique | Dtype   |
|:-----------------:|:--------------------------------------------------------------------------------:|:------:|:-------:|
| **id**            | Unique identifier for a house                                                    | 21420  | int64   |
| **date**          | Date house was sold                                                              | 372    | object  |
| **price**         | Sale price                                                                       | 3622   | float64 |
| **bedrooms**      | Number of bedrooms                                                               | 12     | int64   |
| **bathrooms**     | Number of bathrooms                                                              | 29     | float64 |
| **sqft_living**   | Square footage of living space in the home                                       | 1034   | int64   |
| **sqft_lot**      | Square footage of the lot                                                        | 9776   | int64   |
| **floors**        | Number of floors (levels) in house                                               | 6      | float64 |
| **waterfront**    | Whether the house is on a waterfront                                             | 2      | object  |
| **view**          | Quality of view from house                                                       | 5      | object  |
| **condition**     | How good the overall condition of the house is. Related to maintenance of house. | 5      | object  |
| **grade**         | Overall grade of the house. Related to the construction and design of the house. | 11     | object  |
| **sqft_above**    | Square footage of house apart from basement                                      | 942    | int64   |
| **sqft_basement** | Square footage of the basement                                                   | 304    | object  |
| **yr_built**      | Year when house was built                                                        | 116    | int64   |
| **yr_renovated**  | Year when house was renovated                                                    | 70     | float64 |
| **zipcode**       | ZIP Code used by the United States Postal Service                                | 70     | int64   |
| **lat**           | Latitude coordinate                                                              | 5033   | float64 |
| **long**          | Longitude coordinate                                                             | 751    | float64 |
| **sqft_living15** | The square footage of interior housing living space for the nearest 15 neighbors | 777    | int64   |
| **sqft_lot15**    | The square footage of the land lots of the nearest 15 neighbors                  | 8682   | int64   |


### Distribution of price data across King County

<iframe
  src="/images/price_colormap.html"
  style="width:100%; height:300px;"
></iframe>

### How data is correlated with the target (price)

![correlation](/images/correlation-graph.png)

### Closer look at correlation of sqft_living and price

![sqft-living-price-correlation](/images/sqft-living-price-correlation.png)

### How is data distributed?

![data distribution](/images/data-distributions-scatter-plots.png)


## Methods

This project revolved around using linear regression to study the effects of different home features on home price.

First, we cleaned the data removing duplicates and missing values. I then merged the Zipcode-to-city dataset with the King County home dataset.

While the data came with 20 features, the analysis focused on **changeable** features. Because the business problem at hand revolves around what types of renovations (if any) ACME should carry out, it seemed relevant to focus on features that can be affected by renovations. For those reasons these features were studied:

* price (our target dependent variable)
* bedrooms
* bathrooms
* sqft_living
* floors
* condition
* grade
* sqft_above
* sqft_basement
* city (while not alterable by renovations, we kept city data in the dataset to loop over different regions when doing our analysis)

## Linear Regression

### Model Performance

This analysis iterated through 3 models.

#### Model 1

The first simply looked at the most correlated feature (sqft_living) and modeled price using it.

**Baseline Train score**: 0.45851217766070645
**Baseline Validation score**: 0.4492314854627397

#### Model 2

We then transformed and normalized our data and threw in all relevant features into our model. It performed better.

**Second Model Train score**: 0.5496942177332326
**Second Validation score**: 0.5477594474499472

#### Model 3

Next, we identified that there is Multicollinearity between sqft_living and sqft_above. We eliminated sqft_above from our dataset, and re-ran a third model.

**Third Model Train score**: 0.5424128964000199
**Third Validation score**: 0.5406753638986688

### Model 4 (Final Model)

For our final modeling attempt, we recognized that the region of a home playes a huge role in price. Therefore, we wanted to run models for each major region in King County to better isolate the effects of features.

I performed cross validation and tested this model against a test set to look at both accuracy and variance of the model. Here were the results:

|   Region 	| R-Squared Train 	| R-Squared Test 	| STD Train 	| STD Test 	|
|---------:	|----------------:	|---------------:	|----------:	|----------	|
|  Seattle 	|        0.573522 	|       0.566387 	|  0.007928 	| 0.032994 	|
|    Other 	|        0.602676 	|       0.597933 	|  0.004442 	| 0.017260 	|
|     Kent 	|        0.687568 	|       0.640317 	|  0.017286 	| 0.083821 	|
|   Renton 	|        0.704511 	|       0.678370 	|  0.006868 	| 0.030714 	|
| Bellevue 	|        0.730935 	|       0.704407 	|  0.010963 	| 0.040553 	|


And here is the average cooeficient for each of these tests:

| Feature                    	| Coefficient Value 	|
|----------------------------	|-------------------	|
| GradeCat_Excellent         	|          2.891178 	|
| sqft_living                	|          1.811653 	|
| GradeCat_Good_to_Very_Good 	|          1.371894 	|
| BedCount_Two               	|          1.279032 	|
| FloorCount_More_Than_Two   	|          1.148985 	|
| BedCount_One               	|          1.146430 	|
| CondCat_Good_to_Very_Good  	|          1.136920 	|
| BedCount_Three             	|          1.063210 	|
| FloorCount_Two             	|          0.996269 	|
| BathCount_More_Than_Two    	|          0.994786 	|
| BathCount_Two              	|          0.992005 	|
| GradeCat_Below_Average     	|          0.929721 	|
| BedCount_More_Than_Four    	|          0.929097 	|
| HasBasement_Yes            	|          0.914309 	|
| CondCat_Below_Average      	|          0.762570 	|

After running models for 3 regions, we're pretty consistently getting very high p-values for features related to the number of floors and the number of bedrooms. For that reason, we will skip over that feature when making recommendations.

### Regression Diagnostics

Checking for linearity, we see that Bellevue and Kent fail the linearity assumption. Therefore, it will be hard to trust the results in these regions.

![renton](/images/renton-qq-plot.png)
![seattle](/images/seattle-qq-plot.png)
![bellevue](/images/bellevue-qq-plot.png)
![kent](/images/kent-qq-plot.png)
![other](/images/other-areas-qq-plot.png)

### Checking for Heteroscedasticity

Next, I checked for heteroscedasticity using the Breusch-Pagan Test. Here were the results:

**The null hypothesis (H0)**: Homoscedasticity is present.

**The alternative hypothesis (Ha)**: Homoscedasticity is not present (i.e. heteroscedasticity exists)

|  Region  	| Lagrange multiplier statistic 	|  P-Value  	|
|:--------:	|:-----------------------------:	|:---------:	|
|  Renton  	|               55              	|  1.13e-06 	|
|  Seattle 	|              190              	|  3.43e-33 	|
| Bellevue 	|               32              	|   0.0023  	|
| Kent     	|               65              	| 1.294e-08 	|
| Other    	|              230              	| 2.01e-41  	|

The p-value is well below 0.05 for each region, so we can reject the null hypotheses. Therefore, heteroscedasticity is present everywhere.

## Results

After studying the above coefficients from our linear regression, we were able to draw some conclusions.

* Increasing the square feet of the living area by one standard deviation will increase the home price by 1.8 standard deviations
* Making the home grade 'excellent' or better is expected to increase the home price by 2.9 standard deviations
* Across all regions, the average y-intercept value is $262,278.21. In other words, that is the base home value when all other factors are set to 0.

![average-price](/images/average-home-price-renovated-not-renovated.png)

![Effect of home grade on price](/images/grade-price-effect.png)

![Effect of sqr_foot_living on price](/images/sqft-foot-living-area-price-line.png)

## Conclusions

This analysis has lead to three recommendations for ACME as they look at investing in real estate in King County:

* It’s recommended that ACME renovate the homes it buys in King County
* Focus on expanding the square footage of the living area, improving the build grade to ‘Excellent’ with King County assessors
* Keep renovation costs:
  * below \$1,130,833 for rising building grade to 'Excellent'
  * below \$664,751 for expanding sq. ft by 915

## For more information

See the full analysis in the [Jupyter Notebook](https://github.com/robertharrow/flatiron-phase-2-project/blob/main/data-analysis.ipynb) or review [this presentation](https://github.com/robertharrow/flatiron-phase-2-project/blob/main/presentation.pdf).

For additional info, contact Robert Harrow at rharrow928@gmail.com.

### Repository Structure
```
├── data
├── images
├── README.md
├── presentation.pdf
└── data-analysis.ipynb
```
