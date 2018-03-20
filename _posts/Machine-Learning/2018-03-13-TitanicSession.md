---
layout: single
title: "Titanic, Machine Learning from Disaster"
date: 2018-03-13
categories: machine-learning
author_profile: false
mathjax: true
toc: true
---

- **1 Introduction**
- **2 Exploratory Analysis**
    - Data visualization
    - Oulier detection
    - Working with missing values
- **3 Feature Engineering**
    - Hidden information
    - Combining variables
- **4 Modelling**
    - Hyperparameter tuning

## Introduction

The purpose of this Notebook is to give an end-to-end workflow example when aiming to solve a Machine Learning problem. The data used is taken from [Kaggle](https://www.kaggle.com/c/titanic/data). 

It is vital to always understand the different variables that are given to us, and even more when working with small dimentions like in this case, where we will try to extract *hidden* information and take advantage of this knowledge to extract more powerful features than the raw ones.  

### Data Dictionary

| Variable  | Definition  | 
|---|---|
| survival  | Survival {0,1}  |   
|pclass   | Ticket class {1,2,3}  |  
| sex  | Sex  | 
|Age|Age in years|
|sibsp|# of siblings / spouses aboard the Titanic	|
|parch	|# of parents / children aboard the Titanic	|
|ticket|	Ticket number	|
|fare|	Passenger fare	|
|cabin|	Cabin number	|
|embarked|	Port of Embarkation|

**Note**: Age is fractional if less than 1. If the age is estimated, is it in the form of xx.5.

(Embarkation ports: C = Cherbourg, Q = Queenstown, S = Southampton)

### Exploratory Analysis

Load the libraries and the data:


```python
# Modules for Data Analysis
import numpy as np
import pandas as pd
import regex as re
import xgboost as xgb
import lightgbm as lgb

# And for visualization
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

import warnings
warnings.filterwarnings("ignore")

from xgboost import plot_importance
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
from sklearn import model_selection
from sklearn.metrics import accuracy_score
from sklearn.model_selection import KFold
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import GridSearchCV

from collections import Counter

plt.style.use('fivethirtyeight')
sns.set_palette("GnBu_d")

train = pd.read_csv("../input/train.csv", sep = ",")
test = pd.read_csv("../input/test.csv", sep = ",")
test_id = test["PassengerId"]
```

The **pandas** module allows us to work with *dataframes*, which are just data organized in tables and also gives us powerful methods to explore inside the dataset. Start by checking the data dimentions and take a glimpse of how does it look like:


```python
print('The dataset contains %s rows and %s columns.' 
      % (train.shape[0],train.shape[1]))
train.head(5)
```

    The dataset contains 891 rows and 12 columns.
    




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
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>7.2500</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>71.2833</td>
      <td>C85</td>
      <td>C</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>Heikkinen, Miss. Laina</td>
      <td>female</td>
      <td>26.0</td>
      <td>0</td>
      <td>0</td>
      <td>STON/O2. 3101282</td>
      <td>7.9250</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>Futrelle, Mrs. Jacques Heath (Lily May Peel)</td>
      <td>female</td>
      <td>35.0</td>
      <td>1</td>
      <td>0</td>
      <td>113803</td>
      <td>53.1000</td>
      <td>C123</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0</td>
      <td>3</td>
      <td>Allen, Mr. William Henry</td>
      <td>male</td>
      <td>35.0</td>
      <td>0</td>
      <td>0</td>
      <td>373450</td>
      <td>8.0500</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>



The objective of this problem is creating a classification model that can discern whether an individual would have survived the titanic tragedy or not by just considering sociodemographic data. Thus, it is important to check how the given variables have an impact in the individuals survivavility. 

However, one must have in mind one aspect: Machine Learning lies inside artificial intelligence, meaning that the model will learn from the data we are feeding it, but a good model is not defined by how perfectly it predicts one set of data but by its accuracy when the test data can flow and vary. In order to do so, the prime idea is to let the model learn from the relations between the variables and not, in this case, the final destination a passenger had. This is why one must avoid identificative data and concentrate on descriptive features and then let the model predict that Mr. Owen survived because he was a 22 years old male who embarked at Southampton rather than because he is the passenged with Id = 1.


```python
print(train.dtypes)
```

    PassengerId      int64
    Survived         int64
    Pclass           int64
    Name            object
    Sex             object
    Age            float64
    SibSp            int64
    Parch            int64
    Ticket          object
    Fare           float64
    Cabin           object
    Embarked        object
    dtype: object
    

## Outlier detection

A robust choice when treating outliers is the IQR (interquartile range) method, developed by John Tukey. We label a row as an outlier if its column value is outside the IQR + a step.


```python
def outliers_iqr(vcol):
    quartile_1, quartile_3 = np.percentile(train[vcol], [25, 75])
    iqr = quartile_3 - quartile_1
    step = iqr * 1.5
    lower_bound = quartile_1 - step
    upper_bound = quartile_3 + step
    outliers.extend(train[(train[vcol] < lower_bound) | 
                   (train[vcol] > upper_bound)].index)
```

We will apply this function to the variables that can present numerical outliers: Age, Fare, Parch and SibSp. However recall how Age had missing values that we haven't yet treated. We will work with it separetely.


```python
num_f = ['Fare', 'Parch', 'SibSp']
outliers = []
for col in num_f:
    outliers_iqr(col)
```


```python
outliers = list(set([x for x in outliers if outliers.count(x)>2]))
```

We want to save for further examination those individuals that present outliers in more 2 variables. However, we do not want to have them repeated, that is why we invoke the list comprehension as a set, to have the duplicated eliminated and then we convert it back to a list.


```python
train.loc[outliers]
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
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>324</th>
      <td>325</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Mr. George John Jr</td>
      <td>male</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>201</th>
      <td>202</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Mr. Frederick</td>
      <td>male</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>846</th>
      <td>847</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Mr. Douglas Bullen</td>
      <td>male</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>792</th>
      <td>793</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Miss. Stella Anna</td>
      <td>female</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>180</th>
      <td>181</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Miss. Constance Gladys</td>
      <td>female</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>341</th>
      <td>342</td>
      <td>1</td>
      <td>1</td>
      <td>Fortune, Miss. Alice Elizabeth</td>
      <td>female</td>
      <td>24.0</td>
      <td>3</td>
      <td>2</td>
      <td>19950</td>
      <td>263.00</td>
      <td>C23 C25 C27</td>
      <td>S</td>
    </tr>
    <tr>
      <th>863</th>
      <td>864</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Miss. Dorothy Edith "Dolly"</td>
      <td>female</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>88</th>
      <td>89</td>
      <td>1</td>
      <td>1</td>
      <td>Fortune, Miss. Mabel Helen</td>
      <td>female</td>
      <td>23.0</td>
      <td>3</td>
      <td>2</td>
      <td>19950</td>
      <td>263.00</td>
      <td>C23 C25 C27</td>
      <td>S</td>
    </tr>
    <tr>
      <th>27</th>
      <td>28</td>
      <td>0</td>
      <td>1</td>
      <td>Fortune, Mr. Charles Alexander</td>
      <td>male</td>
      <td>19.0</td>
      <td>3</td>
      <td>2</td>
      <td>19950</td>
      <td>263.00</td>
      <td>C23 C25 C27</td>
      <td>S</td>
    </tr>
    <tr>
      <th>159</th>
      <td>160</td>
      <td>0</td>
      <td>3</td>
      <td>Sage, Master. Thomas Henry</td>
      <td>male</td>
      <td>NaN</td>
      <td>8</td>
      <td>2</td>
      <td>CA. 2343</td>
      <td>69.55</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>



We can see how these passengers have either an enormous family or they paid a exhorbitant amount for their tickets. Also see how almost everyone has the age as a null, and the rest show usual values. Thus, there is no need to further examine outliers for the age.

Now we need to do something about these individuals. **Please do not never ever throw data away** (if it is not horrendous and there's nothing that can be done with it). Instead, let's input the values ourselves. We will change the value of SibSp to the mode when they have 8 and the Fare to the mean for the cases where Fare = 263.00.


```python
train.loc[(train.index.isin(outliers)) & 
          (train['SibSp'] == 8),'SibSp'] = int(train['SibSp'].mode())
train.loc[(train.index.isin(outliers)) & 
          (train['Fare'] == 263.00), 'Fare'] = train['Fare'].mean()
```

Let's start with the visualizations:


### Categorical Variables

### Sex


```python
# barplot of survivavility vs. sex
s = sns.barplot(x="Sex", y="Survived", data=train)

print("Percentage of women that survived:", 
      train["Survived"][train["Sex"] == 'female']
      .value_counts(normalize = True)[1]*100)

print("Percentage of men that survived:", 
      train["Survived"][train["Sex"] == 'male']
      .value_counts(normalize = True)[1]*100)
```

    Percentage of women that survived: 74.2038216561
    Percentage of men that survived: 18.8908145581
    


![png](/assets/images/machinelearning/titanicsession/output_24_1.png)


<div class="alert alert-info"> The method value_counts is an interesting one as it gives us the population of the different values in a column. In this case, we are accessing the column *Survived* of the train dataset for the female individuals. </div>


```python
 train["Survived"][train["Sex"] == 'female'].value_counts(normalize = True)
```




    1    0.742038
    0    0.257962
    Name: Survived, dtype: float64



<div class="alert alert-info"> This information could have also been extracted in a SQL like way. For SQL lovers, check this [link](https://pandas.pydata.org/pandas-docs/stable/comparison_with_sql.html). </div>


```python
train.groupby(['Sex','Survived'])['Survived'].count()
```




    Sex     Survived
    female  0            81
            1           233
    male    0           468
            1           109
    Name: Survived, dtype: int64



### Pclass

As we can see there is a huge bias in terms of survivavility when talking about sex, meaning that it will have a vital paper in the model. There is an obvious *Women and children first*.


```python
s = sns.factorplot(x="Pclass", y="Survived", data=train, kind='bar', size=6)
```


![png](/assets/images/machinelearning/titanicsession/output_31_0.png)



```python
pd.crosstab(train['Pclass'], train['Survived'], margins=True)
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
      <th>Survived</th>
      <th>0</th>
      <th>1</th>
      <th>All</th>
    </tr>
    <tr>
      <th>Pclass</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>80</td>
      <td>136</td>
      <td>216</td>
    </tr>
    <tr>
      <th>2</th>
      <td>97</td>
      <td>87</td>
      <td>184</td>
    </tr>
    <tr>
      <th>3</th>
      <td>372</td>
      <td>119</td>
      <td>491</td>
    </tr>
    <tr>
      <th>All</th>
      <td>549</td>
      <td>342</td>
      <td>891</td>
    </tr>
  </tbody>
</table>
</div>



Some things had not changed: money makes things easier. Moreover, it is also important to check not only the variable impact on the target value, but also how this variable is distributed along the data. If only one passenger would had Pclass = 1 and he/she survived, the first class survival probability would have been a mind blowing 100%, but it would not be representative. Thanks to the crosstab method we do not just see that there are enough individuals present in each class, but also that the data is enough well sampled, meaning that we have at our disposal enough variability in the survival class to actually learn from it. If data would only show a small percentage of survivals, training a model that could actually predict survivavility would get pretty hard as there would not be enough information about the individuals who got to live.

Also, with only a small representation of the survived class (for example only a 5%), a constant model that always give '0' as an output would show an accuracy on 95% on the data! So please, always give a second and a third look to the data before making any conclusions.

### Embarked


```python
s = sns.factorplot(x="Embarked", y="Survived", data=train,
                   size=6, kind="bar")
```


![png](/assets/images/machinelearning/titanicsession/output_35_0.png)



```python
pd.crosstab(train['Embarked'], train['Survived'], margins=True)
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
      <th>Survived</th>
      <th>0</th>
      <th>1</th>
      <th>All</th>
    </tr>
    <tr>
      <th>Embarked</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>C</th>
      <td>75</td>
      <td>93</td>
      <td>168</td>
    </tr>
    <tr>
      <th>Q</th>
      <td>47</td>
      <td>30</td>
      <td>77</td>
    </tr>
    <tr>
      <th>S</th>
      <td>427</td>
      <td>217</td>
      <td>644</td>
    </tr>
    <tr>
      <th>All</th>
      <td>549</td>
      <td>340</td>
      <td>889</td>
    </tr>
  </tbody>
</table>
</div>



Do the numbers ring any bells? We have a total of 168 passengers coming from Cherbourg, which shows the higher survivality rate, followed by the 77 passengers who embarked on Queenstown, with less survivavility and higher variance. If you now scroll up a bit, you can see that 216 passengers had a first class ticket, who also showed the top survivavility chances compared to the other ticket types. Could this two variables be related, meaning that the 216 first class passengers where more present in the 245 passengers embarking in Cherbourg and Queenstown?

What we were performing so far was a **univariate analysis**, where we only took into consideration one variable at a time. While this may be a good enough starting point we are losing information on the relation between features, so another type of analysis gains importance: **multivariate analysis**:


```python
s = sns.factorplot("Pclass", col="Embarked", data=train,
                   size=6, kind="count")
```


![png](/assets/images/machinelearning/titanicsession/output_38_0.png)


Now it is farily easy to see that in S almost every 3rd class passenger embarked, whereas in C passengers are mostly from first class. However, this anulates our hypothesis of saying that first class passengers mainly emarked in C and Q.

We saw that women had a way higher survival rate. Just to be sure, let's check that the effect of the ticket class/embarkation port is no masked in the sex:


```python
s = sns.factorplot("Sex", col="Pclass", data=train,
                   size=6, kind="count")
```


![png](/assets/images/machinelearning/titanicsession/output_40_0.png)



```python
s = sns.factorplot("Sex", col="Embarked", data=train,
                   size=6, kind="count")
```


![png](/assets/images/machinelearning/titanicsession/output_41_0.png)



```python
print('In Cabin we have %s different values.' % len(train['Cabin'].unique()))
print('In Ticket we have %s different values.' %len(train['Ticket'].unique()))
```

    In Cabin we have 148 different values.
    In Ticket we have 681 different values.
    

Both Cabin and Ticket have too much different classes, so we will not treat them as a *"normal"* categorical variable.

### Numerical Variables

A good thing about numerical variables is that we can plot a heatmap showing the correlation of all the variables against each other instead having to plot the probability one by one.


```python
plt.figure(figsize=(6,6))
s = sns.heatmap(train[["Survived","SibSp","Parch","Age","Fare"]].corr(), annot=True, fmt = ".2f", cmap="YlGnBu")
```


![png](/assets/images/machinelearning/titanicsession/output_46_0.png)


Looking at the values of the first row/column (as we are mainly interested in the correlation with Survival), one can see that only Fare shows somehow a bit of correlation. Note that this does not mean that the rest of variables are no explanatory, but maybe we need to play a bit more with them first.

### Age

If we tried to run:
```python
sns.distplot(train['Age'])
```

an error would appear. This is the first encounter we would have with the infamous **missing values**. However, we will do a workaround by now and attack all missing values at the same time.


```python
# Eliminate the nulls 
age = train['Age'][train['Age'].notnull()]
s = sns.distplot(age)
```


![png](/assets/images/machinelearning/titanicsession/output_50_0.png)


We can see that there is an odd peak of individuals with little age. Recall the note that age is fractional if less than 1. Let's check those individuals:


```python
train.loc[train['Age']<5,'PassengerId'].count()
```




    40



If we take a look at the babies abroad, we can see that all of them survived.


```python
train.loc[train['Age']<1]
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
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>78</th>
      <td>79</td>
      <td>1</td>
      <td>2</td>
      <td>Caldwell, Master. Alden Gates</td>
      <td>male</td>
      <td>0.83</td>
      <td>0</td>
      <td>2</td>
      <td>248738</td>
      <td>29.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>305</th>
      <td>306</td>
      <td>1</td>
      <td>1</td>
      <td>Allison, Master. Hudson Trevor</td>
      <td>male</td>
      <td>0.92</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
    </tr>
    <tr>
      <th>469</th>
      <td>470</td>
      <td>1</td>
      <td>3</td>
      <td>Baclini, Miss. Helene Barbara</td>
      <td>female</td>
      <td>0.75</td>
      <td>2</td>
      <td>1</td>
      <td>2666</td>
      <td>19.2583</td>
      <td>NaN</td>
      <td>C</td>
    </tr>
    <tr>
      <th>644</th>
      <td>645</td>
      <td>1</td>
      <td>3</td>
      <td>Baclini, Miss. Eugenie</td>
      <td>female</td>
      <td>0.75</td>
      <td>2</td>
      <td>1</td>
      <td>2666</td>
      <td>19.2583</td>
      <td>NaN</td>
      <td>C</td>
    </tr>
    <tr>
      <th>755</th>
      <td>756</td>
      <td>1</td>
      <td>2</td>
      <td>Hamalainen, Master. Viljo</td>
      <td>male</td>
      <td>0.67</td>
      <td>1</td>
      <td>1</td>
      <td>250649</td>
      <td>14.5000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>803</th>
      <td>804</td>
      <td>1</td>
      <td>3</td>
      <td>Thomas, Master. Assad Alexander</td>
      <td>male</td>
      <td>0.42</td>
      <td>0</td>
      <td>1</td>
      <td>2625</td>
      <td>8.5167</td>
      <td>NaN</td>
      <td>C</td>
    </tr>
    <tr>
      <th>831</th>
      <td>832</td>
      <td>1</td>
      <td>2</td>
      <td>Richards, Master. George Sibley</td>
      <td>male</td>
      <td>0.83</td>
      <td>1</td>
      <td>1</td>
      <td>29106</td>
      <td>18.7500</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>



Check the age distributions of passengers that survived and those who did not:


```python
s = sns.kdeplot(train["Age"][(train["Survived"] == 0) & (train["Age"].notnull())], shade = True)
s = sns.kdeplot(train["Age"][(train["Survived"] == 1) & (train["Age"].notnull())], ax=s, color='red', shade= True)
s.set_xlabel("Age")
s.set_ylabel("Frequency")
s = s.legend(["Not Survived","Survived"])
```


![png](/assets/images/machinelearning/titanicsession/output_56_0.png)


We can clearly see how younger people got to survive and passengers with age between 60 - 80 have less chance to leave the boat.

### SibSp


```python
s = sns.factorplot(x="SibSp", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_59_0.png)


Looks like both passengers without spouse or siblings and those who had a lot had less change to survive.

### Parch


```python
s = sns.factorplot(x="Parch", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_62_0.png)


Again, small families have higher chance to survive, alhough there is a high variance for individuals with 3 parents and children.

### Fare


```python
s = sns.distplot(train['Fare'])
```


![png](/assets/images/machinelearning/titanicsession/output_65_0.png)


The distribution shows a really high skewness (huge tail), so a logarithmic transformation helps in this case to normalize the values. We will make use of anonymous functions here, as we cannot convert those rows with Fare = 0, and from the plot we can see that there might be quite a few.


```python
log_Fare = train["Fare"].map(lambda x: np.log(x) if x > 0 else 0)
```


```python
sum(train.Fare == 0)
```




    15




```python
s = sns.distplot(log_Fare)
```


![png](/assets/images/machinelearning/titanicsession/output_69_0.png)


The new distribution looks more beautiful. Remember to apply all the changes done to the train set to the test! However, null values appear again:


```python
sum(test['Fare'].isnull())
```




    1



As there is only one missing value, we can just input the mean of the column without any impact on the set.


```python
test = test.fillna({'Fare' : test['Fare'].mean()}) 
#test["Fare"] = test["Fare"].map(lambda x: np.log(x) if x > 0 else 0)
```

<div class="alert alert-warning">Although when applying the logarithm we end up with a better distribution, we will apply another strategy in **feature engineering**. </div>

### Working with Missing Values

We have already seen that data will not always be informed, and the larger the dataset the higher number of values that we will have to study how to input. Luckily there is an interesting module **missingno** that helps us retrieve the information that we need:


```python
import missingno as msno

msno.bar(train)
```


![png](/assets/images/machinelearning/titanicsession/output_77_0.png)


This plot helps us visualize the percentage of data that is informed in each column. The main feature that will require our attention are Age and Cabin. Although it is a bit hard to see with this plot, Embarked has also 2 missing values. Another interesting plot provided by the module shows exactly the row where data is missing.


```python
msno.matrix(train)
```


![png](/assets/images/machinelearning/titanicsession/output_79_0.png)


<div class="alert alert-info"> To obtain the raw numbers of missing values we can use the method isnull() from pandas: pd.isnull(train).sum() </div>

<div class="alert alert-info"> The missingno correlation heatmap measures nullity correlation: how strongly the presence or absence of one variable affects the presence of another: msno.heatmap(train) </div> (Not used here because of the lack of real variability in missing values).

Here we are presented with 3 different needed approaches to treat the null values:

- Age: Almost 80% of the data is present so we will try to input values the best we can.
- Embarked & Fare: When almost any value is missing, direct inputation solves the problem without distorting the rest of the data.
- Cabin: With roughly a 23% of data present, inputation is out of the scope.

We will start with the *easy* ones:

For the variable **Embarked**, as almost everyone came from Southampton we can directly put that.


```python
train = train.fillna({'Embarked' : 'S'})
```

With a scenario like **Cabin**, the usual approach is to create a new dummy variable denoting if the cabin was informed or not for a given individual. In this case it makes sense as only a percentage of the passengers might have been wealthy enough to travel with a given private cabin. Note that any change in variables applied to the training set must also be applied to the test dataset.


```python
train['CabinInf'] = train.Cabin.notnull().astype(int) 
test['CabinInf'] = test.Cabin.notnull().astype(int)

# we won't need the Cabin variable anymore:
train.drop(['Cabin'], axis=1, inplace=True)
test.drop(['Cabin'], axis=1, inplace=True)
```

It is a good practice to check if the new variables that we are creating are candidates of helping improve the model:


```python
s = sns.factorplot(x="CabinInf", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_88_0.png)


A naive technique that one could try to use to input **Age** is by applying the mean.


```python
# Recall how the distribution looks like
s = sns.distplot(train.loc[train['Age'].notnull(),'Age'])
```


![png](/assets/images/machinelearning/titanicsession/output_90_0.png)



```python
age = train['Age']
age = list(map(lambda x: age.mean() if np.isnan(x) else x, age))
s = sns.distplot(age)
```


![png](/assets/images/machinelearning/titanicsession/output_91_0.png)


Ok, there are quite a few things happening here. First we used the **map** python functionality, that lets us apply a function to each member of an input list. In this case we are interested in putting the mean of the variable age to the variables that where null. We could have also declared separatedly a function, however we made use of anonimous functions to achieve the same with just one line of code. 

Also, the important part here is that with this lazy approach the age distribution gets heavily distorted, so a deeper thought will be needed for this task.

## Feature Engineering

As we said maybe raw variables do not contain the most information or there are others from which we can extract different features. 

### Hidden Information: Title

We want our model to learn from the characteristics of the individuals, not the individuals themselves. This is why we will drop variables like PassengerId, Name and Ticket, as they are differentiative aspects of the individuals. However:


```python
train['Name'].head()
```




    0                              Braund, Mr. Owen Harris
    1    Cumings, Mrs. John Bradley (Florence Briggs Th...
    2                               Heikkinen, Miss. Laina
    3         Futrelle, Mrs. Jacques Heath (Lily May Peel)
    4                             Allen, Mr. William Henry
    Name: Name, dtype: object




See how each passenger has a different honorific. We can extract this information to learn about the social level of the individual.


```python
for dataset in [train, test]:
    dataset['Title'] = dataset.Name.str.extract('([A-Za-z]+)\.', expand=False)

pd.crosstab(train['Title'], train['Sex'])
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
      <th>Sex</th>
      <th>female</th>
      <th>male</th>
    </tr>
    <tr>
      <th>Title</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Capt</th>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Col</th>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Countess</th>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Don</th>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Dr</th>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>Jonkheer</th>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Lady</th>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Major</th>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Master</th>
      <td>0</td>
      <td>40</td>
    </tr>
    <tr>
      <th>Miss</th>
      <td>182</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Mlle</th>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Mme</th>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Mr</th>
      <td>0</td>
      <td>517</td>
    </tr>
    <tr>
      <th>Mrs</th>
      <td>125</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Ms</th>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Rev</th>
      <td>0</td>
      <td>6</td>
    </tr>
    <tr>
      <th>Sir</th>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



As we do not end up with too many categories another step will be perfomed putting together similar honorifics. We will classify in Master (young men, recall the sample of babies abroad we saw), Mr, Miss, Mrs and the Elite, combining all the honorifics of the novelty and rich.


```python
train['Title'].replace(['Mlle','Mme','Ms','Dr','Major','Lady','Countess','Jonkheer','Col','Rev','Capt','Sir','Don'],
     ['Miss','Miss','Miss','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite'],inplace=True)
test['Title'].replace(['Mlle','Mme','Ms','Dr','Major','Lady','Countess','Jonkheer','Col','Rev','Capt','Sir','Don','Dona'],
    ['Miss','Miss','Miss','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite','Elite'],inplace=True)
```

Now we can use this new variable to input the age with lesser error, as we are working in a closer range of values. For each Title we will compute the mean and standard deviation of the age of the individuals, and instead of putting the same age to rows with missing values, a random normal value will be used to mantain the distribution.


```python
titles = ['Mr','Miss','Mrs','Master','Elite']
for dataset in [train, test]:
    for title in titles:
        mean = dataset.loc[dataset['Title'] == title,'Age'].mean()
        std = dataset.loc[dataset['Title'] == title,'Age'].std()
        dataset.loc[dataset['Title'] == title,'Age'] = list(map(lambda x: np.random.normal(loc=mean, scale=std) 
                                  if np.isnan(x) else x, dataset.loc[dataset['Title'] == title,'Age']))
```

Look how using information from another variables and inputing a random value within a calculated range preserves the distribution.


```python
s = sns.distplot(train['Age'])
```


![png](/assets/images/machinelearning/titanicsession/output_105_0.png)


Which is the impact of this new feature in the survivavility:


```python
s = sns.factorplot(x="Title", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_107_0.png)


We indeed see significative differences among the classes.

### Combining Variables: Family

Both SibSp and Parch give different parts of the same information: does a passenger travel alone or with company? How many family members are with him? Thus, we will join this information together and try to differentiate meaningful classes.


```python
for dataset in [train, test]:
    dataset['Family'] = dataset['SibSp'] + dataset['Parch']
```


```python
s = sns.factorplot(x="Family", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_112_0.png)


Let's create dummy variables for each category that shows difference enough in survivavility: Being alone, 1 to 3 family members, 4 to 5 or 6 members abroad.


```python
for dataset in [train, test]:
    dataset['Alone'] = dataset['Family'].map(lambda i: 1 if i == 0 else 0)
    dataset['SmallF'] = dataset['Family'].map(lambda i: 1 if 1 <= i < 4 else 0)
    dataset['MedF'] = dataset['Family'].map(lambda i: 1 if 4 <= i < 6 else 0)
    dataset['LargeF'] = dataset['Family'].map(lambda i: 1 if i >= 6 else 0)
```

### Ticket


```python
train['Ticket'].head()
```




    0           A/5 21171
    1            PC 17599
    2    STON/O2. 3101282
    3              113803
    4              373450
    Name: Ticket, dtype: object



Observe how we have mainly two types of tickets: containing letters and not. Lettered ones may refer also to the cabin or any special place to stay of the boat. We therefore create a new variable exploiting this difference, as otherwise Ticket is just a identificative variable that needs to be dropped.


```python
train['TicketInf'] = train['Ticket'].map(lambda i: 1 if re.match('[a-zA-Z]', i) else 0)
```

Check if our assumption was right and it makes sense to use this variable:


```python
s = sns.factorplot(x="TicketInf", y="Survived", data=train,
                   size=6, kind="bar", palette="GnBu_d")
```


![png](/assets/images/machinelearning/titanicsession/output_120_0.png)


There is no real difference in survivavility rates so we are better dropping it.


```python
train.drop(['TicketInf'], axis=1, inplace=True)
```

### Age and Fare bins

We will not work with just raw values here but group them in intervals. Function pd.qcut works by using the quantiles of the data, so we group train and test to obtain the same bins. Note that we need to do this as the example is for a kaggle competition, but in real life there is not a test dataset.


```python
ids_train = train.shape[0]
data = pd.concat([train, test], axis=0)

data['BinFare'] = pd.qcut(data['Fare'], 5)
data['BinAge'] = pd.qcut(data['Age'], 4)

train['BinFare'] = data['BinFare'][:ids_train]
test['BinFare'] = data['BinFare'][ids_train:]

train['BinAge'] = data['BinAge'][:ids_train]
test['BinAge'] = data['BinAge'][ids_train:]

train["Pclass"] = train["Pclass"].astype("category")
train = pd.get_dummies(train, columns = ["Pclass"], prefix="Pc")

test["Pclass"] = test["Pclass"].astype("category")
test = pd.get_dummies(test, columns = ["Pclass"], prefix="Pc")
```


```python
train = pd.get_dummies(train, columns=['BinAge','BinFare','Embarked','Title'])
test = pd.get_dummies(test, columns=['BinAge','BinFare','Embarked','Title'])
```


```python
c = ['[',']','<',' ']
def change_colname(s):
    for elem in c:
        s=s.replace(elem,'')
        print(s)
        return s
```

XGBoost model does not allow some characters in the feature names: $]$, $[$ or $<$. Thus, we remove the ones introduced by the get_dummies.


```python
train.rename(columns=lambda x: x.replace(']',''), inplace=True)
test.rename(columns=lambda x: x.replace(']',''), inplace=True)
```

Finally map sex into 0,1:


```python
train['Sex'].replace(['male','female'],[0,1],inplace=True)
test['Sex'].replace(['male','female'],[0,1],inplace=True)
```

Now we are almost good to go. We just need to drop the identificators and *old variables* and create dummy variables.


```python
train.drop(['PassengerId','Name','Ticket','SibSp','Parch','Family','Fare','Age'], axis=1, inplace=True)
test.drop(['PassengerId','Name','Ticket','SibSp','Parch','Family','Fare','Age'], axis=1, inplace=True)
```



## Modelling

Ok, what happened so far?

- Loaded and visualized data trying to get an idea of relation between explanatory variables and the response variable or target.
- Created new features trying to exploit hidden data and increase the effect on the target.
- Encoded categorical variables.
- Dropped useless information.

We are now left with a numerical matrix of data. Training dataset will get splitted into a train set which will correspond to the data used to fit the model and a validation set to assess performance. Then, the best parameters for the model will be looked for using the GridSearch technique, which iterates over all possible combinations and outputs the best one.



```python
y_train=train['Survived']
X_train=train.drop('Survived', axis=1)
X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=0.33, random_state=13)
```

We don't want to run that expensive GS again:

![XGBoost GridSearchCV](/assets/images/machinelearning/titanicsession/GSCVxgboost.png)


```python
model = xgb.XGBClassifier(booster='gbtree', silent=1, seed=0, base_score=0.5, subsample=0.75)
grid = {'max_delta_step': 0, 
       'max_depth': 10, 
       'min_child_weight': 1, 
       'n_estimators': 240, 
       'colsample_bytree': 0.55, 
       'gamma': 3,
       'learning_rate': 0.1}
model.set_params(**grid)
model.fit(X_train,y_train)
kfold = KFold(n_splits=10, random_state=7)
results = cross_val_score(model, np.array(X_val), np.array(y_val), cv=kfold)
print("Accuracy: %.2f%% (%.2f%%)" % (results.mean()*100, results.std()*100))
plot_importance(model)
plt.show()
```

    Accuracy: 81.32% (8.67%)
    


![png](/assets/images/machinelearning/titanicsession/output_140_1.png)



```python
X_train=train.drop('Survived', axis=1)
y_train=train['Survived']
model.fit(X_train, y_train)
y_pred = model.predict(test)
```


```python
result = pd.DataFrame({'PassengerId': test_id,'Survived': y_pred}, columns=['PassengerId','Survived'])
result.head()
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
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.to_csv('../output/sub2_xgb.csv', sep=",", index=False)
```

Let's compare it with LightGBM:


```python
params = {'boosting_type': 'gbdt',
          'max_depth' : -1,
          'objective': 'binary', 
          'nthread': 5, # Updated from nthread
          'num_leaves': 64, 
          'learning_rate': 0.05, 
          'max_bin': 512, 
          'subsample_for_bin': 200,
          'subsample': 1, 
          'subsample_freq': 1, 
          'colsample_bytree': 0.8, 
          'reg_alpha': 5, 
          'reg_lambda': 10,
          'min_split_gain': 0.5, 
          'min_child_weight': 1, 
          'min_child_samples': 5, 
          'scale_pos_weight': 1,
          'num_class' : 1,
          'metric' : 'binary_error'}

# Create parameters to search
gridParams = {
    'learning_rate': [0.005,0.1],
    'n_estimators': [8,16,24,64,128],
    'num_leaves': [6,8,12,16],
    'boosting_type' : ['gbdt'],
    'objective' : ['binary'],
    'random_state' : [501], # Updated from 'seed'
    'colsample_bytree' : [0.64, 0.65, 0.66],
    'subsample' : [0.7,0.75],
    'reg_alpha' : [1,1.2],
    'reg_lambda' : [1,1.2,1.4],
    'max_depth' : [-1,2,4,6]
    }

# Create classifier to use. Note that parameters have to be input manually
# not as a dict!
mdl = lgb.LGBMClassifier(boosting_type= 'gbdt', 
          objective = 'binary', 
          n_jobs = 5, # Updated from 'nthread' 
          silent = True,
          max_depth = params['max_depth'],
          max_bin = params['max_bin'], 
          subsample_for_bin = params['subsample_for_bin'],
          subsample = params['subsample'], 
          subsample_freq = params['subsample_freq'], 
          min_split_gain = params['min_split_gain'], 
          min_child_weight = params['min_child_weight'], 
          min_child_samples = params['min_child_samples'], 
          scale_pos_weight = params['scale_pos_weight'])

# Create the grid
grid = GridSearchCV(mdl, gridParams, verbose=1, cv=4, n_jobs=-1)
```

I installed lightGBM for computing with GPU, check how incredibly faster the computations are:


```python
grid.fit(X_train, y_train)

# Print the best parameters found
print(grid.best_params_)
print(grid.best_score_)
```

    Fitting 4 folds for each of 5760 candidates, totalling 23040 fits
    

    [Parallel(n_jobs=-1)]: Done  34 tasks      | elapsed:    3.4s
    [Parallel(n_jobs=-1)]: Done 1002 tasks      | elapsed:   11.7s
    [Parallel(n_jobs=-1)]: Done 2010 tasks      | elapsed:   19.4s
    [Parallel(n_jobs=-1)]: Done 3410 tasks      | elapsed:   30.7s
    [Parallel(n_jobs=-1)]: Done 5210 tasks      | elapsed:   48.4s
    [Parallel(n_jobs=-1)]: Done 7410 tasks      | elapsed:  1.2min
    [Parallel(n_jobs=-1)]: Done 10010 tasks      | elapsed:  1.6min
    [Parallel(n_jobs=-1)]: Done 13010 tasks      | elapsed:  2.1min
    [Parallel(n_jobs=-1)]: Done 16410 tasks      | elapsed:  2.7min
    [Parallel(n_jobs=-1)]: Done 20210 tasks      | elapsed:  3.3min
    [Parallel(n_jobs=-1)]: Done 23040 out of 23040 | elapsed:  3.8min finished
    

    {'boosting_type': 'gbdt', 'colsample_bytree': 0.64, 'learning_rate': 0.005, 'max_depth': -1, 'n_estimators': 128, 'num_leaves': 8, 'objective': 'binary', 'random_state': 501, 'reg_alpha': 1, 'reg_lambda': 1, 'subsample': 0.75}
    0.8238255033557047
    


```python
params = {'boosting_type': 'gbdt', 
          'colsample_bytree': 0.64, 
          'learning_rate': 0.005, 
          'n_estimators': 128, 
          'num_leaves': 8, 
          'objective': 'binary', 
          'num_leaves': 6,
          'random_state': 501, 
          'reg_alpha': 1, 
          'reg_lambda': 1,
          'map_depth': -1,
          'subsample': 0.75}
```


```python
mdl.set_params(**params)
mdl.fit(X_train,y_train)
kfold = KFold(n_splits=10, random_state=7)
results = cross_val_score(mdl, np.array(X_val), np.array(y_val), cv=kfold)
print("Accuracy: %.2f%% (%.2f%%)" % (results.mean()*100, results.std()*100))
lgb.plot_importance(mdl)
plt.show()
```

    Accuracy: 78.57% (10.46%)
    


![png](/assets/images/machinelearning/titanicsession/output_149_1.png)



```python
mdl.fit(X_train, y_train)
y_pred = mdl.predict(test)

result = pd.DataFrame({'PassengerId': test_id,'Survived': y_pred}, columns=['PassengerId','Survived'])
result.head()
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
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.to_csv('../output/sub4_lgb.csv', sep=",", index=False)
```
