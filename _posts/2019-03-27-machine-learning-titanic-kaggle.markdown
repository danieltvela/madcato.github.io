---
layout:     post
title:      "Machine Learning for Titanic dissaster - Kaggle competition"
date:       2019-03-27 00:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-11.jpg"
locale:       en
lang-ref:   machine-learning-titanic-kaggle
---



# Titanic: Machine Learning from Disaster

**This is a second try to complete this Kaggle competition**

- In this file use only SVM because was the best predictor in the previous sample.
- In this file the test data will be cured more carefully looking at the historigrams.

### This Jupyter Notebook is an example of how to apply Machine Learning to the [Titanic disaster](https://www.kaggle.com/c/titanic)

The sinking of the RMS Titanic is one of the most infamous shipwrecks in history.  On April 15, 1912, during her maiden voyage, the Titanic sank after colliding with an iceberg, killing 1502 out of 2224 passengers and crew. This sensational tragedy shocked the international community and led to better safety regulations for ships.

One of the reasons that the shipwreck led to such loss of life was that there were not enough lifeboats for the passengers and crew. Although there was some element of luck involved in surviving the sinking, some groups of people were more likely to survive than others, such as women, children, and the upper-class.


### Machine Learning

In this notebook we try to practice all the classification algorithms that we learned in this course.

We load a dataset using Pandas library, and apply the following algorithms, and find the best one for this specific dataset by accuracy evaluation methods.

Lets first load required libraries:


```python
import itertools
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.ticker import NullFormatter
import pandas as pd
import numpy as np
import matplotlib.ticker as ticker
from sklearn import preprocessing
%matplotlib inline
```


```python
# notice: installing seaborn might takes a few minutes
!conda install -c anaconda seaborn -y
```

### About dataset

This dataset is about Titanic passengers. The __train.csv__ data set includes details of 891 passengers of the first travel of the Titanic:

| Variable          | Definition          | Key
|-------------------|---------------------|---------------------------|
| survival          | Survival            | 0 = No, 1 = Yes           |
| pclass            | Ticket class        | 1 = 1st, 2 = 2nd, 3 = 3rd |
| sex               | Sex                 |                           |
| Age               | Age in years        |                           |
| sibsp             | # of siblings / spouses aboard the Titanic      |
| parch             | # of parents / children aboard the Titanic      |
| ticket            | Ticket number       |                           |
| fare              | Passenger fare      |                           |
| cabin             | Cabin number        |                           |
| embarked          | Port of Embarkation |C = Cherbourg, Q = Queenstown, S = Southampton |


Lets download the datasets


```python
# Submission file
!wget -O gender_submission.csv "https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/gender_submission.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770449&Signature=npMum5eXLm0%2B95BewgF6SVHrcaamF8F1tx57ycYmUVG9ILJ9Cmr6KD9zMWlmrWkDGh%2BQSZnhb5pVHqb9wJ65B%2BaRdzCeiJiFwA6FGhq%2B4RBpw6tmc3AZqic8DCPNhmLgWw55zjT1t2fHywhbxaiypF5hA6IOQqhxDCoOgXwqtIRNV7nZj5HRXlN%2BqR8UGL%2BO%2Fpi7QYoHuGIP2ZNQCgTMSon6rQNwnhYoMAi9KVIOOHYycimx2vX6zYEP3n9VA%2FAxMysr2sFhz%2FxJWi9%2FPd3X1LJe8RqvfjajPwUhVxiLi4uT7JzxpbXDHrRfJAD3dnnvGgHIskkW5jAuMuE%2BplNNmA%3D%3D"
```

    --2019-03-25 11:56:06--  https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/gender_submission.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770449&Signature=npMum5eXLm0%2B95BewgF6SVHrcaamF8F1tx57ycYmUVG9ILJ9Cmr6KD9zMWlmrWkDGh%2BQSZnhb5pVHqb9wJ65B%2BaRdzCeiJiFwA6FGhq%2B4RBpw6tmc3AZqic8DCPNhmLgWw55zjT1t2fHywhbxaiypF5hA6IOQqhxDCoOgXwqtIRNV7nZj5HRXlN%2BqR8UGL%2BO%2Fpi7QYoHuGIP2ZNQCgTMSon6rQNwnhYoMAi9KVIOOHYycimx2vX6zYEP3n9VA%2FAxMysr2sFhz%2FxJWi9%2FPd3X1LJe8RqvfjajPwUhVxiLi4uT7JzxpbXDHrRfJAD3dnnvGgHIskkW5jAuMuE%2BplNNmA%3D%3D
    Resolving storage.googleapis.com (storage.googleapis.com)... 216.58.211.240
    Connecting to storage.googleapis.com (storage.googleapis.com)|216.58.211.240|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 3258 (3.2K) [text/csv]
    Saving to: 'gender_submission.csv'
    
    gender_submission.c 100%[===================>]   3.18K  --.-KB/s    in 0s      
    
    2019-03-25 11:56:06 (8.90 MB/s) - 'gender_submission.csv' saved [3258/3258]
    



```python
# test data file
!wget -O test.csv "https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/test.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770470&Signature=JroVG4v2wFgxCDlvpwoUTVWPP15WV5GUZ5oNImWiK7nUmcd2WO09DSaipZ4eUml5jKDuK1Qi1LjAAZWLalsKLHNRWQ6pULjFFAYozfb1CS7d7LLEXb7%2FCzqPhOapvgNV28eF9Jsl%2BkuTEXQYLf0xxigmMZh9NzUL3v1opX7FbxXQgpmy2F0Ci1NxNlKk7WGnioO45T4LnWoVhyYW5vNTVPYQa3KkQR6dc63w3jrxx6mzvSSczTLLnHj2luY9qbq%2BFMObKcaqfBzzq0l3uQCGGiMrLK5AlbCNiOVqBJveRmNk5rJ1bse4sV0giC3Fz0r7vInOfSiPr3qNyQZ1IDoOjg%3D%3D"
```

    --2019-03-25 11:56:12--  https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/test.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770470&Signature=JroVG4v2wFgxCDlvpwoUTVWPP15WV5GUZ5oNImWiK7nUmcd2WO09DSaipZ4eUml5jKDuK1Qi1LjAAZWLalsKLHNRWQ6pULjFFAYozfb1CS7d7LLEXb7%2FCzqPhOapvgNV28eF9Jsl%2BkuTEXQYLf0xxigmMZh9NzUL3v1opX7FbxXQgpmy2F0Ci1NxNlKk7WGnioO45T4LnWoVhyYW5vNTVPYQa3KkQR6dc63w3jrxx6mzvSSczTLLnHj2luY9qbq%2BFMObKcaqfBzzq0l3uQCGGiMrLK5AlbCNiOVqBJveRmNk5rJ1bse4sV0giC3Fz0r7vInOfSiPr3qNyQZ1IDoOjg%3D%3D
    Resolving storage.googleapis.com (storage.googleapis.com)... 216.58.211.240
    Connecting to storage.googleapis.com (storage.googleapis.com)|216.58.211.240|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 28629 (28K) [text/csv]
    Saving to: 'test.csv'
    
    test.csv            100%[===================>]  27.96K  --.-KB/s    in 0.02s   
    
    2019-03-25 11:56:12 (1.42 MB/s) - 'test.csv' saved [28629/28629]
    



```python
# train data file
!wget -O train.csv "https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/train.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770471&Signature=ImoWIFoTzTBcmNL9o8BRgQ3s13MMBveBDw6W7MTGjq0KNMTDqNYKmVAsBOHaSdvcUMHbtWY5CV7DwsNb5SzjbioSUvCZRaIANXS%2FMkH2MDkcFrgCwb0wXZ5rp9nRfonlYga%2FwjcqikCcQFhuwtTp9QkLbU4r8Gs0vQ3YF8WoGFEIESkmNY9k1n1sLfVuuxfdvYZ69Spox1q5UMSDq6kTETPz3In4spF1B0nj%2BwX6MVgotl6zF%2BoHpLMX3Hfu46rhOKW3JJddWYoCmFTdVjAd0w%2FEEeZ%2FQ4%2FuH7zaRQyay1UwSMr0JVyZXIlc7mZSpA4s%2B%2BSngeiQhzDiPN1FoF1YQA%3D%3D"
```

    --2019-03-25 11:56:18--  https://storage.googleapis.com/kaggle-competitions-data/kaggle/3136/train.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1553770471&Signature=ImoWIFoTzTBcmNL9o8BRgQ3s13MMBveBDw6W7MTGjq0KNMTDqNYKmVAsBOHaSdvcUMHbtWY5CV7DwsNb5SzjbioSUvCZRaIANXS%2FMkH2MDkcFrgCwb0wXZ5rp9nRfonlYga%2FwjcqikCcQFhuwtTp9QkLbU4r8Gs0vQ3YF8WoGFEIESkmNY9k1n1sLfVuuxfdvYZ69Spox1q5UMSDq6kTETPz3In4spF1B0nj%2BwX6MVgotl6zF%2BoHpLMX3Hfu46rhOKW3JJddWYoCmFTdVjAd0w%2FEEeZ%2FQ4%2FuH7zaRQyay1UwSMr0JVyZXIlc7mZSpA4s%2B%2BSngeiQhzDiPN1FoF1YQA%3D%3D
    Resolving storage.googleapis.com (storage.googleapis.com)... 216.58.211.240
    Connecting to storage.googleapis.com (storage.googleapis.com)|216.58.211.240|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 61194 (60K) [text/csv]
    Saving to: 'train.csv'
    
    train.csv           100%[===================>]  59.76K  --.-KB/s    in 0.04s   
    
    2019-03-25 11:56:18 (1.62 MB/s) - 'train.csv' saved [61194/61194]
    


### Load Data From CSV File  


```python
df = pd.read_csv('train.csv')
df = df.set_index(['PassengerId'])
df.head()
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
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <th>2</th>
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
      <th>3</th>
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
      <th>4</th>
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
      <th>5</th>
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




```python
df.shape
```




    (891, 11)



## Clean data

#### Convert 'Sex' feature into categorical values (dummies)


```python
df = pd.concat([df,pd.get_dummies(df['Sex'])], axis=1)
df.head()
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
      <th>female</th>
      <th>male</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
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
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
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
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Convert 'Embarked' feature into categorical values (dummies)


```python
df = pd.concat([df,pd.get_dummies(df['Embarked'])], axis=1)
df.head()
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
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Create a title column to manage title names


```python
title = [item.split(', ')[1].split('.')[0] for item in df['Name']]
df['Title'] = pd.Series(title)
df['Title'] = df['Title'].replace(['Don', 'Rev', 'Dr', 'Mme', 'Ms', 'Major', 'Lady', 'Sir',
                                                       'Mlle', 'Col', 'the Countess', 'Jonkheer'], 'Rare')
df.head()

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
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
      <th>Title</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mrs</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>Miss</td>
    </tr>
    <tr>
      <th>3</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mrs</td>
    </tr>
    <tr>
      <th>4</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mr</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mr</td>
    </tr>
  </tbody>
</table>
</div>



#### Convert class into categorical fields


```python
df = pd.concat([df,pd.get_dummies(df['Pclass'])], axis=1)
df.head()
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
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
      <th>Title</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>Miss</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
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
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mr</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Mr</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Convert title into categorical fields


```python
df = pd.concat([df,pd.get_dummies(df['Title'])], axis=1)
df.head()
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
      <th>...</th>
      <th>Title</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>Capt</th>
      <th>Master</th>
      <th>Miss</th>
      <th>Mr</th>
      <th>Mrs</th>
      <th>Rare</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>...</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>...</td>
      <td>Miss</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
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
      <td>...</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
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
      <td>...</td>
      <td>Mr</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>...</td>
      <td>Mr</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>



# Data visualization and pre-processing



#### Correlation matrix


```python
corr = df.corr()
corr.style.background_gradient(cmap='coolwarm').set_precision(2)
```




<style  type="text/css" >
    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col0 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col1 {
            background-color:  #9bbcff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col2 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col3 {
            background-color:  #7ea1fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col4 {
            background-color:  #92b4fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col5 {
            background-color:  #e2dad5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col6 {
            background-color:  #f29072;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col7 {
            background-color:  #85a8fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col8 {
            background-color:  #e5d8d1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col9 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col10 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col11 {
            background-color:  #f4c5ad;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col12 {
            background-color:  #c6d6f1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col13 {
            background-color:  #7699f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col14 {
            background-color:  #536edd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col15 {
            background-color:  #7b9ff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col16 {
            background-color:  #b3cdfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col17 {
            background-color:  #b9d0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col18 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col19 {
            background-color:  #688aef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col0 {
            background-color:  #6384eb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col1 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col2 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col3 {
            background-color:  #9ebeff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col4 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col5 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col6 {
            background-color:  #cbd8ee;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col7 {
            background-color:  #ecd3c5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col8 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col9 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col10 {
            background-color:  #d8dce2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col11 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col12 {
            background-color:  #89acfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col13 {
            background-color:  #c73635;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col14 {
            background-color:  #4e68d8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col15 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col16 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col17 {
            background-color:  #bfd3f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col18 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col19 {
            background-color:  #7597f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col0 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col1 {
            background-color:  #96b7ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col2 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col3 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col4 {
            background-color:  #485fd1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col5 {
            background-color:  #c5d6f2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col6 {
            background-color:  #d1dae9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col7 {
            background-color:  #e8d6cc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col8 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col9 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col10 {
            background-color:  #c6d6f1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col11 {
            background-color:  #f7bca1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col12 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col13 {
            background-color:  #799cf8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col14 {
            background-color:  #465ecf;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col15 {
            background-color:  #7295f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col16 {
            background-color:  #b1cbfc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col17 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col18 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col19 {
            background-color:  #84a7fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col0 {
            background-color:  #a9c6fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col1 {
            background-color:  #e0dbd8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col2 {
            background-color:  #485fd1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col3 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col4 {
            background-color:  #e4d9d2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col5 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col6 {
            background-color:  #ead4c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col7 {
            background-color:  #cdd9ec;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col8 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col9 {
            background-color:  #a3c2fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col10 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col11 {
            background-color:  #ccd9ed;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col12 {
            background-color:  #a7c5fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col13 {
            background-color:  #cdd9ec;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col15 {
            background-color:  #81a4fb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col16 {
            background-color:  #b1cbfc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col17 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col18 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col19 {
            background-color:  #6485ec;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col0 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col1 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col2 {
            background-color:  #6384eb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col3 {
            background-color:  #ead5c9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col4 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col5 {
            background-color:  #dbdcde;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col6 {
            background-color:  #f4c5ad;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col7 {
            background-color:  #b9d0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col8 {
            background-color:  #cad8ef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col9 {
            background-color:  #97b8ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col10 {
            background-color:  #d6dce4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col11 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col12 {
            background-color:  #b3cdfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col13 {
            background-color:  #bfd3f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col15 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col16 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col17 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col18 {
            background-color:  #9dbdff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col19 {
            background-color:  #7a9df8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col0 {
            background-color:  #e1dad6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col1 {
            background-color:  #7396f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col2 {
            background-color:  #abc8fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col3 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col4 {
            background-color:  #b6cefa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col5 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col6 {
            background-color:  #f1cdba;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col7 {
            background-color:  #c3d5f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col8 {
            background-color:  #f0cdbb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col9 {
            background-color:  #8db0fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col10 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col11 {
            background-color:  #f08b6e;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col12 {
            background-color:  #9abbff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col13 {
            background-color:  #6384eb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col15 {
            background-color:  #7b9ff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col16 {
            background-color:  #bfd3f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col17 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col18 {
            background-color:  #a6c4fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col19 {
            background-color:  #6788ee;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col0 {
            background-color:  #f7aa8c;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col1 {
            background-color:  #c0d4f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col2 {
            background-color:  #7b9ff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col3 {
            background-color:  #a6c4fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col4 {
            background-color:  #bed2f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col5 {
            background-color:  #d5dbe5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col6 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col7 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col8 {
            background-color:  #d8dce2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col9 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col10 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col11 {
            background-color:  #e2dad5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col12 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col13 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col14 {
            background-color:  #3e51c5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col15 {
            background-color:  #7b9ff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col16 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col17 {
            background-color:  #bfd3f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col18 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col19 {
            background-color:  #6384eb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col0 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col1 {
            background-color:  #e7d7ce;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col2 {
            background-color:  #abc8fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col3 {
            background-color:  #688aef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col4 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col5 {
            background-color:  #88abfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col6 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col7 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col8 {
            background-color:  #bed2f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col9 {
            background-color:  #98b9ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col10 {
            background-color:  #dfdbd9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col11 {
            background-color:  #c5d6f2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col12 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col13 {
            background-color:  #d5dbe5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col14 {
            background-color:  #4c66d6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col15 {
            background-color:  #7da0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col16 {
            background-color:  #c0d4f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col17 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col18 {
            background-color:  #a1c0ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col19 {
            background-color:  #7da0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col0 {
            background-color:  #d3dbe7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col1 {
            background-color:  #adc9fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col2 {
            background-color:  #9dbdff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col3 {
            background-color:  #779af7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col4 {
            background-color:  #779af7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col5 {
            background-color:  #e4d9d2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col6 {
            background-color:  #e7d7ce;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col7 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col8 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col9 {
            background-color:  #86a9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col10 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col11 {
            background-color:  #f5c4ac;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col12 {
            background-color:  #97b8ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col13 {
            background-color:  #9bbcff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col15 {
            background-color:  #6e90f2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col16 {
            background-color:  #c4d5f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col17 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col18 {
            background-color:  #a6c4fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col19 {
            background-color:  #7a9df8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col0 {
            background-color:  #b1cbfc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col1 {
            background-color:  #f0cdbb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col2 {
            background-color:  #8db0fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col3 {
            background-color:  #81a4fb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col4 {
            background-color:  #6384eb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col5 {
            background-color:  #97b8ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col6 {
            background-color:  #e6d7cf;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col7 {
            background-color:  #d3dbe7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col8 {
            background-color:  #b1cbfc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col9 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col10 {
            background-color:  #6c8ff1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col11 {
            background-color:  #bcd2f7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col12 {
            background-color:  #97b8ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col13 {
            background-color:  #e4d9d2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col14 {
            background-color:  #4358cb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col15 {
            background-color:  #7597f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col16 {
            background-color:  #bcd2f7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col17 {
            background-color:  #bfd3f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col18 {
            background-color:  #96b7ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col19 {
            background-color:  #6f92f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col0 {
            background-color:  #8db0fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col1 {
            background-color:  #e0dbd8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col2 {
            background-color:  #8badfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col3 {
            background-color:  #9bbcff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col4 {
            background-color:  #8caffe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col5 {
            background-color:  #8caffe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col6 {
            background-color:  #cbd8ee;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col7 {
            background-color:  #ecd3c5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col8 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col9 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col10 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col11 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col12 {
            background-color:  #d8dce2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col13 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col14 {
            background-color:  #4b64d5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col15 {
            background-color:  #8db0fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col16 {
            background-color:  #a9c6fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col17 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col18 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col19 {
            background-color:  #688aef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col0 {
            background-color:  #e6d7cf;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col1 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col2 {
            background-color:  #e3d9d3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col3 {
            background-color:  #799cf8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col4 {
            background-color:  #7597f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col5 {
            background-color:  #f59f80;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col6 {
            background-color:  #e9d5cb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col7 {
            background-color:  #cfdaea;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col8 {
            background-color:  #f2cab5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col9 {
            background-color:  #85a8fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col10 {
            background-color:  #adc9fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col11 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col12 {
            background-color:  #7396f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col13 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col15 {
            background-color:  #7a9df8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col16 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col17 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col18 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col19 {
            background-color:  #6a8bef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col0 {
            background-color:  #c4d5f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col1 {
            background-color:  #b6cefa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col2 {
            background-color:  #96b7ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col3 {
            background-color:  #799cf8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col4 {
            background-color:  #7a9df8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col5 {
            background-color:  #97b8ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col6 {
            background-color:  #e5d8d1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col7 {
            background-color:  #d4dbe6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col8 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col9 {
            background-color:  #8caffe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col10 {
            background-color:  #e8d6cc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col11 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col12 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col13 {
            background-color:  #455cce;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col15 {
            background-color:  #7b9ff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col16 {
            background-color:  #bbd1f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col17 {
            background-color:  #b9d0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col18 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col19 {
            background-color:  #7396f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col0 {
            background-color:  #6788ee;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col1 {
            background-color:  #c43032;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col2 {
            background-color:  #465ecf;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col3 {
            background-color:  #a1c0ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col4 {
            background-color:  #7ea1fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col5 {
            background-color:  #5572df;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col6 {
            background-color:  #cad8ef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col7 {
            background-color:  #edd2c3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col8 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col9 {
            background-color:  #dadce0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col10 {
            background-color:  #cad8ef;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col11 {
            background-color:  #6687ed;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col12 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col13 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col14 {
            background-color:  #4f69d9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col15 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col16 {
            background-color:  #abc8fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col17 {
            background-color:  #bed2f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col18 {
            background-color:  #a7c5fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col19 {
            background-color:  #7396f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col0 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col1 {
            background-color:  #d9dce1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col2 {
            background-color:  #94b6ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col3 {
            background-color:  #84a7fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col4 {
            background-color:  #7699f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col5 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col6 {
            background-color:  #d9dce1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col7 {
            background-color:  #e0dbd8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col8 {
            background-color:  #c9d7f0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col9 {
            background-color:  #a7c5fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col10 {
            background-color:  #cfdaea;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col11 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col12 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col13 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col14 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col15 {
            background-color:  #7a9df8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col16 {
            background-color:  #b3cdfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col17 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col18 {
            background-color:  #a2c1ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col19 {
            background-color:  #6f92f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col0 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col1 {
            background-color:  #d6dce4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col2 {
            background-color:  #8badfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col3 {
            background-color:  #8db0fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col4 {
            background-color:  #7da0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col5 {
            background-color:  #b1cbfc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col6 {
            background-color:  #dcdddd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col7 {
            background-color:  #dddcdc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col8 {
            background-color:  #c3d5f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col9 {
            background-color:  #a3c2fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col10 {
            background-color:  #d5dbe5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col11 {
            background-color:  #d4dbe6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col12 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col13 {
            background-color:  #bed2f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col14 {
            background-color:  #4358cb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col15 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col16 {
            background-color:  #9fbfff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col17 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col18 {
            background-color:  #92b4fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col19 {
            background-color:  #6687ed;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col0 {
            background-color:  #adc9fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col1 {
            background-color:  #ccd9ed;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col2 {
            background-color:  #8caffe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col3 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col4 {
            background-color:  #6e90f2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col5 {
            background-color:  #bad0f8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col6 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col7 {
            background-color:  #e2dad5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col8 {
            background-color:  #d5dbe5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col9 {
            background-color:  #aec9fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col10 {
            background-color:  #c0d4f5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col11 {
            background-color:  #dcdddd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col12 {
            background-color:  #b7cff9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col13 {
            background-color:  #afcafc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col14 {
            background-color:  #4055c8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col15 {
            background-color:  #5e7de7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col16 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col17 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col18 {
            background-color:  #7597f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col19 {
            background-color:  #5875e1;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col0 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col1 {
            background-color:  #dadce0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col2 {
            background-color:  #90b2fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col3 {
            background-color:  #96b7ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col4 {
            background-color:  #88abfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col5 {
            background-color:  #adc9fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col6 {
            background-color:  #e1dad6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col7 {
            background-color:  #d8dce2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col8 {
            background-color:  #c4d5f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col9 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col10 {
            background-color:  #cedaeb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col11 {
            background-color:  #cedaeb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col12 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col13 {
            background-color:  #c1d4f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col14 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col15 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col16 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col17 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col18 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col19 {
            background-color:  #3b4cc0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col0 {
            background-color:  #b5cdfa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col1 {
            background-color:  #d4dbe6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col2 {
            background-color:  #9ebeff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col3 {
            background-color:  #80a3fa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col4 {
            background-color:  #6f92f3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col5 {
            background-color:  #b2ccfb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col6 {
            background-color:  #dfdbd9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col7 {
            background-color:  #dadce0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col8 {
            background-color:  #ccd9ed;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col9 {
            background-color:  #9abbff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col10 {
            background-color:  #d1dae9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col11 {
            background-color:  #d8dce2;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col12 {
            background-color:  #adc9fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col13 {
            background-color:  #bcd2f7;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col14 {
            background-color:  #4257c9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col15 {
            background-color:  #6485ec;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col16 {
            background-color:  #8badfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col17 {
            background-color:  #516ddb;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col18 {
            background-color:  #b40426;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col19 {
            background-color:  #5d7ce6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col0 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col1 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col2 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col3 {
            background-color:  #7da0f9;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col4 {
            background-color:  #84a7fc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col5 {
            background-color:  #aac7fd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col6 {
            background-color:  #d7dce3;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col7 {
            background-color:  #e2dad5;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col8 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col9 {
            background-color:  #a7c5fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col10 {
            background-color:  #c7d7f0;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col11 {
            background-color:  #d2dbe8;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col12 {
            background-color:  #b6cefa;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col13 {
            background-color:  #bed2f6;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col14 {
            background-color:  #445acc;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col15 {
            background-color:  #7295f4;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col16 {
            background-color:  #a5c3fe;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col17 {
            background-color:  #8badfd;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col18 {
            background-color:  #96b7ff;
        }    #T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col19 {
            background-color:  #b40426;
        }</style>  
<table id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Survived</th> 
        <th class="col_heading level0 col1" >Pclass</th> 
        <th class="col_heading level0 col2" >Age</th> 
        <th class="col_heading level0 col3" >SibSp</th> 
        <th class="col_heading level0 col4" >Parch</th> 
        <th class="col_heading level0 col5" >Fare</th> 
        <th class="col_heading level0 col6" >female</th> 
        <th class="col_heading level0 col7" >male</th> 
        <th class="col_heading level0 col8" >C</th> 
        <th class="col_heading level0 col9" >Q</th> 
        <th class="col_heading level0 col10" >S</th> 
        <th class="col_heading level0 col11" >1</th> 
        <th class="col_heading level0 col12" >2</th> 
        <th class="col_heading level0 col13" >3</th> 
        <th class="col_heading level0 col14" >Capt</th> 
        <th class="col_heading level0 col15" >Master</th> 
        <th class="col_heading level0 col16" >Miss</th> 
        <th class="col_heading level0 col17" >Mr</th> 
        <th class="col_heading level0 col18" >Mrs</th> 
        <th class="col_heading level0 col19" >Rare</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row0" class="row_heading level0 row0" >Survived</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col0" class="data row0 col0" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col1" class="data row0 col1" >-0.34</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col2" class="data row0 col2" >-0.077</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col3" class="data row0 col3" >-0.035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col4" class="data row0 col4" >0.082</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col5" class="data row0 col5" >0.26</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col6" class="data row0 col6" >0.54</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col7" class="data row0 col7" >-0.54</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col8" class="data row0 col8" >0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col9" class="data row0 col9" >0.0037</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col10" class="data row0 col10" >-0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col11" class="data row0 col11" >0.29</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col12" class="data row0 col12" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col13" class="data row0 col13" >-0.32</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col14" class="data row0 col14" >0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col15" class="data row0 col15" >-0.0039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col16" class="data row0 col16" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col17" class="data row0 col17" >0.0091</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col18" class="data row0 col18" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row0_col19" class="data row0 col19" >-0.027</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row1" class="row_heading level0 row1" >Pclass</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col0" class="data row1 col0" >-0.34</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col1" class="data row1 col1" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col2" class="data row1 col2" >-0.37</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col3" class="data row1 col3" >0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col4" class="data row1 col4" >0.018</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col5" class="data row1 col5" >-0.55</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col6" class="data row1 col6" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col7" class="data row1 col7" >0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col8" class="data row1 col8" >-0.24</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col9" class="data row1 col9" >0.22</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col10" class="data row1 col10" >0.082</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col11" class="data row1 col11" >-0.89</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col12" class="data row1 col12" >-0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col13" class="data row1 col13" >0.92</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col14" class="data row1 col14" >0.028</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col15" class="data row1 col15" >0.011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col16" class="data row1 col16" >-0.061</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col17" class="data row1 col17" >0.04</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col18" class="data row1 col18" >-0.0061</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row1_col19" class="data row1 col19" >0.016</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row2" class="row_heading level0 row2" >Age</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col0" class="data row2 col0" >-0.077</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col1" class="data row2 col1" >-0.37</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col2" class="data row2 col2" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col3" class="data row2 col3" >-0.31</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col4" class="data row2 col4" >-0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col5" class="data row2 col5" >0.096</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col6" class="data row2 col6" >-0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col7" class="data row2 col7" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col8" class="data row2 col8" >0.036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col9" class="data row2 col9" >-0.022</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col10" class="data row2 col10" >-0.033</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col11" class="data row2 col11" >0.35</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col12" class="data row2 col12" >0.007</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col13" class="data row2 col13" >-0.31</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col14" class="data row2 col14" >0.0034</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col15" class="data row2 col15" >-0.035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col16" class="data row2 col16" >-0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col17" class="data row2 col17" >-0.014</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col18" class="data row2 col18" >0.04</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row2_col19" class="data row2 col19" >0.067</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row3" class="row_heading level0 row3" >SibSp</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col0" class="data row3 col0" >-0.035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col1" class="data row3 col1" >0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col2" class="data row3 col2" >-0.31</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col3" class="data row3 col3" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col4" class="data row3 col4" >0.41</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col5" class="data row3 col5" >0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col6" class="data row3 col6" >0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col7" class="data row3 col7" >-0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col8" class="data row3 col8" >-0.06</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col9" class="data row3 col9" >-0.026</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col10" class="data row3 col10" >0.071</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col11" class="data row3 col11" >-0.055</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col12" class="data row3 col12" >-0.056</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col13" class="data row3 col13" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col14" class="data row3 col14" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col15" class="data row3 col15" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col16" class="data row3 col16" >-0.031</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col17" class="data row3 col17" >0.052</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col18" class="data row3 col18" >-0.027</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row3_col19" class="data row3 col19" >-0.04</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row4" class="row_heading level0 row4" >Parch</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col0" class="data row4 col0" >0.082</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col1" class="data row4 col1" >0.018</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col2" class="data row4 col2" >-0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col3" class="data row4 col3" >0.41</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col4" class="data row4 col4" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col5" class="data row4 col5" >0.22</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col6" class="data row4 col6" >0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col7" class="data row4 col7" >-0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col8" class="data row4 col8" >-0.011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col9" class="data row4 col9" >-0.081</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col10" class="data row4 col10" >0.063</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col11" class="data row4 col11" >-0.018</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col12" class="data row4 col12" >-0.00073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col13" class="data row4 col13" >0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col14" class="data row4 col14" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col15" class="data row4 col15" >0.012</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col16" class="data row4 col16" >-0.043</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col17" class="data row4 col17" >0.048</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col18" class="data row4 col18" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row4_col19" class="data row4 col19" >0.034</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row5" class="row_heading level0 row5" >Fare</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col0" class="data row5 col0" >0.26</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col1" class="data row5 col1" >-0.55</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col2" class="data row5 col2" >0.096</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col3" class="data row5 col3" >0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col4" class="data row5 col4" >0.22</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col5" class="data row5 col5" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col6" class="data row5 col6" >0.18</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col7" class="data row5 col7" >-0.18</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col8" class="data row5 col8" >0.27</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col9" class="data row5 col9" >-0.12</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col10" class="data row5 col10" >-0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col11" class="data row5 col11" >0.59</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col12" class="data row5 col12" >-0.12</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col13" class="data row5 col13" >-0.41</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col14" class="data row5 col14" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col15" class="data row5 col15" >-0.0019</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col16" class="data row5 col16" >0.041</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col17" class="data row5 col17" >-0.022</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col18" class="data row5 col18" >0.0039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row5_col19" class="data row5 col19" >-0.033</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row6" class="row_heading level0 row6" >female</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col0" class="data row6 col0" >0.54</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col1" class="data row6 col1" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col2" class="data row6 col2" >-0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col3" class="data row6 col3" >0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col4" class="data row6 col4" >0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col5" class="data row6 col5" >0.18</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col6" class="data row6 col6" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col7" class="data row6 col7" >-1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col8" class="data row6 col8" >0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col9" class="data row6 col9" >0.074</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col10" class="data row6 col10" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col11" class="data row6 col11" >0.098</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col12" class="data row6 col12" >0.065</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col13" class="data row6 col13" >-0.14</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col14" class="data row6 col14" >-0.025</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col15" class="data row6 col15" >-0.0011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col16" class="data row6 col16" >-0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col17" class="data row6 col17" >0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col18" class="data row6 col18" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row6_col19" class="data row6 col19" >-0.044</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row7" class="row_heading level0 row7" >male</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col0" class="data row7 col0" >-0.54</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col1" class="data row7 col1" >0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col2" class="data row7 col2" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col3" class="data row7 col3" >-0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col4" class="data row7 col4" >-0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col5" class="data row7 col5" >-0.18</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col6" class="data row7 col6" >-1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col7" class="data row7 col7" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col8" class="data row7 col8" >-0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col9" class="data row7 col9" >-0.074</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col10" class="data row7 col10" >0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col11" class="data row7 col11" >-0.098</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col12" class="data row7 col12" >-0.065</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col13" class="data row7 col13" >0.14</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col14" class="data row7 col14" >0.025</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col15" class="data row7 col15" >0.0011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col16" class="data row7 col16" >0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col17" class="data row7 col17" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col18" class="data row7 col18" >-0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row7_col19" class="data row7 col19" >0.044</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row8" class="row_heading level0 row8" >C</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col0" class="data row8 col0" >0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col1" class="data row8 col1" >-0.24</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col2" class="data row8 col2" >0.036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col3" class="data row8 col3" >-0.06</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col4" class="data row8 col4" >-0.011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col5" class="data row8 col5" >0.27</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col6" class="data row8 col6" >0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col7" class="data row8 col7" >-0.083</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col8" class="data row8 col8" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col9" class="data row8 col9" >-0.15</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col10" class="data row8 col10" >-0.78</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col11" class="data row8 col11" >0.3</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col12" class="data row8 col12" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col13" class="data row8 col13" >-0.15</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col14" class="data row8 col14" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col15" class="data row8 col15" >-0.049</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col16" class="data row8 col16" >0.062</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col17" class="data row8 col17" >-0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col18" class="data row8 col18" >0.0036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row8_col19" class="data row8 col19" >0.036</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row9" class="row_heading level0 row9" >Q</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col0" class="data row9 col0" >0.0037</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col1" class="data row9 col1" >0.22</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col2" class="data row9 col2" >-0.022</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col3" class="data row9 col3" >-0.026</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col4" class="data row9 col4" >-0.081</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col5" class="data row9 col5" >-0.12</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col6" class="data row9 col6" >0.074</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col7" class="data row9 col7" >-0.074</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col8" class="data row9 col8" >-0.15</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col9" class="data row9 col9" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col10" class="data row9 col10" >-0.5</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col11" class="data row9 col11" >-0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col12" class="data row9 col12" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col13" class="data row9 col13" >0.24</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col14" class="data row9 col14" >-0.01</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col15" class="data row9 col15" >-0.028</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col16" class="data row9 col16" >0.023</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col17" class="data row9 col17" >0.036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col18" class="data row9 col18" >-0.067</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row9_col19" class="data row9 col19" >-0.0059</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row10" class="row_heading level0 row10" >S</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col0" class="data row10 col0" >-0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col1" class="data row10 col1" >0.082</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col2" class="data row10 col2" >-0.033</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col3" class="data row10 col3" >0.071</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col4" class="data row10 col4" >0.063</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col5" class="data row10 col5" >-0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col6" class="data row10 col6" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col7" class="data row10 col7" >0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col8" class="data row10 col8" >-0.78</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col9" class="data row10 col9" >-0.5</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col10" class="data row10 col10" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col11" class="data row10 col11" >-0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col12" class="data row10 col12" >0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col13" class="data row10 col13" >-0.0095</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col14" class="data row10 col14" >0.021</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col15" class="data row10 col15" >0.062</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col16" class="data row10 col16" >-0.066</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col17" class="data row10 col17" >0.015</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col18" class="data row10 col18" >0.034</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row10_col19" class="data row10 col19" >-0.027</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row11" class="row_heading level0 row11" >1</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col0" class="data row11 col0" >0.29</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col1" class="data row11 col1" >-0.89</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col2" class="data row11 col2" >0.35</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col3" class="data row11 col3" >-0.055</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col4" class="data row11 col4" >-0.018</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col5" class="data row11 col5" >0.59</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col6" class="data row11 col6" >0.098</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col7" class="data row11 col7" >-0.098</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col8" class="data row11 col8" >0.3</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col9" class="data row11 col9" >-0.16</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col10" class="data row11 col10" >-0.17</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col11" class="data row11 col11" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col12" class="data row11 col12" >-0.29</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col13" class="data row11 col13" >-0.63</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col14" class="data row11 col14" >-0.019</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col15" class="data row11 col15" >-0.0088</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col16" class="data row11 col16" >0.051</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col17" class="data row11 col17" >-0.043</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col18" class="data row11 col18" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row11_col19" class="data row11 col19" >-0.02</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row12" class="row_heading level0 row12" >2</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col0" class="data row12 col0" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col1" class="data row12 col1" >-0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col2" class="data row12 col2" >0.007</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col3" class="data row12 col3" >-0.056</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col4" class="data row12 col4" >-0.00073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col5" class="data row12 col5" >-0.12</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col6" class="data row12 col6" >0.065</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col7" class="data row12 col7" >-0.065</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col8" class="data row12 col8" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col9" class="data row12 col9" >-0.13</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col10" class="data row12 col10" >0.19</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col11" class="data row12 col11" >-0.29</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col12" class="data row12 col12" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col13" class="data row12 col13" >-0.57</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col14" class="data row12 col14" >-0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col15" class="data row12 col15" >-0.0035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col16" class="data row12 col16" >0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col17" class="data row12 col17" >0.0081</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col18" class="data row12 col18" >-0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row12_col19" class="data row12 col19" >0.01</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row13" class="row_heading level0 row13" >3</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col0" class="data row13 col0" >-0.32</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col1" class="data row13 col1" >0.92</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col2" class="data row13 col2" >-0.31</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col3" class="data row13 col3" >0.093</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col4" class="data row13 col4" >0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col5" class="data row13 col5" >-0.41</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col6" class="data row13 col6" >-0.14</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col7" class="data row13 col7" >0.14</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col8" class="data row13 col8" >-0.15</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col9" class="data row13 col9" >0.24</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col10" class="data row13 col10" >-0.0095</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col11" class="data row13 col11" >-0.63</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col12" class="data row13 col12" >-0.57</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col13" class="data row13 col13" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col14" class="data row13 col14" >0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col15" class="data row13 col15" >0.01</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col16" class="data row13 col16" >-0.058</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col17" class="data row13 col17" >0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col18" class="data row13 col18" >0.0073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row13_col19" class="data row13 col19" >0.009</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row14" class="row_heading level0 row14" >Capt</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col0" class="data row14 col0" >0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col1" class="data row14 col1" >0.028</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col2" class="data row14 col2" >0.0034</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col3" class="data row14 col3" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col4" class="data row14 col4" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col5" class="data row14 col5" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col6" class="data row14 col6" >-0.025</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col7" class="data row14 col7" >0.025</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col8" class="data row14 col8" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col9" class="data row14 col9" >-0.01</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col10" class="data row14 col10" >0.021</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col11" class="data row14 col11" >-0.019</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col12" class="data row14 col12" >-0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col13" class="data row14 col13" >0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col14" class="data row14 col14" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col15" class="data row14 col15" >-0.0073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col16" class="data row14 col16" >-0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col17" class="data row14 col17" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col18" class="data row14 col18" >-0.014</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row14_col19" class="data row14 col19" >-0.0058</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row15" class="row_heading level0 row15" >Master</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col0" class="data row15 col0" >-0.0039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col1" class="data row15 col1" >0.011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col2" class="data row15 col2" >-0.035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col3" class="data row15 col3" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col4" class="data row15 col4" >0.012</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col5" class="data row15 col5" >-0.0019</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col6" class="data row15 col6" >-0.0011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col7" class="data row15 col7" >0.0011</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col8" class="data row15 col8" >-0.049</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col9" class="data row15 col9" >-0.028</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col10" class="data row15 col10" >0.062</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col11" class="data row15 col11" >-0.0088</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col12" class="data row15 col12" >-0.0035</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col13" class="data row15 col13" >0.01</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col14" class="data row15 col14" >-0.0073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col15" class="data row15 col15" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col16" class="data row15 col16" >-0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col17" class="data row15 col17" >-0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col18" class="data row15 col18" >-0.088</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row15_col19" class="data row15 col19" >-0.038</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row16" class="row_heading level0 row16" >Miss</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col0" class="data row16 col0" >-0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col1" class="data row16 col1" >-0.061</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col2" class="data row16 col2" >-0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col3" class="data row16 col3" >-0.031</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col4" class="data row16 col4" >-0.043</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col5" class="data row16 col5" >0.041</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col6" class="data row16 col6" >-0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col7" class="data row16 col7" >0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col8" class="data row16 col8" >0.062</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col9" class="data row16 col9" >0.023</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col10" class="data row16 col10" >-0.066</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col11" class="data row16 col11" >0.051</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col12" class="data row16 col12" >0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col13" class="data row16 col13" >-0.058</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col14" class="data row16 col14" >-0.017</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col15" class="data row16 col15" >-0.11</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col16" class="data row16 col16" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col17" class="data row16 col17" >-0.59</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col18" class="data row16 col18" >-0.2</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row16_col19" class="data row16 col19" >-0.088</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row17" class="row_heading level0 row17" >Mr</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col0" class="data row17 col0" >0.0091</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col1" class="data row17 col1" >0.04</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col2" class="data row17 col2" >-0.014</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col3" class="data row17 col3" >0.052</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col4" class="data row17 col4" >0.048</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col5" class="data row17 col5" >-0.022</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col6" class="data row17 col6" >0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col7" class="data row17 col7" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col8" class="data row17 col8" >-0.042</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col9" class="data row17 col9" >0.036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col10" class="data row17 col10" >0.015</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col11" class="data row17 col11" >-0.043</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col12" class="data row17 col12" >0.0081</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col13" class="data row17 col13" >0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col14" class="data row17 col14" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col15" class="data row17 col15" >-0.25</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col16" class="data row17 col16" >-0.59</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col17" class="data row17 col17" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col18" class="data row17 col18" >-0.47</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row17_col19" class="data row17 col19" >-0.2</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row18" class="row_heading level0 row18" >Mrs</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col0" class="data row18 col0" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col1" class="data row18 col1" >-0.0061</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col2" class="data row18 col2" >0.04</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col3" class="data row18 col3" >-0.027</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col4" class="data row18 col4" >-0.039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col5" class="data row18 col5" >0.0039</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col6" class="data row18 col6" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col7" class="data row18 col7" >-0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col8" class="data row18 col8" >0.0036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col9" class="data row18 col9" >-0.067</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col10" class="data row18 col10" >0.034</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col11" class="data row18 col11" >0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col12" class="data row18 col12" >-0.03</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col13" class="data row18 col13" >0.0073</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col14" class="data row18 col14" >-0.014</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col15" class="data row18 col15" >-0.088</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col16" class="data row18 col16" >-0.2</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col17" class="data row18 col17" >-0.47</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col18" class="data row18 col18" >1</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row18_col19" class="data row18 col19" >-0.07</td> 
    </tr>    <tr> 
        <th id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2level0_row19" class="row_heading level0 row19" >Rare</th> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col0" class="data row19 col0" >-0.027</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col1" class="data row19 col1" >0.016</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col2" class="data row19 col2" >0.067</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col3" class="data row19 col3" >-0.04</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col4" class="data row19 col4" >0.034</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col5" class="data row19 col5" >-0.033</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col6" class="data row19 col6" >-0.044</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col7" class="data row19 col7" >0.044</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col8" class="data row19 col8" >0.036</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col9" class="data row19 col9" >-0.0059</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col10" class="data row19 col10" >-0.027</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col11" class="data row19 col11" >-0.02</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col12" class="data row19 col12" >0.01</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col13" class="data row19 col13" >0.009</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col14" class="data row19 col14" >-0.0058</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col15" class="data row19 col15" >-0.038</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col16" class="data row19 col16" >-0.088</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col17" class="data row19 col17" >-0.2</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col18" class="data row19 col18" >-0.07</td> 
        <td id="T_826cd8a6_4fc0_11e9_9ac8_58ef687f10d2row19_col19" class="data row19 col19" >1</td> 
    </tr></tbody> 
</table> 



The mayor correlation with 'Survived' is the 'female' feature. Woman had more probability to survive the disaster.
Also we can see that there were more female than male who travel with siblins

Lets plot some columns to underestand data better:


```python
import seaborn as sns

bins = np.linspace(df.Age.min(), df.Age.max(), 10)
g = sns.FacetGrid(df, col="male", hue="Pclass", palette="Set1", col_wrap=2)
g.map(plt.hist, 'Age', bins=bins, ec="k")

g.axes[-1].legend()
plt.show()
```


![png]({{ site.url}}/assets/output_29_0.png)


- **First grid**: Females per Age
- **Second grid**: Males per Age
- **Red**: First class
- **Blue**: Second class
- **Green**: Third class


```python
bins = np.linspace(df.Age.min(), df.Age.max(), 10)
g = sns.FacetGrid(df, col="male", hue="Survived", palette="Set1", col_wrap=2)
g.map(plt.hist, 'Age', bins=bins, ec="k")

g.axes[-1].legend()
plt.show()
```


![png]({{ site.url}}/assets/output_31_0.png)


All woman survived. Child males also survived.


```python
# Take only males to distinguish which classes survived the most
male_df = df[df.male != 0]
male_df.head()
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
      <th>...</th>
      <th>Title</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>Capt</th>
      <th>Master</th>
      <th>Miss</th>
      <th>Mr</th>
      <th>Mrs</th>
      <th>Rare</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
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
      <td>...</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
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
      <td>...</td>
      <td>Mr</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>3</td>
      <td>Moran, Mr. James</td>
      <td>male</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>330877</td>
      <td>8.4583</td>
      <td>NaN</td>
      <td>...</td>
      <td>Mr</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0</td>
      <td>1</td>
      <td>McCarthy, Mr. Timothy J</td>
      <td>male</td>
      <td>54.0</td>
      <td>0</td>
      <td>0</td>
      <td>17463</td>
      <td>51.8625</td>
      <td>E46</td>
      <td>...</td>
      <td>Master</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>3</td>
      <td>Palsson, Master. Gosta Leonard</td>
      <td>male</td>
      <td>2.0</td>
      <td>3</td>
      <td>1</td>
      <td>349909</td>
      <td>21.0750</td>
      <td>NaN</td>
      <td>...</td>
      <td>Mrs</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>




```python
bins = np.linspace(male_df.Age.min(), male_df.Age.max(), 10)
g = sns.FacetGrid(male_df, col="Pclass", hue="Survived", palette="Set1", col_wrap=2)
g.map(plt.hist, 'Age', bins=bins, ec="k")

g.axes[-1].legend()
plt.show()
```


![png]({{ site.url}}/assets/output_34_0.png)


First class males had the most survival probability

#### Check number of survivors by sex


```python
sns.countplot(data=df, x='Survived', hue='Sex')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1a1a951048>




![png]({{ site.url}}/assets/output_37_1.png)


#### Survived by class


```python
sns.countplot(data=df, x='Survived', hue='Pclass')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1a1c1ec908>




![png]({{ site.url}}/assets/output_39_1.png)


## Prepare data

#### Remove unneeded features


```python
df.drop(['Name', 'Sex', 'Ticket', 'Fare', 'Embarked', 'Cabin', 'Pclass', 'Title'], axis=1, inplace=True)
df.head()
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
      <th>Survived</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>Capt</th>
      <th>Master</th>
      <th>Miss</th>
      <th>Mr</th>
      <th>Mrs</th>
      <th>Rare</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>26.0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>35.0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0</td>
      <td>35.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Drop NAN
df = df.dropna()
df.shape
```




    (714, 18)




```python
Feature = df.astype(np.float64)
X = Feature.drop(['Survived'], axis=1)
X.shape
```




    (714, 17)



What are our lables?


```python
y = df['Survived'].values
y[0:5]
```




    array([0, 1, 1, 1, 0])



## Normalize Data 

Data Standardization give data zero mean and unit variance (technically should be done after train test split )


```python
# X= preprocessing.StandardScaler().fit(X).transform(X)
X[0:5]
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
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>Capt</th>
      <th>Master</th>
      <th>Miss</th>
      <th>Mr</th>
      <th>Mrs</th>
      <th>Rare</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>22.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>38.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>26.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>35.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>35.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



# Classification 

Use only:

- Support Vector Machine


# Support Vector Machine


```python
from sklearn import svm
model_svm = svm.SVC(kernel='linear')
model_svm.fit(X, y)
model_svm
```




    SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
      decision_function_shape='ovr', degree=3, gamma='auto_deprecated',
      kernel='linear', max_iter=-1, probability=False, random_state=None,
      shrinking=True, tol=0.001, verbose=False)



## Let's prepare a submission for the Kaggle competion


```python
# Load test data
df_test = pd.read_csv('test.csv')
df_test.head()
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
      <td>892</td>
      <td>3</td>
      <td>Kelly, Mr. James</td>
      <td>male</td>
      <td>34.5</td>
      <td>0</td>
      <td>0</td>
      <td>330911</td>
      <td>7.8292</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>3</td>
      <td>Wilkes, Mrs. James (Ellen Needs)</td>
      <td>female</td>
      <td>47.0</td>
      <td>1</td>
      <td>0</td>
      <td>363272</td>
      <td>7.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>2</td>
      <td>Myles, Mr. Thomas Francis</td>
      <td>male</td>
      <td>62.0</td>
      <td>0</td>
      <td>0</td>
      <td>240276</td>
      <td>9.6875</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>3</td>
      <td>Wirz, Mr. Albert</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>315154</td>
      <td>8.6625</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>3</td>
      <td>Hirvonen, Mrs. Alexander (Helga E Lindqvist)</td>
      <td>female</td>
      <td>22.0</td>
      <td>1</td>
      <td>1</td>
      <td>3101298</td>
      <td>12.2875</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>



### Cure data

#### Fill the Age using rules

Each rule will depend on the rest of the data:

- If the name of a woman starts with Mrs and have children, put an age between 25 and 40
- If a male travels alone, put the average age of the males traveling alone.
- 


```python
# Find women with children/parents with Mrs in his name

df_test2 = df_test[df_test['Name'].str.contains("Mrs", regex=False)]
avg = df_test2['Age'].mean()
df_test.loc[df_test['Name'].str.contains("Mrs", regex=False), 'Age'] = df_test.loc[df_test['Name'].str.contains("Mrs", regex=False), 'Age'].fillna(avg)
df_test.loc[df_test['Name'].str.contains("Ms.", regex=False), 'Age'] = df_test.loc[df_test['Name'].str.contains("Ms.", regex=False), 'Age'].fillna(avg)
df_test.head()
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
      <td>892</td>
      <td>3</td>
      <td>Kelly, Mr. James</td>
      <td>male</td>
      <td>34.5</td>
      <td>0</td>
      <td>0</td>
      <td>330911</td>
      <td>7.8292</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>3</td>
      <td>Wilkes, Mrs. James (Ellen Needs)</td>
      <td>female</td>
      <td>47.0</td>
      <td>1</td>
      <td>0</td>
      <td>363272</td>
      <td>7.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>2</td>
      <td>Myles, Mr. Thomas Francis</td>
      <td>male</td>
      <td>62.0</td>
      <td>0</td>
      <td>0</td>
      <td>240276</td>
      <td>9.6875</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>3</td>
      <td>Wirz, Mr. Albert</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>315154</td>
      <td>8.6625</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>3</td>
      <td>Hirvonen, Mrs. Alexander (Helga E Lindqvist)</td>
      <td>female</td>
      <td>22.0</td>
      <td>1</td>
      <td>1</td>
      <td>3101298</td>
      <td>12.2875</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Find men traveling alone and put the average of all

df_test2 = df_test.loc[(df_test['Sex'] == 'male') & (df_test['Parch'] == 0) & (df_test['SibSp'] == 0)]
avg = df_test2['Age'].mean()
df_test.loc[(df_test['Sex'] == 'male') & (df_test['Parch'] == 0) & (df_test['SibSp'] == 0), 'Age'] = df_test.loc[(df_test['Sex'] == 'male') & (df_test['Parch'] == 0) & (df_test['SibSp'] == 0), 'Age'].fillna(avg)
df_test.head()
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
      <td>892</td>
      <td>3</td>
      <td>Kelly, Mr. James</td>
      <td>male</td>
      <td>34.5</td>
      <td>0</td>
      <td>0</td>
      <td>330911</td>
      <td>7.8292</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>3</td>
      <td>Wilkes, Mrs. James (Ellen Needs)</td>
      <td>female</td>
      <td>47.0</td>
      <td>1</td>
      <td>0</td>
      <td>363272</td>
      <td>7.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>2</td>
      <td>Myles, Mr. Thomas Francis</td>
      <td>male</td>
      <td>62.0</td>
      <td>0</td>
      <td>0</td>
      <td>240276</td>
      <td>9.6875</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>3</td>
      <td>Wirz, Mr. Albert</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>315154</td>
      <td>8.6625</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>3</td>
      <td>Hirvonen, Mrs. Alexander (Helga E Lindqvist)</td>
      <td>female</td>
      <td>22.0</td>
      <td>1</td>
      <td>1</td>
      <td>3101298</td>
      <td>12.2875</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Fill with average of Miss each one nan

df_test2 = df_test[df_test['Name'].str.contains("Miss", regex=False)]
avg = df_test2['Age'].mean()
print(avg)
df_test.loc[df_test['Name'].str.contains("Miss", regex=False), 'Age'] = df_test.loc[df_test['Name'].str.contains("Miss", regex=False), 'Age'].fillna(avg)
df_test.head()

```

    21.774843750000002





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
      <td>892</td>
      <td>3</td>
      <td>Kelly, Mr. James</td>
      <td>male</td>
      <td>34.5</td>
      <td>0</td>
      <td>0</td>
      <td>330911</td>
      <td>7.8292</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>3</td>
      <td>Wilkes, Mrs. James (Ellen Needs)</td>
      <td>female</td>
      <td>47.0</td>
      <td>1</td>
      <td>0</td>
      <td>363272</td>
      <td>7.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>2</td>
      <td>Myles, Mr. Thomas Francis</td>
      <td>male</td>
      <td>62.0</td>
      <td>0</td>
      <td>0</td>
      <td>240276</td>
      <td>9.6875</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>3</td>
      <td>Wirz, Mr. Albert</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>315154</td>
      <td>8.6625</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>3</td>
      <td>Hirvonen, Mrs. Alexander (Helga E Lindqvist)</td>
      <td>female</td>
      <td>22.0</td>
      <td>1</td>
      <td>1</td>
      <td>3101298</td>
      <td>12.2875</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>




```python
# The rest of males put the average of all males

df_test2 = df_test.loc[(df_test['Sex'] == 'male')]
avg = df_test2['Age'].mean()
df_test.loc[(df_test['Sex'] == 'male'), 'Age'] = df_test.loc[(df_test['Sex'] == 'male'), 'Age'].fillna(avg)
df_test.head()
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
      <td>892</td>
      <td>3</td>
      <td>Kelly, Mr. James</td>
      <td>male</td>
      <td>34.5</td>
      <td>0</td>
      <td>0</td>
      <td>330911</td>
      <td>7.8292</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>893</td>
      <td>3</td>
      <td>Wilkes, Mrs. James (Ellen Needs)</td>
      <td>female</td>
      <td>47.0</td>
      <td>1</td>
      <td>0</td>
      <td>363272</td>
      <td>7.0000</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>894</td>
      <td>2</td>
      <td>Myles, Mr. Thomas Francis</td>
      <td>male</td>
      <td>62.0</td>
      <td>0</td>
      <td>0</td>
      <td>240276</td>
      <td>9.6875</td>
      <td>NaN</td>
      <td>Q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>895</td>
      <td>3</td>
      <td>Wirz, Mr. Albert</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>315154</td>
      <td>8.6625</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>896</td>
      <td>3</td>
      <td>Hirvonen, Mrs. Alexander (Helga E Lindqvist)</td>
      <td>female</td>
      <td>22.0</td>
      <td>1</td>
      <td>1</td>
      <td>3101298</td>
      <td>12.2875</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_test[df_test.Age.isnull()]

# No more nan in Age
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
  </tbody>
</table>
</div>



#### Prepare test data the same way the train data


```python
df_test = df_test.set_index(['PassengerId'])
df_test = pd.concat([df_test,pd.get_dummies(df_test['Sex'])], axis=1)
df_test = pd.concat([df_test,pd.get_dummies(df_test['Embarked'])], axis=1)
title = [item.split(', ')[1].split('.')[0] for item in df_test['Name']]
df_test['Title'] = pd.Series(title, index=df_test.index)
df_test['Title'] = df_test['Title'].replace(['Don', 'Rev', 'Dr', 'Mme', 'Ms', 'Major', 'Lady', 'Sir',
                                                       'Mlle', 'Col', 'the Countess', 'Jonkheer'], 'Rare')
```


```python
df_test = pd.concat([df_test,pd.get_dummies(df_test['Pclass'])], axis=1)
df_test = pd.concat([df_test,pd.get_dummies(df_test['Title'])], axis=1)
df_test.drop(['Name', 'Sex', 'Fare', 'Ticket', 'Embarked', 'Cabin', 'Pclass', 'Title'], axis=1, inplace=True)
df_test2 = df_test.dropna()
df_test2.shape
```




    (418, 17)




```python
Feature = df_test.astype(np.float64)
X_test = Feature
X_test.shape
```




    (418, 17)




```python
# X_test = preprocessing.StandardScaler().fit(X_test).transform(X_test)
X_test[0:5]
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
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>female</th>
      <th>male</th>
      <th>C</th>
      <th>Q</th>
      <th>S</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>Dona</th>
      <th>Master</th>
      <th>Miss</th>
      <th>Mr</th>
      <th>Mrs</th>
      <th>Rare</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>892</th>
      <td>34.5</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>893</th>
      <td>47.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>894</th>
      <td>62.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>895</th>
      <td>27.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>896</th>
      <td>22.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
yhat_test = model_svm.predict(X_test)
yhat_test[0:50]
```




    array([0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 0, 0,
           1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1,
           1, 0, 0, 0, 1, 1])




```python
from sklearn.linear_model import LogisticRegression
logmodel = LogisticRegression(solver='liblinear')
logmodel.fit(X, y)
yhat_test = logmodel.predict(X_test)
```

## Prepare submission


```python
submission_df = pd.read_csv('gender_submission.csv', index_col=False)
submission_df.head()
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
results = pd.DataFrame(yhat_test)
def transformValueR(x):
    if x > 0.5:
        return 1
    return 0

fixed_results = results[0].apply(lambda x: transformValueR(x))
t = pd.concat([submission_df['PassengerId'], fixed_results], axis=1, keys=['PassengerId', 'Survived'])
t = t.set_index('PassengerId')
t.to_csv("submission_cured_more_dummies.csv")
t.head()
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
      <th>Survived</th>
    </tr>
    <tr>
      <th>PassengerId</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>892</th>
      <td>0</td>
    </tr>
    <tr>
      <th>893</th>
      <td>0</td>
    </tr>
    <tr>
      <th>894</th>
      <td>0</td>
    </tr>
    <tr>
      <th>895</th>
      <td>0</td>
    </tr>
    <tr>
      <th>896</th>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>


