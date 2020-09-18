# Financial Wellbeing Analysis (Python)

## Project Overview 

### Objective 

The objective of this project is to understand the association between an individual's ownership specific financial products (e.g. savings account, health insurance) and their 'financial wellbeing', a measure of the extent to which one's financial situation provides security and freedom. In addition to exploring whether their is a correlation between ownership of one of these products and financial wellbeing, I will also investigate whether their might be a causal effect on financial wellbeing (e.g. owning a savings account 'causes' an individual's financial wellbeing to improve). 

As a measure financial wellbeing, I use The Consumer Financial Protection Bureau's Financial Well-Being Scale. The Financial Well-Being Scale scores individuals from 0-100 based on "the extent to which someone’s financial situation and the financial capability that they have developed provide them with security and freedom of choice". For more information on the scale, see [the CFPB website](https://www.consumerfinance.gov/data-research/research-reports/financial-well-being-scale/).

The financial products under consideration are: 

* Savings account
* Health insurance
* Retirement savings account
* Pension
* Investment
* Education savings account
* Student loan

### Results and Insight
* Financial products do not explain much of the variation in financial wellbeing 
* Retirement and investment accounts had the greatest (positive) effect on financial wellbeing. 
* Student loans, which had a negative effect, followed close behind
* The impact of all products was reduced once income, employment status, education and health status was accounte for 


## Code and Resources Used 
**Python Version:** 3.7

**Packages:** pandas, numpy, statsmodels, matplotlib, seaborn

## The Dataset 
For this project I used data from the CFPB's 2016 financial wellbeing survey, downloaded from [the CFPB website](https://www.consumerfinance.gov/data-research/financial-well-being-survey-data/). The survey dataset includes respondents’ scores on the CFPB Financial Wellbeing Scale, the financial products that they use as well as other measures of individual and household characteristics that research suggests may influence adults’ financial well-being (e.g. income and employment, education). 

The National Financial Well-Being Survey was conducted in English and Spanish via web mode between October 27, 2016 and December 5, 2016. Overall, 6,394 surveys were completed: 5,395 from the general population sample and 999 from an oversample of adults aged 62 and older.

## Data Cleaning 
After retrieving the CFPB survey data, I cleaned the data to improve the accuracy of my findings. Changes made to the raw dataset included: 

* **Removing irrelevant columns:** The survey collects around 200 characteristics of each respondent that go far beyond their financial wellbeing scores and the financial products that they own. To speed up and smooth out analysis, I removed columns with information that is not essential for the current project. The columns that I kept were as follows: 
  * **Financial Wellbeing Score:** The outcome of interest
  * **Financial Products:** Whether the respondent includes one of the listed financial products
  * **Controls:** As I am interested in not just observing the correlation between product ownership and financial wellbeing, but also investigating causal effects, I included several characteristics that might be correlated with both financial wellbeing and product ownership. Controlling for these will help unearth causal relations. The characteristics I include are: income, employment status, education, financial skill and health status. 

* **Removing Junk Values:** Several columns included 'junk values' that represented a respondent's data being unavailable. After checking that only around 200 out of 6394 responses included junk values I decided that, since little data would be lost, to remove rows inclduing junk values. 

* **Assigning categories to numeric labels:** Based on CFPB's provided Codebook, I replaced the numeric labels used to code education, employment and income status with their associated categorical values. This made creating dummy variables for each category easier down the line. 

* **Re-labeled columns for easy comprehension**

## Exploratory Analysis 
To gain an initial understanding of the data, I created several visualizations. For the numerical data, I created histograms to understand their distribution. For the  and bar charts for the categorical data to understand the balance of classes. 

[INSERT IMAGES] 

In addition, I transformed the categorical variables into dummy variables to create pivot tables that for each product showed the mean and medians associated with owning and not owning the product. I added a third row that calculates the difference between the median and the mean for each product, to see which products create the greatest differences. I used the following code to create pivot tables:


    for i in df_products.columns:
    table = pd.pivot_table(df, values=['financial_wellbeing'], index=[i],
                   aggfunc=[np.mean,np.median])
    table.loc['difference'] = np.abs(table.loc[0].sub(table.loc[1], fill_value=0))
    print(table)
    print()



| Retirement Account | Mean | Median |
| --------------- | --------------- | --------------- |
| Yes | 60.1 | 50 |
| No | 50.4 | 60 |
| Difference | 9.7 | 10 |


| Investment Account | Mean | Median |
| --------------- | --------------- | --------------- |
| Yes | 63.7 | 63 |
| No | 52.4 | 52 |
| Difference | 11.2 | 11 |


Retirement accounts were associated with the greatest difference in means, while investment accounts saw the greatest difference in medians. 

## Regression Analysis 

I proceeded in two steps:
1. Run univariate regressions of each product on the financial wellbeing score
2. For the models with the best fit, add different collections of control variables and run multivariate regressions

For step 1, I added visualizations of each regression and created a table with the R-squared value of each model to easily compare fit across models. 

  
    for i in df_products.columns:
       
      X = df_products[i] 
      y = df['financial_wellbeing']
      X = sm.add_constant(X)
      est = sm.OLS(y, X).fit()

      print(est.summary())



The models with highest R-squared values were the models that included the retirement account, pension, investment account and health insurance. These products also saw some of the strongest correlations with financial wellbeing. 

[Add visualizations & table]


| Product | Coefficient | R Squared | p-value |
| --------------- | --------------- | --------------- | ---- |
| Savings Account | 8.6 | 0.046 | 0.00 |
| Life Insurance | 4.8 | 0.030 | 0.00 |
| Health Insurance | 6.9 | 0.048 | 0.00 |
| Retirement Account | 9.8 | 0.118 | 0.00 |
| Pension | 9.0 | 0.092 | 0.00 |
| Investment Account | 11.1 | 0.136 | 0.00 | 
| Education Savings Account | 5.22 | 0.008 | 0.00 |
| Student Loan | -6.1 | 0.023 | 0.00 |

For the second step, I first added each control variable individually to see how they changed the effect of the impact of the products on financial wellbeing. 

| Product | Coefficient | R Squared | p-value |
| --------------- | --------------- | --------------- | ---- |
| Income | 0.0(0008) | 0.131 | 0.00 |
| Education | 4.8 | 0.071 | 0.00 |
| Health | 4.5 | 0.244| 0.00 |
| Financial Skill | 0.6 | 0.244 | 0.00 |
| Employent | 9.0 | 0.092 | 0.00 |
| Investment Account | 11.1 | 0.136 | 0.00 | 
| Education Savings Account | 5.22 | 0.008 | 0.00 |
| Student Loan | -6.1 | 0.023 | 0.00 |

Finally, I added all control variables to study the impact of each product on financial wellbeing. 

| Product | Coefficient | Adjusted R Squared | p-value |
| --------------- | --------------- | --------------- | ---- |
| Savings Account | 3.4 | 0.427 | 0.00 |
| Life Insurance | 1.3 | 0.422| 0.00 |
| Health Insurance | 2.0 | 0.423 | 0.00 |
| Retirement Account | 4.4 | 0.438 | 0.00 |
| Pension | 3.2 | 0.49 | 0.00 |
| Investment Account | 4.3 | 0.435 | 0.00 | 
| Education Savings Account | -0.2 | 0.419 | 0.70 |
| Student Loan | -4.1 | 0.430 | 0.00 |


## Discussion and Looking Forward 

Financial products do not explain much of the variation in financial wellbeing 
Retirement and investment accounts had the greatest (positive) effect on financial wellbeing. 
Student loans, which had a negative effect, followed close behind
The impact of all products was reduced once income, employment status, education and health status was accounte for 

* Did not check for interaction effects: Impact might be different depending on characteristics
* Multi-colinearity (could check with VIF test)
  * Not a problem if it is just between the control vartiables 
  * Problem if it is between our variables of interest and control variables 

