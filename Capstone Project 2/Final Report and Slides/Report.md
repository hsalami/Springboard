
# Overview
In this report, I explain the approach I followed to analyze the factors of user adoption. I first performed some exploratory data analysis by visualizing some of the features and by computing the statistical association between the features and the target variable. Then, I built a predictive model (gradient boosting tree) and used its features importances as part of the analysis. The code and plots can be found [here](https://github.com/hsalami/Springboard/blob/master/Take%20Home%20Challenges/Relax%20Challenges/code.ipynb). 

# Data Preparation for Analysis 

*Target variable (adopted)*: I first identified the "adopted" users based on their login activity. The users who did not not log into the product the intended amount of time and the users who *never* logged were identified as "non-adopted". The percentage of "adopted" users is 13% and the percentage of "non-adopted" users is 87% (out of 12000 users).

*Processing the columns "email" and "invited_by_user_id"*: I only kept the domain part of the emails. I replaced the nan values in the column of "invited_by_user_id" with 0, as an indicator that the user did not receive any invitation from other user.

*Adding additional columns*: Using the column 'creation_time', I extracted the year, month, day and hour of the creation time and added each as one column. I also added the size (in terms of the number of users) of the organization to which the user belongs.

Note that I discarded the column 'last_session_created', because it consists of the last login time, that might have been used to identify if a user is adopted or not.

# Exploratory Data Analysis

*Data Visualization*: All of the columns in the data are categorical (except for the added column: size of the organization). For the categorical columns with low cardinality, I plotted using bar plot, the distribution of "adopted" and "non adopted" users within each category (i.e., unique value), and compared it to the overall distribution (13% for adopted and "87%" for non-adopted). **Key Observations**: the columns 'opted_in_to_mailing_list', 'enabled_for_marketing_drip' and 'day name' showed no differences, on the other hand, the columns 'month', 'year', 'source of creation' and 'email' showed more differences, especially for the following categories: 'personal_projects' and the months of 5 and 6 and the year of 2014.

*Measuring the association*: I statistically measured the association between each categorical feature and the target variable using Cramer's V coefficient. The top 3 features were the ids of the users who sent the invitation, the creation month and the ids of the organization. The last 3 features had 0 as Cramer's V coefficients, which are: 'opted_in_to_mailing_list', 'enabled_for_marketing_drip' and 'day name' (similarly to what I have observed in the bar plots). This means that these features have no association with the target variable. This is why I decided to remove them before proceeding to the next step.

# Model Building

I first split the data into 80% for training and 20% for testing. The nominal categorical features with low cardinality (creation source and email domain) were encoded using one hot encoding. For the categorical features with high cardinality (ids of the invited_by_user and the ids of the organization), I decided to use leave-one-out encoding scheme, because it replaces each unique value with one number. Since the data is imbalanced, I decided to use the oversampling method 'SMOTE' in order to create a balanced training set before feeding it to the model, in this way the minority class won't be ignored. To measure the performance of the trained class, I used the F1-score metric instead of accuracy, again because of the imbalances in the data. Three models were tried (logistic regression, random forest and gradient boosting) and one model was selected through 5-fold cross validation based on its F1-score. The final model was gradient boosting tree (F1-score: 21% and accuracy: 73%). According to th feature importances of the model, the top 4 features are the id of the invited_by_user, the id of the organization, the creation month and the creation source: personal projects.

# Insights & Recommendations

- The columns 'opted_in_to_mailing_list' and 'enabled_for_marketing_drip' have no statistical significant association with the target variable.
- The ids of the users who sent the invitations and the ids of the organization showed the strongest association with target variable. It might be worth to investigate more the characteristics of each organization and each user who sent invitation. The characteristics might be in terms of the level of activeness, or the types of activities or services provided., and the degree of influence of the users who are reaching out for other users.
- It might be also helpful to examine the month at which the account has been created and check if it is related to any special event or occasion. "Personal projects" might be a good indicator that the user will bes less likely to be an adopted user. This might be because users are not interested in other's workplace.
- The obtained model did not perform well in terms of F1-score, this means that more data need to be collected as well as more features to gain better insight into the user engagement.


```python

```
