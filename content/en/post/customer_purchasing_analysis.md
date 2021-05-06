---
date: 2020-10-10T11:00:59-04:00
description: ""
featured_image: "/images/customer_purchasing_modified.jpg"
title: "Understanding Customer Purchasing Patterns"
tag: "data wrangling"

---

[comment]: <> (add image files like coord_flip_ex.png to the image folder inside the static folder)

[Link to GitHub Repository](https://github.com/kpiatti/WIFI-Fingerprinting-Project)

# Project Setup, Wrangling, & EDA

The purpose of this project was to learn how to use Anaconda, Python, Jupyter Lab, and version control/reproducibility practices and tools such as virtual environments, Git, and GitHub while analyzing a dataset of customer demographic and transaction information from an regional electronics retailer.

## Project Setup

At the beginning of every new project you need to do a few things before creating your project jupyter notebook:
1. Create a *dedicated project directory* (accessible to your Anaconda installation.
    * Note: if creating directory at root (e.g. c:), to avoid permission denied error I had to setup the Anaconda prompt to run as an administrator.
2. Create a *virtual environment* for the project (by default, you will start in the (base) environment) using the following steps:
    1. open Anaconda prompt
    2. change directories to the project dir you created in (1).
    3. enter ```conda create --name nameofenv```
    4. activate the ve ```activate nameofenv```
    5. install pkgs in ve ```conda install pandas, numpy, jupyter notebook, ipykernel```
    6. (optional) create jup nb kernel for ve ```ipython kernel install --user --name = nameofenv```
    7. deactivate the ve ```conda deactivate```
    8. re-activate the ve ```activate nameofenv```
3. With your project VE activated, launch Jupyter Lab ```jupyter lab```.

## Import Packages & Data


```python
#import data science pkgs I need
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

```python
#import data file
df = pd.read_csv('Demographic_Data.csv')
```

```python
#view first 5 lines of dataframe to confirm it looks as expected
df.head()
```

```python
#view characteristics of each variable in the dataframe
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 80000 entries, 0 to 79999
    Data columns (total 5 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   in-store  80000 non-null  int64  
     1   age       80000 non-null  int64  
     2   items     80000 non-null  int64  
     3   amount    80000 non-null  float64
     4   region    80000 non-null  int64  
    dtypes: float64(1), int64(4)
    memory usage: 3.1 MB


# Data Wrangling

The data was in good shape when I got it, so there wasn't much I needed to do to prepare it for analysis.

```python
#get rid of duplicate transactions
df = df.drop_duplicates()
```
This removed 70 rows.


```python
#check for missing values
print(df.isnull().sum())
```

    in-store    0
    age         0
    items       0
    amount      0
    region      0
    dtype: int64

# Data Exploration (EDA)

## *How is the data distributed?*


```python
#create histograms of each var in the dataframe
plt.subplots(figsize = (15,15))

#create in-store plot
plt.subplot(3,3,1)
plt.hist(df['in-store'])
plt.title('In-Store')
plt.ylabel('Transaction Count')

#create plot of age
plt.subplot(3,3,2)
plt.hist(df['age'])
plt.title('Age')

#create plot of items
plt.subplot(3,3,3)
plt.hist(df['items'], bins=8)
plt.title('Items')

#create plot of amount
plt.subplot(3,3,4)
plt.hist(df['amount'])
plt.title('Amount')
plt.ylabel('Transaction Count')

#create plot of region
plt.subplot(3,3,5)
plt.hist(df['region'])
plt.title('Region')

plt.show()
```
{{< figure src="/images/c1_subplot_array_hist1.jpeg" title="" >}}

*Inferences/Observations*

- *in-store*
  * From documentation: 1 = in-store purchase, 0 = online purchase.
  * Nominal/categorical variable.
  * Descriptive stats (e.g. mean, std) are not meaningful with nominal variables.
  * Cross-tabs and Chi-squared test useful tools for nominal variables.
- *age*
  * mean of 45.8 means the average consumer in the data set is around 45
  * std tells us about how spread out the observations are. std of 15.7 means that 68% of customers in the dataset are between 29.9 and 61.5, and 95% are between 14.2 and 77.2
  * the minimum age is 18 (uh-oh there's a problem) and the max is 86, 25% are 33 or below, 50% are 45 or below (why is this # diff than the mean?)
- *items*
  * most customers purchased between 2.5 and 6.5 items, and most spent between 114.6 and 1,557
- *region*
  * the regions are 1=north, 2=south, 3=east, 4=west.this is a nominal variable, so mean etc aren't meaningful
- **General**
  * 50% of transactions are online and 50% are in-store.
  * Customers' ages skew slightly younger, with the bulk between roughly 25-60.
  * In most transactions, customers buy 2-7 items.
  * Amount spent per transaction skews to the low end, usually less than $250.00.
  * North region has the fewest transactions, the West has the most. The South and East have roughly the same number of transactions.


  ## *How are the variables related?*

  ```python
  #define smaller sample of df (to speed up processing and improve viz)
  data_sample=df.sample(1000)

  #create cross plot of all variables in df using sample
  sns.pairplot(data_sample, hue='in-store', kind='scatter')
  ```

{{< figure src="/images/c1_pairplot.jpeg" title="" >}}

```python
#distribution plot of amount variable in each region for in-store & online
sns.set_theme(style="darkgrid")
sns.displot(
    df, x="amount", col="region", row="in-store",
    binwidth=3, height=3, facet_kws=dict(margin_titles=True),
)
```

{{< figure src="/images/output_29_1.png" >}}

{{< figure src="/images/output_37_0.png" >}}

{{< figure src="/images/output_41_0.png" >}}

{{< figure src="/images/output_43_0.png" >}}

{{< figure src="/images/output_47_0.png" >}}

{{< figure src="/images/output_60_0.png" >}}

{{< figure src="/images/output_61_0.png" >}}

{{< figure src="/images/output_64_1.png" >}}

{{< figure src="/images/output_67_0.png" >}}

{{< figure src="/images/output_29_1.png" >}}

{{< figure src="/images/output_72_0.png" >}}

{{< figure src="/images/output_78_0.png" >}}

{{< figure src="/images/output_80_0.png" >}}

{{< figure src="/images/output_83_1.png" >}}

{{< figure src="/images/output_85_0.png" >}}

{{< figure src="/images/output_87_0.png" >}}

{{< figure src="/images/output_89_0.png" >}}

{{< figure src="/images/output_90_0.png" >}}

{{< figure src="/images/output_91_1.png" >}}

{{< figure src="/images/output_98_2.png" >}}
