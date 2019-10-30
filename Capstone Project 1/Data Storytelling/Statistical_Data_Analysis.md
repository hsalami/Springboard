<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Overview" data-toc-modified-id="Overview-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Overview</a></span></li><li><span><a href="#Contingency-Table-Chi-Square-Test" data-toc-modified-id="Contingency-Table-Chi-Square-Test-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Contingency Table Chi-Square Test</a></span></li><li><span><a href="#Importing-Modules-and-Defining-Functions" data-toc-modified-id="Importing-Modules-and-Defining-Functions-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Importing Modules and Defining Functions</a></span></li><li><span><a href="#Inferential-Statistics-on-the-Features-of-the-Data" data-toc-modified-id="Inferential-Statistics-on-the-Features-of-the-Data-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>Inferential Statistics on the Features of the Data</a></span><ul class="toc-item"><li><span><a href="#Posted-Speed-Limit" data-toc-modified-id="Posted-Speed-Limit-4.1"><span class="toc-item-num">4.1&nbsp;&nbsp;</span>Posted Speed Limit</a></span></li><li><span><a href="#All-other-features" data-toc-modified-id="All-other-features-4.2"><span class="toc-item-num">4.2&nbsp;&nbsp;</span>All other features</a></span></li></ul></li></ul></div>

# Overview

We have so far visualized how each type of crashes (injury and no injury) distributes across the categories of each crash feature or descriptor. In this report, we statistically explore the contribution of each feature to each crash type. More specifically, we perform the chi-square test to check if each feature has an effect or no on the type of crash. 

# Contingency Table Chi-Square Test

To test whether there is an association or not between each crash feature and the crash type, we use contingency table chi-square test. This test considers the following null and alternate hypotheses:
- Null hypothesis $H_0$: A given crash feature has no effect on the crash type;
- Alternate hypothesis $H_a$: A given crash feature has an effect on the crash type.

Given a specific crash feature, we perform the test by first building a table consisting of the numbers of each crash type observed for each category of the feature. We then compare the observed value of each subgroup with the expected value we could have obtained if the null hypothesis were true. The null hypothesis assumes that the considered crash feature and the crash type are independent. Therefore, under the null hypothesis, the expected value for each category is computed by assuming that the probability of a crash with injury given any category of the feature is the same as the overall probability of a crash with injury, and the probability of a crash with no injury given any category of the feature is the same as the overall probability of a crash with no injury. After having computed the expected ($E_i$) and observed ($O_i$) numbers of crashes for each subgroup, we compute the following statistic $\sum_i\frac{(O_i-E_i)^2}{E_i}$, which follows a chi-square distribution. We then find the probability of obtaining a value for the statistic as extreme as the observed one (p-value). The last two steps (finding the statistic and its p-value) can be performed using the chisquare() function from scipy.stats. If the p-value is small enough, we reject the null hypothesis. We next state the obtained result for each feature, but before we define the functions used in the analysis.

# Importing Modules and Defining Functions

In this section, we first import the required modules.


```python
import pandas as pd
import numpy as np
from scipy.stats import chisquare
```

We then define the following two functions: extract_features() and extract_expected(). The first function takes a dataframe and a column name (i.e., feature) and returns a data frame that consists of the count values for each crash type and category of the feature. The rows of the returned dataframe consists of the type of the crash (injury or no injury) and the columns consists of the categories of the considered feature. The first function also returns the total number of crashes with injury and the total number of crashes with no injury. The second function takes as input a dataframe similar to what is returned by the first funciton, and returns a list of the observed values of each cell of the dataframe input and a list of the corresponding expected values we could have observed for each cell if the null hypothesis were true. 


```python
def extract_feature(df, colname):
    injury = crashes[colname][crashes.CRASH_TYPE ==
                              "INJURY AND / OR TOW DUE TO CRASH"].value_counts()
    no_injury = crashes[colname][crashes.CRASH_TYPE !=
                                 "INJURY AND / OR TOW DUE TO CRASH"].value_counts()

    n_injury = sum(injury)
    n_no_injury = sum(no_injury)
    n_total = n_injury + n_no_injury

    no_injury = no_injury.reset_index()
    no_injury.columns = [colname, 'Count']
    injury = injury.reset_index()
    injury.columns = [colname, 'Count']

    feature = pd.merge(injury,
                       no_injury,
                       how='outer',
                       on=colname,
                       suffixes=('_injury', '_no_injury'))
    feature = feature.fillna(0)
    feature = feature.set_index(colname).transpose()

    return (feature, n_injury, n_no_injury)
```


```python
def extract_expected (df,n_injury, n_no_injury):
    f_observed = []
    f_expected = []
    n_total = n_injury + n_no_injury
    for col in df.columns:
        f_observed.append(df[col][0])
        f_expected.append((n_injury/n_total)*sum(df[col]))
    for col in df.columns:
        f_observed.append(df[col][1])
        f_expected.append((n_no_injury/n_total)*sum(df[col]))
        
    ddof = len(f_observed) - 1 - (len(df.columns) - 1)
    return (f_observed, f_expected, ddof)
```

We now load the data.


```python
crashes = pd.read_csv("crashes.csv")
```

# Inferential Statistics on the Features of the Data

In this section, we perform the chi-square test on each feature of the crashes. We first detail the results for the first feature: posted speed limit to clarify the procedure and then state the results of the remaining features.

## Posted Speed Limit

We first extract the counts for each possible speed limit and for each crash type.


```python
speed, n_injury, n_no_injury = extract_feature(crashes, "POSTED_SPEED_LIMIT")
```


```python
speed
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>POSTED_SPEED_LIMIT</th>
      <th>30</th>
      <th>35</th>
      <th>25</th>
      <th>15</th>
      <th>20</th>
      <th>40</th>
      <th>45</th>
      <th>55</th>
      <th>10</th>
      <th>50</th>
      <th>60</th>
      <th>65</th>
      <th>70</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Count_injury</td>
      <td>57666.0</td>
      <td>6990.0</td>
      <td>4165.0</td>
      <td>3239.0</td>
      <td>2184.0</td>
      <td>1196.0</td>
      <td>680.0</td>
      <td>141.0</td>
      <td>101.0</td>
      <td>10.0</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>Count_no_injury</td>
      <td>188938.0</td>
      <td>15456.0</td>
      <td>15119.0</td>
      <td>32422.0</td>
      <td>8844.0</td>
      <td>1882.0</td>
      <td>1333.0</td>
      <td>516.0</td>
      <td>978.0</td>
      <td>34.0</td>
      <td>7.0</td>
      <td>2.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



This table reflects the observed number of crashes for each subgroup. We want to test the following the hypotheses:
- Null Hypothesis:  The posted speed limit has no effect on the type of the crash that happened;
- Alternate Hypothesis: The posted speed limit has an effect on the type of the crash that happened.

As explained above, we first find the expected number of crashes in each cell of the above table under the null hypothesis. For instance, under the null hypothesis (independence between the speed limit and the crash type), the expected number of crashes with injury when the posted speed limit is 30 is computed as follows: 
$$\frac{\text{Total number of crashes with injury}}{\text{Total number of crashes}} \times \text{Total number of crashes at the speed limit 30}$$
We perform these calculations for each cell:


```python
f_obs, f_exp, ddof = extract_expected(speed, n_injury, n_no_injury)
```

To check the expected values and compare them with the observed ones, we construct a similar table from the epcted values.


```python
df_expected = pd.DataFrame(np.reshape(np.array(f_exp),
                                      (2, len(speed.columns))))
df_expected.columns = speed.columns
df_expected.rename(index={0: speed.index[0], 1: speed.index[1]}, inplace=True)
df_expected
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>POSTED_SPEED_LIMIT</th>
      <th>30</th>
      <th>35</th>
      <th>25</th>
      <th>15</th>
      <th>20</th>
      <th>40</th>
      <th>45</th>
      <th>55</th>
      <th>10</th>
      <th>50</th>
      <th>60</th>
      <th>65</th>
      <th>70</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Count_injury</td>
      <td>55090.349674</td>
      <td>5014.346843</td>
      <td>4307.968659</td>
      <td>7966.525116</td>
      <td>2463.611199</td>
      <td>687.612919</td>
      <td>449.696168</td>
      <td>146.771179</td>
      <td>241.044295</td>
      <td>9.829424</td>
      <td>2.904148</td>
      <td>1.11698</td>
      <td>0.223396</td>
    </tr>
    <tr>
      <td>Count_no_injury</td>
      <td>191513.650326</td>
      <td>17431.653157</td>
      <td>14976.031341</td>
      <td>27694.474884</td>
      <td>8564.388801</td>
      <td>2390.387081</td>
      <td>1563.303832</td>
      <td>510.228821</td>
      <td>837.955705</td>
      <td>34.170576</td>
      <td>10.095852</td>
      <td>3.88302</td>
      <td>0.776604</td>
    </tr>
  </tbody>
</table>
</div>



We now compute the statistic and its p-value using the chisqure function.


```python
chisquare(f_obs, f_exp, ddof)
```




    Power_divergenceResult(statistic=5569.535596244047, pvalue=0.0)



We see that the p-value is very small close to zero and in this case we fail to reject the null hypothesis, i.e, we fail to reject that the posted speed limit has no effect on the type of crash that occurred.

## All other features

We now perform the same test for all remaining features:

- Traffic Control Device: 


```python
device, n_injury, n_no_injury = extract_feature(crashes, "TRAFFIC_CONTROL_DEVICE")
f_obs, f_exp, ddof = extract_expected(device, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=5063.420455233677, pvalue=0.0)
    

- Weather Condition:


```python
weather, n_injury, n_no_injury = extract_feature(crashes, "WEATHER_CONDITION")
f_obs, f_exp, ddof = extract_expected(weather, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=1119.7261719166079, pvalue=2.103655474949214e-236)
    

- Lighting Condition:


```python
light, n_injury, n_no_injury = extract_feature(crashes, "LIGHTING_CONDITION")
f_obs, f_exp, ddof = extract_expected(light, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=6025.154213534629, pvalue=0.0)
    

- Road Alignment:


```python
align, n_injury, n_no_injury = extract_feature(crashes, "ALIGNMENT")
f_obs, f_exp, ddof = extract_expected(align, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=1094.0577828746289, pvalue=2.5878585302373544e-234)
    

- Road Condition:


```python
road, n_injury, n_no_injury = extract_feature(crashes, "ROADWAY_SURFACE_COND")
f_obs, f_exp, ddof = extract_expected(road, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=1410.2406494150573, pvalue=4.159226906544088e-304)
    

- Collision Type:


```python
collision, n_injury, n_no_injury = extract_feature(crashes, "FIRST_CRASH_TYPE")
f_obs, f_exp, ddof = extract_expected(collision, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=43081.16185838227, pvalue=0.0)
    

- Trafficway Type:


```python
trafficway, n_injury, n_no_injury = extract_feature(crashes, "TRAFFICWAY_TYPE")
f_obs, f_exp, ddof = extract_expected(trafficway, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=9538.105562426485, pvalue=0.0)
    

- Contributory Cause:


```python
cause, n_injury, n_no_injury = extract_feature(crashes, "PRIM_CONTRIBUTORY_CAUSE")
f_obs, f_exp, ddof = extract_expected(cause, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=31993.29780516069, pvalue=0.0)
    

- Crash Hour:


```python
hour, n_injury, n_no_injury = extract_feature(crashes, "CRASH_HOUR")
f_obs, f_exp, ddof = extract_expected(hour, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=6786.549164952034, pvalue=0.0)
    

- Crash Day:


```python
day, n_injury, n_no_injury = extract_feature(crashes, "CRASH_DAY_OF_WEEK")
f_obs, f_exp, ddof = extract_expected(day, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=538.5418733470326, pvalue=4.1657452622742266e-113)
    

- Crash Month:


```python
month, n_injury, n_no_injury = extract_feature(crashes, "CRASH_MONTH")
f_obs, f_exp, ddof = extract_expected(month, n_injury, n_no_injury)
print(chisquare(f_obs, f_exp, ddof))
```

    Power_divergenceResult(statistic=182.009548238753, pvalue=3.9430286408476375e-33)
    

We see that for each feature, the obtained p-value is very low close to zero. Therefore for all of these feature, we reject the null hypothesis, i.e., we reject the hypothesis that each crash feature has no effect on crash type.
