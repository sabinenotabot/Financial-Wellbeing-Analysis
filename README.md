# Financial Wellbeing Analysis (Python)

## Project Overview 

### Objective 

The objective of this project is to understand the association between an individual's ownership of specific types of financial products (e.g. savings account, health insurance) and their 'financial wellbeing', a measure of the extent to which one's financial situation provides security and freedom. In addition to exploring whether their is a correlation between ownership of certain types of financial produts and financial wellbeing, I will also investigate whether their might be a causal effect on financial wellbeing (e.g. owning a savings account 'causes' an individual's financial wellbeing to improve) by controlling for confounding variables.  

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
* Financial products do not explain much of the variation in financial wellbeing among respondents. By themselves, most products explain less than 10% of the variation in financial wellbeing scores. However, the effect of products on financial wellbeing scores is significant. 
* Retirement and investment accounts had the greatest (positive) effect on financial wellbeing 
* Student loans, which has a negative effect of financial wellbeing, followed close behind
* The impact of all products reduces once income, employment status, education and health are accounted for. Yet, the effect of the financial products on the financial wellbeing score remains significant.


## Code and Resources Used 
**Python Version:** 3.7

**Packages:** pandas, numpy, statsmodels, matplotlib, seaborn

## The Dataset 
I use data from the CFPB's 2016 financial wellbeing survey, downloaded from [the CFPB website](https://www.consumerfinance.gov/data-research/financial-well-being-survey-data/). The survey dataset includes respondents’ scores on the CFPB Financial Wellbeing Scale, the financial products that they use as well as other measures of individual and household characteristics that research suggests may influence adults’ financial well-being (e.g. income and employment, education). 

The National Financial Well-Being Survey was conducted in English and Spanish via web mode between October 27, 2016 and December 5, 2016. Overall, 6,394 surveys were completed: 5,395 from the general population sample and 999 from an oversample of adults aged 62 and older.

## Data Cleaning 
After retrieving the CFPB survey data, I clean the data to improve the accuracy of my findings. Changes made to the raw dataset include: 

* **Removing irrelevant columns:** The survey collects 217 characteristics of each respondent that go far beyond their financial wellbeing scores and the financial products that they own. To speed up and smooth out analysis, I remove columns with information that is not essential to the current question. The columns that are kept are as follows: 
  * **Financial Wellbeing Score:** The outcome of interest
  * **Financial Products:** Whether the respondent owns one of the listed financial product types
  * **Controls:** As I am interested in not just observing the correlation between product ownership and financial wellbeing, but also investigating causal effects, I included several characteristics that might be correlated with both financial wellbeing and product ownership. Controlling for these will help unearth causal relations. The characteristics I include are: income, employment status, education, financial skill and health status. 

* **Removing Junk Values:** Several columns included 'junk values' that represented a respondent's data being unavailable. After checking that only around 200 out of 6394 responses included junk values I decide to remove rows inclduing junk values, since little data (relative to the size of the compelete dataset) is lost as a result.

* **Assigning categories to numeric labels:** Based on CFPB's provided Codebook, I replace the numeric labels used to code education, employment and income status with their associated categorical values. This makes creating dummy variables for each category easier down the line. 

* **Re-labeled columns for easy comprehension**

## Exploratory Analysis 
To gain an initial understanding of the data, I create several visualizations. For the numerical data, I create histograms to understand their distribution. For the categorical data, I create bar charts for the categorical data to understand the balance of classes. 

Here are two examples of the visualizations included:

 ![Histogram of Financial Skill Scores](https://github.com/sabinenotabot/Financial-Wellbeing-Analysis/blob/master/financial_skill%20plot.png)
 
 ![Barchart of educational status](https://github.com/sabinenotabot/Financial-Wellbeing-Analysis/blob/master/education%20plot.png)

In addition, I transform the categorical variables into dummy variables. Using these, I create pivot tables that, for each product type, show the mean and medians associated with owning and not owning the product type. I add a third row that calculates the difference between the median and the mean for each product type, to see which products are associated with the greatest differences in financial wellbeing. 

A short code like this can be used to create informative pivot tables:


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


Retirement accounts are associated with the greatest difference in means, while investment accounts see the greatest difference in medians. 

## Regression Analysis 
### Univariate Regression

I begin my regression analysis by running univariate regressions of the financial wellbeing score on each product type. The outcome of thsi regression indicates whether owning certain types of financial products is associated with differences in financial wellbeing. 

Using statsmodels, one can quickly run a univariate regression by using the following type of code: 
    
    for i in df_products.columns:
       
      X = df_products[i] 
      y = df['financial_wellbeing']
      X = sm.add_constant(X)
      est = sm.OLS(y, X).fit()

      print(est.summary())

I add visualizations of each regression and create a table with the R-squared value of each model to easily compare fit across models. 

An example of a boxplot used to visualize one of the univariate regression models:

![Boxplot of regression of pension on financial wellbeing](https://github.com/sabinenotabot/Financial-Wellbeing-Analysis/blob/master/pension%20boxplot.png)

The results of the univariate regression are as follows: 

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

There are several things to note about these results: 
* The R Squared values for most products are low, meaning that the associated univariate regression model explains little of the variation in financial wellbeing. The model that includes investment accounts explains the most variation, around 13.6%.
* While the explanatory power of the models is low, the p-value indicates that for all financial products the effect on financial wellbeing is statistically significant 
* The products that seem to have the greatest impact on financial wellbeing are: investment accounts, retirement accounts and pensions
* Student loans are the only product type that has a negative effect on financial wellbeing. This is unsurprising as it is the product type that represents a liability.
* The R squared of the model including education savings accounts is very low. A contributing factor might be that the data is unbalanced, with very few respondents owning education savings accounts.

### Multivariate Regression

Before adding the control variables, I examine the effect that the control variables individually have on financial wellbeing. Running univariate regressions of the financial wellbeing score on each control variable. The results of these regressions include:


| Product | Coefficient | R Squared | p-value |
| --------------- | --------------- | --------------- | ---- |
| Income | 0.0(0008) | 0.131 | 0.00 |
| Health | 4.5 | 0.244| 0.00 |
| Financial Skill | 0.6 | 0.244 | 0.00 |

| Product | R Squared |
| --------------- | --------------- |
| Education | 0.071 |
| Employment |0.092 |

For each dummy variable representing a type of employment or education level, p < 0.5. 

These results indicate that the control variables individually explain some, but not a lot, of variation in financial wellbeing scores. Health and financial skill perform the best in this regard. In addition, for all control variables, the effects on financial wellbeing are significant. 

While significance or strong explanatory power t required to motivate inclduing 

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

The results of the regression show that financial products do not explain much of the variation in financial wellbeing. However, the effect on financial wellbeing was found to be significant, even after controlling for confounding characteristics. Retirement and investment accounts had the greatest (positive) effect on financial wellbeing. Student loans, which had a negative effect, followed close behind The impact of all products was reduced once income, employment status, education and health status was accounte for 

There are several ways in which the modelling might be improved:
* **Adding Interaction Terms**: I did not check for interaction effects between products and control variables. If it is the case that the impact of product ownership on financial wellbeing varies for different levels of the control variables, introducing interaction terms would improve the moel. 
* **Accounting for Multi-colinearity**: I did not check for multi-colinearity (with, for example, a VIF test). There is chance that multi-colinearity is distorting are results as it might be the case that multi-colinearity occurs between some of the product variables and the control variables (e.g. income might be correlated with whether one has an investment account). While there might also be multi-colinearity between the control variables, this is less of a concern, as it does not affect the values we are interested in. 


