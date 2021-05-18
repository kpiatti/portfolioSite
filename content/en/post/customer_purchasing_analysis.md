---
date: 2020-11-17
description: ""
featured_image: "/images/customer_purchasing_feature.jpg"
title: "Analyzing Purchasing Patterns with Python"
tag: "data wrangling"
show_reading_time: true
---


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

#view first 5 lines of dataframe to verify imports
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
#look for duplicate observations/cases
data.duplicated()

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


### Correlation
   * Pearson Correlation Coefficient—use when testing continuous variables that have a linear relationship.
   * Spearman Correlation Coefficient—use when testing variables with a non-linear relationship

```python
#generate correlation matrix
corr_mat = df.corr()
print(corr_mat)
```

              in-store       age     items    amount    region
    in-store  1.000000 -0.178180 -0.003897 -0.085573 -0.133171
    age      -0.178180  1.000000  0.000657 -0.282033 -0.235370
    items    -0.003897  0.000657  1.000000  0.000384 -0.001904
    amount   -0.085573 -0.282033  0.000384  1.000000  0.403486
    region   -0.133171 -0.235370 -0.001904  0.403486  1.000000


*Notes*  

* I don't think the correlation co-efficients for in-store and region are meaningful.
* There is a slight negative correlation between amount spent per transaction and age, suggesting that as age increases, the amount spent per transaction decreases. But I know from the scatter plot I generated earlier that the negative correlation between age and amount spent doesn't kick in until around age 62-65, when you see a sharp dropoff. Prior to that there doesn't appear to be any correlation between the two variables.

### Covariance

```python
#generate covariance matrix
cov_mat = df.cov()
print(cov_mat)
```

               in-store          age     items         amount      region
    in-store   0.250003    -1.400071 -0.004017     -30.860425   -0.075019
    age       -1.400071   246.966189  0.021270   -3196.782841   -4.167305
    items     -0.004017     0.021270  4.248751       0.570791   -0.004421
    amount   -30.860425 -3196.782841  0.570791  520221.252295  327.874873
    region    -0.075019    -4.167305 -0.004421     327.874873    1.269321

## Visualization Options

This is primarily a learning exercise in how to create core types of data vizualizations in Python.

### *Line Plots*

{{< figure src="/images/output_29_1.png" >}}

*Coding Notes*  

  - *fix*: title and axes labels appeared on a empty figure because ```plt.show()``` command was below the ```ax.plot()``` command.
  - You don't need to use ```plt.show()``` in Jupyter notebooks to generate matplotlib visualisations. But including it suppresses several lines of code that would otherwise appear above your graph/plot.

### *Scatter Plots*

```python
#create a var that is a random sample of 1000 observations from the dataset
data_sample100 = df.sample(1000)

#set x-axis to 'age' and y-axis to 'amount'
x = data_sample['age']
y = data_sample['amount']

#create a scatter plot
plt.scatter(x,y, marker='o')

#add a title
plt.title('Amount Spent Per Purchase by Age, Sample of 1000')
#add labels to x and y axes
plt.xlabel('Age')
plt.ylabel('Spending/Purchase')

plt.show()
```

{{< figure src="/images/output_43_0.png" >}}

*Coding Notes*  

  - SyntaxError fix: after adding title and label lines and running, got a syntax error indicating problem with ```plt.xlabel('Age')```. But problem was a missing close paren.
  - ```data.sample(n)``` takes a *random* sample of n observations from the dataset, so unless specified, the sample will include different obs. every time the code is run.
  - To fix which obs. are sampled (for reproducibility) use ```random_state```. See [Pandas documentation](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.sample.html) for more information.

Can't tell much from this scatter plot alone, except there is less variability in spending for customers 70+. (Turns out this pattern is a product of intentional data manipulation.)


### *Boxplots*

Box plots are especially good for:  
* finding outliers
* getting rough idea of how your data is distributed
* how tightly your data is grouped


```python
#create object to represent the amount feature
A = df['amount']

#generate boxplot of amount feature
plt.boxplot(A,0,'gD')

#add plot title and label y-axis
plt.title('Amount Spent Per Purchase')
plt.ylabel ('$ Spent')

plt.show()
```

{{< figure src="/images/output_47_0.png" >}}

*Coding Notes*   
  * The 0 parameter is determining whether the plot has notches. 0=No, 1=Yes.
  * The 'gD' parameter is determinint the color and shape of the smybol representing outliers.
    * To supress symbols altogether use ```plt.boxplot(A, 0, ' ')```
    * To generate a basic box plot just use ```plt.boxplot(A)```
    * To generate a horizontal box plot use ```plt.boxplot(A, 0, 'rs', 0)```
    * To generate boxplot *without* definining a new object use ```plt.boxplot(data['amount'])```


# Regional Analysis

In this section I'll use the information about the data I've gathered so far and do some additional analysis to answer Blackwell's questions about regional differences in customer spending.

## *Do customers in different regions spend more per transaction?*

```python
# get variable means for each region
 df.groupby('region').mean()
 df.groupby('region').median()
 df.groupby('region').std()
```
*Insights*
* Customers in the South spend the *least* per transaction (about $252). Customers in the East spend the most (about $1284).
* All transactions in the North occurred in-store.
* All transactions in the South occurred online.
* The avg. customer in the East is 15 years younger than the average customer in the South.
* The avg. number of items purchased in each transaction is 4.5 in all regions.
* Median spending per transaction is significantly lower than the mean in the North and West, suggesting there are high outlier transactions in those regions.
* The relatively small standard deviation in amount spent per transaction in the East suggests most purchases are in a small range around the mean of $250.

```python
#add new var that maps region #s to names, temp set all values to North
df['region_name']='North'

#change 2s to South
df.loc[df['region']==2,'region_name']='South'
#change 3s to East
df.loc[df['region']==3,'region_name']='East'
#change 4s to West
df.loc[df['region']==4,'region_name']='West'

df.head()
```


```python
#create violin plot of amount variable in each region
abr=sns.catplot(x = 'region_name', y = 'amount', data = df, kind = 'violin',
            aspect = 1.5)
abr.set_xticklabels( rotation = 45) #argument({'tic 1', 'tic 2',...}, rotation=45)
abr.fig.suptitle('Regional Spending')
abr.fig.subplots_adjust(top=0.9)
abr.set_axis_labels('Region', 'Spending per Transaction')
```

{{< figure src="/images/output_60_0.png" >}}


*Summary*  

Short answer: *YES*. A quick look at the average amount spent in each region reveals that the Southern region spends the least (~$252.00$). Although it looks like the Eastern region spends the most per transaction (~$1284), further investigation reveals that number is heavily skewed by several outlier transactions. When the outliers are left aside, we can see that typical spendintg in the Eastern and Northern regions are very similar. We can also see that customers in the Western region typically spend the most. One final thing to note is that the amount spent per transaction in the Southern region spans a noteably smaller range than the other three areas, and is concentrated at the low end.

## *Which regions spend the most/least?*

We saw earlier that customers in the **South** typically spend to *least* per transaction (~ $250.00) and customers in the **West** typically spend the *most* (~ $1230.00).

But perhaps the question is *which region(s) bring in the most/least revenue?*

```python
#caluculate regional sums
region_sum=df.groupby(['region']).sum()
print(region_sum)
```
               items      amount
    region                                         
    1          72151     1.191762e+07
    2          90229     5.040442e+06
    3          80892     1.652345e+07
    4          117044    3.336699e+07


```python
# create pie chart showing % of total revenue for each region
labels = 'North', 'South', 'East', 'West'
sizes = [17.8, 7.5, 24.7, 49.9]
explode = (0, 0.1, 0, 0.1)  # "explode" 2nd and 3rd slices
#explode highlights regions of interest

fig1, ax1 = plt.subplots()
ax1.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%',
        shadow=True, startangle=90)
ax1.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle.

#add plot title
fig1.suptitle('Revenue by Region', fontsize=16)
```

{{< figure src="/images/output_67_0.png" >}}

The pie chart nicely illustrates that the West brings in the most revenue and the South brings in the least.


## *Are there regional differences in online vs. in-store spending?*

```python
df.groupby('in-store').describe()['amount']
```

```python
#revenue online vs. in-store by region
oir = sns.catplot(data=df, kind='box',
                  x='region_name', y='amount', hue='in-store', height=6)
oir.despine(left=True)
#oir.title("Online vs. In-Store Spending by Region")<--doesn't work
oir.set_xticklabels( rotation=45)
oir.set_axis_labels('Region', 'Spending')
oir.legend.set_title('')
#NA--oir.subplots_adjust(top=0.9)
oir.fig.suptitle("Online vs. In-Store Spending",
                  fontsize=16, fontdict={"weight": "bold"}, y = 1.05)
```
{{< figure src="/images/output_72_0.png" >}}


***Observations***

* As you can see in the above plot, the Southern region has **only online** sales, and the Northern region has **no online** sales. Assuming the data is sound, the complete lack of online sales in the South presents a big opportunity for the company.
* Wwe can also see in the above graph that online sales account for all the high dollar transactions. Almost all the sales above $2,100.00$ are online transactions. This suggests potential value in developing a strategy aimed at growing high-dollar purchases in-store.


# Age Analysis

## *Create Categorical Age Variables*

*Discretization* is the process of taking a continuous variable, like age or amount, and turning it into a categorical variable by putting all values into discreet bins (e.g. over 65 and under 65). [This](https://s3.amazonaws.com/gbstool/courses/1094/docs/An%20Introduction%20to%20Discretization%20Techniques%20for%20Data%20Scientists.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201023T205053Z&X-Amz-SignedHeaders=host&X-Amz-Expires=36900&X-Amz-Credential=AKIAJBIZLMJQ2O6DKIAA%2F20201023%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=4e9e02e0403840c946cd9a5a9fe5270c8146878715dcdb9853fa908ee0821602) article contains the pkgs and commands needed to do various kinds of discretization.

***The Value of Discretization***

* Results can be *more meaningful* and *easier to understand* when the data has been binned into categories that have been thoughtfully constructed to fit the question(s) we are trying to answer. (e.g.  binning age into generations--baby boomers, millenials, gen z, etc.--to uncover generational differences in consumer behavior.     - Certain statistical methods/models are only appropriate when working with categorical variables.
* Binning data can reduce the impact of noise in the data.

***Approaches to discretization***

- *equal width* — separate all values into 'N' number of bins, each having the same width (i.e. span of values)
- *equal frequency* — separate the values in 'N' number of bins, each containing the same number of observations.
- *K-means* — use k-means clustering to to group data.

### Over 65 vs. Under 65

```python
#define new discritized age variable, two bins: under 65 and 65+
young = pd.cut(df.age, bins=[18,63,84], labels=['Under 65', '65+'])

#add new young variable to df
df['young']=young

df.head()
```

```python
ca = sns.countplot(data=df,
                  x='region_name', y=None, hue='young')
ca.set(xlabel='Region', ylabel='Count')
ca.legend().set_title('')
```

{{< figure src="/images/output_78_0.png" >}}


* This bar chart tells us that (1) all customers in the Western region are under 65, and the Southern region has the most number of customers over 65. This suggests marketing target at age groups may be more successful in those two regions.
* That being said, the relatively small size of Blackwell's customer base that is 65+ makes it an area for big potential gains.

### Generational Analysis

```python
#define new generation variable
gen = pd.cut(df.age,
             bins=[18,23,39,55,74,85],
             labels =['Gen Z (8-23)',
                      'Millenial (24-39)',
                      'Gen X (40-55)',
                      'Boomer (56-74)',
                      'Silent Gen (75-95)'])

#add new gen variable to df
df['gen']=gen
df.head()
```


```python
#create generational spending plot
gen_plot = sns.catplot(data=df,
                  x='region_name', y=None, hue='gen', kind='count',
                      height=4, aspect=1)
gen_plot.set(xlabel='Region', ylabel='Count',
            title='Regional Transactions by Age Group')
gen_plot.set_xticklabels(rotation=45)

gen_plot.legend.set_title('')

gen_plot.savefig('pic_gen_plot.png')
```

*Coding Notes*
- None of the following worked when creating the generation bar chart.
  - gen_plot_leg=gen_plot.legend()
  - gen_plot.fig.legend(title='', labels=[])
  - legend = gen_plot.legend()
  - legend.texts[0].set_text('')
  - fig=gen_plot.get_figure()

{{< figure src="/images/output_80_0.png" >}}

***Observations***

* All customers of the Silent Generation are in the South.
* Generation Z (ages 18-24) is the smallest cohort of customers in 3 of 4 regions.
* There are no Gen Z customers in the South.

## *Do the generations differ in their purchasing?*

```python
#violin plot of spending by customers under 65 and 65+ in each region
car = sns.catplot(data=df, kind='violin',
                  x='region_name', y='amount', hue='young',
                  palette='rocket', legend_out=True, ci = None)
car.despine(left=True)
car.set_xticklabels( rotation=45)
car.set_axis_labels('', 'Spending per Transaction')
car.fig.suptitle("Customer Spending by Age: Under 65 vs 65+",
                  fontsize=16, fontdict={"weight": "bold"})
#car.fig.legend(title="", labels=['Online', 'In-store'])
car.legend.set_title('')
```
{{< figure src="/images/output_87_0.png" >}}


```python
gen_spending_boxen = sns.catplot(data=df,
x='region_name', y='amount', hue='gen', kind='boxen', height=6, aspect=2)

gen_spending_boxen.set(xlabel='Region', ylabel='Amount Spent',
                            title='Spending by Age Group')
```

{{< figure src="/images/output_91_1.png" >}}

```python
#calculate mean of age, items, & amount for each gen within each region
gen_mean=df.groupby(['region_name', 'gen'])['age', 'items', 'amount'].mean()
print(gen_mean)

```

                                          age     items       amount
    region_name gen                                                 
    East        Gen Z (8-23)        20.954082  4.503827  1584.896097
                Millenial (24-39)   31.656688  4.467841   958.103526
                Gen X (40-55)       47.476020  4.508320   927.122623
                Boomer (56-74)      63.478210  4.496804   751.545588
                Silent Gen (75-95)        NaN       NaN          NaN
    North       Gen Z (8-23)        21.157093  4.478312  1018.099817
                Millenial (24-39)   31.661903  4.515131   804.479170
                Gen X (40-55)       47.337918  4.499066   771.135680
                Boomer (56-74)      64.708195  4.529785   520.992073
                Silent Gen (75-95)        NaN       NaN          NaN
    South       Gen Z (8-23)              NaN       NaN          NaN
                Millenial (24-39)   33.685424  4.500248   250.351972
                Gen X (40-55)       47.550467  4.505747   252.129675
                Boomer (56-74)      64.981328  4.522211   252.277828
                Silent Gen (75-95)  79.886164  4.520128   253.608984
    West        Gen Z (8-23)        21.098558  4.564197  1266.141474
                Millenial (24-39)   31.466787  4.485318  1264.815006
                Gen X (40-55)       47.407644  4.504028  1251.748461
                Boomer (56-74)      59.264597  4.491659  1545.107915
                Silent Gen (75-95)        NaN       NaN          NaN







# Modeling

* In this project the goal is just to train and test a decision tree classifier model to "predict" (or correctly classify) the region each transaction is from using the other four variables.
* Aiming for a model that correctly predicts at least 75% of transactions (> 75% accuracy).

### Import Libraries

```python
#DS Basics
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

#SKLearn Stuff
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import cross_val_score
from sklearn import metrics

#helpers
%matplotlib inline
```

## Data Preparation

Because modeling is not central to the analysis in this project, I'm not doing any additional data prep before modeling. In a more typical scenario in addition to the  wrangling/processing done before exploration, there would likely be additional preparation to do before modeling.  **Data Normalization/Standardization**


# Decision Tree Classifier

```python
#specify the independent variables (features/attributes) for the model
X = data.iloc[:, 0:4]
print('Summary of feature sample')
X.head()

#specify the dependent variable (target) for the model
y = data['region']
```


```python
#split df into a training set and testing set
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = .30,
                                                   random_state = 123)
```

*Coding Notes*  
* ```test_size = .30``` specifies 70% of the data for training and 30% for testing.
* ```random_state = 123``` ensures the same observations are selected for the training and testing sets every time the code is run (for reproducibility). 123 is not important (any other number would work just as well).


```python
#instatiate the decision tree algorithm
algo = DecisionTreeClassifier(max_depth=3)

#train the model on the training data set
model = algo.fit(X_train, y_train)
```


```python
#test the model on testing data to see how good the predictions are
preds = model.predict(X_test)

#create classification report on how well model did on the testing data
print(classification_report(y_test, preds))

#create confusion matrix showing model predictions for each class
print(metrics.confusion_matrix(y_test, preds))
```

*Coding Notes*
* ```preds = model.predict(X_test)```is where we are giving our trained model some new sets of X values (```X_test```) and asking it predict what region each is from.
* In ```print(classification_report(y_test, preds))``` we are comparing the prediction the model made for each observation in X_test, comparing the predicted region to the actual region for each observation (i.e. the values in ```y_test```), and generating a classification report
* **Understanding the Classification Report:**
  - *precison* = the true positives/true positives + false positives
  - *recall* = true positives/true positives + false negatives' f1-score = a balance between precision and recall
  - *support* = the number of observations in X_test that were in each region.
  - *accuracy* = number of correct predictions/ total number of predictions
    - accuracy is only meaningful if test set is roughly balanced between the classes.
    Suppose 98% of the observations were in region 1, then the model could get 98% accuracy just by predicting that all observations are from region 1.
* The confusion matrix is telling us for the observations actually from each region (rows), how many the model predicted in each region (e.g. for the cases actually in region 1, the model predicted 3,266 were in 1, 0 were in 2, 494 were in 3, and 1078 were in 4)

```python
#print a text-only representation of the decison tree model
text_representation = tree.export_text(model)
print(text_representation)
```

### Tree Plot

```python
#import pkgs
from sklearn.tree import plot_tree
#graphviz creates better visualizations of decision trees
import graphviz
# pkgs also needed to plot tree using graphviz
from sklearn import tree
import pydotplus
```


```python
#plot the decision tree
fig = plt.figure(figsize=(30,15))
tree = plot_tree(model, max_depth=3, feature_names=X.columns,
                 class_names =['0','1', '2','3'],
                 filled = True, fontsize=14)
```


```python
#change y values into strings because graphviz requires dv to be strings
y_str=y.astype(str)

#apply the graphviz tree plotting function to my model and data
dot_data = tree.export_graphviz(model, max_depth=3,  
                                feature_names=X.columns,  
                                class_names=y_str,
                                filled=True)
# Draw graph
pydot_graph = pydotplus.graph_from_dot_data(dot_data)
pydot_graph.set_size('"8,8!"')
#pydot_graph.write_png('resized_tree.png')

graph = graphviz.Source(dot_data, format="png")
print(graph)

#output a .png file of graphviz tree & save to directory with jup nb
graph.render("decision_tree_graphivz")
```

{{< figure src="/images/c1_decision_tree_plot.png" >}}


*Coding Notes*

* The tree from ```plot_tree()``` was too crowded, so I used the graphviz pkg instead.
* Each node of the tree tells us the *feature-value* the model selected for splitting the data during each iteration (e.g. first split at amount <= to 499.95).
* Each node/leaf also tells us the *gini co-efficient* for that node. Gini is a measure of impurity—lower values are better.
  * If a partition has all and only observations within a single region, the gini will be 0.
* samples = tells us how many observations from the data set are in that partition, so as we move down a branch from top to bottom the sample = number should be getting smaller and smaller as the model partitions the data into smaller and smaller sections.
* the value = line tells us how many observations are in each partition at that node/leaf. (e.g. in the age<= 55.5 node we find out that there are 4091 obs. in region 1, 0 in region 2, 3676 in region 3, and 2047 in region 4).
* class = is telling us the what region the model is predicting (at that point in the tree) for all the observations in that partition...i'm not sure this make sense, review later.  
* decision trees can also be plotted using the dtreeviz pkg—these are the best looking. For more info see [this github post](https://github.com/parrt/dtreeviz).

### Parameter Experimentation

#### gini → entropy


```python
#changing default gini criterion to entropy
algo_ent = DecisionTreeClassifier(criterion='entropy', max_depth=3)

#train the decision tree algorithm on the training data
model_ent = algo_ent.fit(X_train, y_train)
```


```python
#run model on X_test and predict values in y_test
preds_ent = model_ent.predict(X_test)
```


```python
#check model's predicted values against actual values (ground truth) in y_test
#and report the results
print(classification_report(y_test, preds_ent))
```


---

### Tree Depth
This parameter controls/limits how wide and deep the decision tree can get. All decision trees begin with a single origin node which branches into 2 or more child nodes. Those of those child nodes then branches to further child nodes, which can then branch to a further layer of child nodes, and so on on.

However, decision tree models are pone to overfitting. One way to reduce the risk of model overfitting is to limit how  deep the tree can get--the number of layers.

For this project, if the model is allowed to terminate on its own, the resulting tree has a depth of more than 50. In my initial model, I limited the tree depth (max_depth) to 3.

#### max_depth = 4


```python
#change max depth from 3 to 4
algo_ent4 = DecisionTreeClassifier(criterion='entropy', max_depth=4)

#train the decision tree algorithm on the training data
model_ent4 = algo_ent4.fit(X_train, y_train)
```


```python
#run model on X_test and predict values in y_test
preds_ent4 = model_ent4.predict(X_test)
```


```python
#check model's predicted values against actual values (ground truth) in y_test
#and report the results
print(classification_report(y_test, preds_ent4))
```

*Notes*  
* these scores are identical to scores for max depth 3 model. not sure if that's b/c i did something wrong or the model doesn't gain any predictive power after depth 3
    * ben said it's b/c the model doesn't improve beyond max depth 3

---

#### max depth = 1


```python
#change max depth from 4 to 1
algo_ent1 = DecisionTreeClassifier(criterion='entropy', max_depth=1)

#train the decision tree algorithm on the training data
model_ent1 = algo_ent1.fit(X_train, y_train)
```


```python
#run model on X_test and predict values in y_test
preds_ent1 = model_ent1.predict(X_test)
```


```python
#check model's predicted values against actual values (ground truth)
#in y_test and report the results
print(classification_report(y_test, preds_ent1))
```

*Notes*
* i think this warning appeared because there were no observations in regions 3 & 4 after the split represented in the root node of the tree. but with max depth of the tree set to 1, there are no more splits in the data (***verify***)  
---

## Model Performance

### Cross Validation

how does cross validation decrease the chance that a ML model will be overfit?
* basically because you are testing and training the model on several smaller datasets (the folds), and then averaging the results of each of those to come up with the final model, it's more likely the overfitting will be smoothed out in the average. say the first model is overfit by .6, ***I DON'T THINK THIS IS CORRECT. I'VE DONE A LOT OF RESEARCH AND CANNOT FIND A SATISFACTORY ANSWER***

#### 3-fold CV


```python
#instatiate cross validation model with decision tree algorithm
cv_model = DecisionTreeClassifier(random_state=123)

#apply model with 3 fold to dataset
scores = cross_val_score(cv_model, X, y, cv = 3)
print(scores)
```


```python
#calculate mean of accuracy scores from each fold
avg_score = np.mean(scores)
print(avg_score)
```

#### 4-fold CV


```python
#apply 4-fold-model to data
scores = cross_val_score(cv_model, X, y, cv =4)
print(scores)
```


```python
#calculate mean of accuracy scores from each fold
avg_score = np.mean(scores)
print(avg_score)
```

#### 8-fold CV


```python
#apply 8-fold model to data
scores = cross_val_score(cv_model, X, y, cv = 8)
print(scores)
```


```python
#calculate mean of accuracy scores from each fold
avg_score = np.mean(scores)
print(avg_score)
```

#### 20-fold CV


```python
#apply 20 fold model to data
scores = cross_val_score(cv_model, X, y, cv = 20)
print(scores)
```


```python
#calculate mean of accuracy scores from each fold
avg_score = np.mean(scores)
print(avg_score)
```

*Notes*  
* i'm not sure how to choose the optimal number of folds.
* i expected the accuracy of the model to improve at least slightly (up to a point) the more folds you used in the cv model because i was mistakenly thinking that in a cv model, the model was adjusted after being trained on each fold, so that after training the final model was basically the average of the models trained on each fold.
* however, after talking to david i now know that is not how it works.
* when you specify the type of algorithm (e.g. decision tree, random forest), set the various parameters, and specify your feature and target variables—the model is set. the model is not changed/refined in the training on each fold. the same model is used on every fold
* but then how does cross validation cut the risk of overfitting? basically, the accuracy scores on each fold should be roughly the same. if one accuracy score is very different than the others, you know that something is going on with that part of the data and you need to investigate to figure it out. (***verify: i'm not sure about this***)



### Feature Importance


```python
# use pandas and random forest model to see which features within the df are most important
feature_imp = pd.series(model.feature_importances_, index=data.feature.names)
    .sort_values(ascending=False)
```

[Link to Project GitHub Repository](https://github.com/kpiatti/customer-purchasing-patterns)
