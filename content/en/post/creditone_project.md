---
date: false
description: ""
featured_image: "/images/credit_feature.webp"
title: "Credit Risk Assessment with Python"
show_reading_time: true
show_date: false
---
The prescribed objective for this project was to outline and then implement a data science process—like CRISP-DM—resulting in a model that could reliably predict whether or not a customer would default on their loan(s) from their demographic information, as well as six months worth of data on their billing and payment history.  

<!--
```python
#import required packages
from sqlalchemy import create_engine
import pymysql
import pandas as pd
import numpy as np
```
-->


### *Obtain Data*

I obtained the raw data for this project from a MySQL database by establishing a connection to the server, executing a simple SELECT * query, and saving the result/output as a .csv file. I use [this](/folder/creditone_data_info1.pdf) data supplement to understand what the column/variable names represent.

<!--
(even though it's possible to perform some data processing tasks within the database using SQL, if feasible, my preference is to work with data usingR or Python all the processing inside my IDE--in this case Jupyter Lab)
* *Original Source* — data was first used in [this](https://bradzzz.gitbooks.io/ga-seattle-dsi/content/dsi/dsi_05_classification_databases/2.1-lesson/assets/datasets/DefaultCreditCardClients_yeh_2009.pdf)  academic paper.
* *

```python
#connect python to MySQL server (database?)
db_connection_str = 'mysql+pymysql://deepanalytics:Sqltask1234!@34.73.222.197/deepanalytics'

#create engine for SQLalchemy to interface with the database API (see notes below)
db_connection = create_engine(db_connection_str)

#query the credit one data and extract it into a pd df
sql_query = pd.read_sql('SELECT * FROM credit', con=db_connection)

#create pandas df from SQL query
df = pd.DataFrame(sql_query)

#export df as .csv file
df.to_csv ('raw_creditone_data.csv', index = False)
```

> *Coding Note*
> * Unsure why (or if) the ```create_engine()``` command is needed. (See my SQL collection in Zotero).
-->




&nbsp;
## Data Wrangling
To get the data ready for analysis, I did the following:

- cleaned variable names to make them more meaningful and transparent
- removed 72 rows of duplicate data (including 2 extra header rows)
- dropped row containing the one and only missing value
- changed variable data types from *object* to *int* (integer) where appropriate
- reordered columns so all related variables are grouped together

The cleaned data contains 30,000 observations & 25 variables.

<!--
```python
#import exported csv file with raw data
df = pd.read_csv('raw_credit_one_data.csv')

#verify imported data looks as expected
df.head()

<bound method DataFrame.to_markdown of   MyUnknownColumn         X1      X2          X3        X4   X5     X6     X7  \
0              ID  LIMIT_BAL     SEX   EDUCATION  MARRIAGE  AGE  PAY_0  PAY_2   
1               1      20000  female  university         1   24      2      2   
2               2     120000  female  university         2   26     -1      2   
3               3      90000  female  university         2   34      0      0   
4               4      50000  female  university         1   37      0      0   

      X8     X9  ...        X15        X16        X17       X18       X19  \
0  PAY_3  PAY_4  ...  BILL_AMT4  BILL_AMT5  BILL_AMT6  PAY_AMT1  PAY_AMT2   
1     -1     -1  ...          0          0          0         0       689   
2      0      0  ...       3272       3455       3261         0      1000   
3      0      0  ...      14331      14948      15549      1518      1500   
4      0      0  ...      28314      28959      29547      2000      2019   

        X20       X21       X22       X23                           Y  
0  PAY_AMT3  PAY_AMT4  PAY_AMT5  PAY_AMT6  default payment next month  
1         0         0         0         0                     default  
2      1000      1000         0      2000                     default  
3      1000      1000      1000      5000                 not default  
4      1200      1100      1069      1000                 not default  

[5 rows x 25 columns]>


# number of rows and columns in df as benchmark before making any changes
df.shape

    (30204, 25)
```
The raw data has 30,204 rows and 25 columns.


&nbsp;

### *Change Column Headings*
In the raw data, the variables are named X1-X23 & Y. More meaningful names appear in row 0. I need to get rid of the existing column headings and replace them with the values in row 0.

```python
#replace column headings to values in index (row) 0
df.columns = df.iloc[0]

#verify change was successful
df.head().to_markdown
---
    <bound method DataFrame.to_markdown of 0  ID  LIMIT_BAL     SEX   EDUCATION  MARRIAGE  AGE  PAY_0  PAY_2  PAY_3  \
    0  ID  LIMIT_BAL     SEX   EDUCATION  MARRIAGE  AGE  PAY_0  PAY_2  PAY_3   
    1   1      20000  female  university         1   24      2      2     -1   
    2   2     120000  female  university         2   26     -1      2      0   
    3   3      90000  female  university         2   34      0      0      0   
    4   4      50000  female  university         1   37      0      0      0   

    0  PAY_4  ...  BILL_AMT4  BILL_AMT5  BILL_AMT6  PAY_AMT1  PAY_AMT2  PAY_AMT3  \
    0  PAY_4  ...  BILL_AMT4  BILL_AMT5  BILL_AMT6  PAY_AMT1  PAY_AMT2  PAY_AMT3   
    1     -1  ...          0          0          0         0       689         0   
    2      0  ...       3272       3455       3261         0      1000      1000   
    3      0  ...      14331      14948      15549      1518      1500      1000   
    4      0  ...      28314      28959      29547      2000      2019      1200   

    0  PAY_AMT4  PAY_AMT5  PAY_AMT6  default payment next month  
    0  PAY_AMT4  PAY_AMT5  PAY_AMT6  default payment next month  
    1         0         0         0                     default  
    2      1000         0      2000                     default  
    3      1000      1000      5000                 not default  
    4      1100      1069      1000                 not default  


    [5 rows x 25 columns]>

#drop row 0
df.drop(df.index[0], inplace = True)    
```

&nbsp;

### *Drop Duplicates*

```python
# locate duplicate rows
duplicate_rows = df[df.duplicated()]

print(duplicate_rows)


    0     ID LIMIT_BAL     SEX        EDUCATION MARRIAGE AGE PAY_0 PAY_2 PAY_3  \
    204    1     20000  female       university        1  24     2     2    -1   
    205    2    120000  female       university        2  26    -1     2     0   
    206    3     90000  female       university        2  34     0     0     0   
    207    4     50000  female       university        1  37     0     0     0   
    208    5     50000    male       university        1  57    -1     0    -1   
    ..   ...       ...     ...              ...      ...  ..   ...   ...   ...   
    400  197    150000  female       university        1  34    -2    -2    -2   
    401  198     20000  female  graduate school        2  22     0     0     0   
    402  199    500000  female  graduate school        1  34    -2    -2    -2   
    403  200     30000  female      high school        2  22     1     2     2   
    404  201    180000  female  graduate school        1  38    -2    -2    -2   

    0   PAY_4  ... BILL_AMT4 BILL_AMT5 BILL_AMT6 PAY_AMT1 PAY_AMT2 PAY_AMT3  \
    204    -1  ...         0         0         0        0      689        0   
    205     0  ...      3272      3455      3261        0     1000     1000   
    206     0  ...     14331     14948     15549     1518     1500     1000   
    207     0  ...     28314     28959     29547     2000     2019     1200   
    208     0  ...     20940     19146     19131     2000    36681    10000   
    ..    ...  ...       ...       ...       ...      ...      ...      ...   
    400    -2  ...       116         0      1500        0        0      116   
    401     0  ...      8332     18868     19247     1500     1032      541   
    402    -1  ...      1251      1206      1151      138     2299     1251   
    403     0  ...     29836      1630         0     1000       85     1714   
    404    -2  ...         0         0         0        0        0        0   

    0   PAY_AMT4 PAY_AMT5 PAY_AMT6 default payment next month  
    204        0        0        0                    default  
    205     1000        0     2000                    default  
    206     1000     1000     5000                not default  
    207     1100     1069     1000                not default  
    208     9000      689      679                not default  
    ..       ...      ...      ...                        ...  
    400        0     1500        0                not default  
    401    20000      693     1000                not default  
    402     1206     1151    15816                not default  
    403      104        0        0                    default  
    404        0        0        0                not default  

    [201 rows x 25 columns]

#drop 201 duplicate rows
df.drop_duplicates(inplace = True)  #inplace=True is to make change to df permanent

#check df length to verify rows were dropped
len(df)

    30002
```

&nbsp;

### *Remove Missing Values*

```python
#check for missing values
print(df.isnull().sum())

    0
    ID                            1
    LIMIT_BAL                     0
    SEX                           0
    EDUCATION                     0
    MARRIAGE                      0
    AGE                           0
    PAY_0                         0
    PAY_2                         0
    PAY_3                         0
    PAY_4                         0
    PAY_5                         0
    PAY_6                         0
    BILL_AMT1                     0
    BILL_AMT2                     0
    BILL_AMT3                     0
    BILL_AMT4                     0
    BILL_AMT5                     0
    BILL_AMT6                     0
    PAY_AMT1                      0
    PAY_AMT2                      0
    PAY_AMT3                      0
    PAY_AMT4                      0
    PAY_AMT5                      0
    PAY_AMT6                      0
    default payment next month    0
    dtype: int64
```
It appears there's only 1 missing value in the df.

```python
#drop the row with missing ID
df.dropna(inplace=True)

#check df length to verify row was dropped
len(df)

    30001
```

&nbsp;

### *Remove Invalid Data*

```python
#locate rows 201-206 (not inclusive)
df.loc[201:206]

#drop row 203 containing original column headings
df.drop(203, inplace=True)

#check df length to verify row was dropped
len(df)

    30000
```

>*Coding Notes*
>* Dropping rows doesn't stick unless you add ```inplace = True``` parameter to command. Read [this](https://stackabuse.com/python-with-pandas-dataframe-tutorial-with-examples/) to understand why parameter is needed.
>* After dropping index, that index number is dropped from the df (e.g. if you drop index 0, when you use df.head() the first entry will be index 1). In other words, the data from below does not move up and fill the dropped indices as it would in an Excel file..

&nbsp;

### *Change Variable Data Types*

```python
#get data types for each variable
df.dtypes

    0
    ID                            object
    LIMIT_BAL                     object
    SEX                           object
    EDUCATION                     object
    MARRIAGE                      object
    AGE                           object
    PAY_0                         object
    PAY_2                         object
    PAY_3                         object
    PAY_4                         object
    PAY_5                         object
    PAY_6                         object
    BILL_AMT1                     object
    BILL_AMT2                     object
    BILL_AMT3                     object
    BILL_AMT4                     object
    BILL_AMT5                     object
    BILL_AMT6                     object
    PAY_AMT1                      object
    PAY_AMT2                      object
    PAY_AMT3                      object
    PAY_AMT4                      object
    PAY_AMT5                      object
    PAY_AMT6                      object
    default payment next month    object
    dtype: object

#change data type of LIMIT_BAL var from object to int
df["LIMIT_BAL"] = df['LIMIT_BAL'].astype('int')

#change data type of AGE var from object to int
df["AGE"] = df['AGE'].astype('int')

#change data type of BILL_AMT1 var from object to int
df["BILL_AMT1"] = df['BILL_AMT1'].astype('int')

#change data type of BILL_AMT2 var from object to int
df["BILL_AMT2"] = df['BILL_AMT2'].astype('int')

#change data type of BILL_AMT3 var from object to int
df["BILL_AMT3"] = df['BILL_AMT3'].astype('int')

#change data type of BILL_AMT4 var from object to int
df["BILL_AMT4"] = df['BILL_AMT4'].astype('int')

#change data type of BILL_AMT5 var from object to int
df["BILL_AMT5"] = df['BILL_AMT5'].astype('int')

#change data type of BILL_AMT6 var from object to int
df["BILL_AMT6"] = df['BILL_AMT6'].astype('int')

#change data type of PAY_AMT1 var from object to int
df["PAY_AMT1"] = df['PAY_AMT1'].astype('int')

#change data type of PAY_AMT2 var from object to int
df["PAY_AMT2"] = df['PAY_AMT2'].astype('int')

#change data type of PAY_AMT3 var from object to int
df["PAY_AMT3"] = df['PAY_AMT3'].astype('int')

#change data type of PAY_AMT4 var from object to int
df["PAY_AMT4"] = df['PAY_AMT4'].astype('int')

#change data type of PAY_AMT5 var from object to int
df["PAY_AMT5"] = df['PAY_AMT5'].astype('int')

#change data type of PAY_AMT6 var from object to int
df["PAY_AMT6"] = df['PAY_AMT6'].astype('int')

#rename default payment next month column
df.rename(columns={"default payment next month": "DEFAULT"}, inplace = True)

#get var dtypes to verify changes
df.dtypes

    0
    ID           object
    LIMIT_BAL     int32
    SEX          object
    EDUCATION    object
    MARRIAGE     object
    AGE           int32
    PAY_0        object
    PAY_2        object
    PAY_3        object
    PAY_4        object
    PAY_5        object
    PAY_6        object
    BILL_AMT1     int32
    BILL_AMT2     int32
    BILL_AMT3     int32
    BILL_AMT4     int32
    BILL_AMT5     int32
    BILL_AMT6     int32
    PAY_AMT1      int32
    PAY_AMT2      int32
    PAY_AMT3      int32
    PAY_AMT4      int32
    PAY_AMT5      int32
    PAY_AMT6      int32
    DEFAULT      object
    dtype: object
```


### Categorical Variables

```python
#get values for SEX var
df.SEX.value_counts()
---
    female    18112
    male      11888
    Name: SEX, dtype: int64

#get counts for education categories
df.EDUCATION.value_counts()
---
    university         14030
    graduate school    10585
    high school         4917
    other                468
    Name: EDUCATION, dtype: int64

#get counts for marriage categories
df.MARRIAGE.value_counts()
---
    2    15964
    1    13659
    3      323
    0       54
    Name: MARRIAGE, dtype: int64

# get counts for the target var, i.e. DEFAULT
df.DEFAULT.value_counts()
---
    not default    23364
    default         6636
    Name: DEFAULT, dtype: int64
```

&nbsp;

### *Rename Features*

```python
#rename PAY features to convey month
df.rename(columns = {'PAY_0': 'pay_s', 'PAY_2':'pay_ag', 'PAY_3':'pay_jy','PAY_4':'pay_ju', 'PAY_5':'pay_m', 'PAY_6':'pay_ap'}, inplace = True)

#rename BILL_AMT features to convey month
df.rename(columns = {'BILL_AMT1': 'bill_s', 'BILL_AMT2':'bill_ag', 'BILL_AMT3':'bill_jy','BILL_AMT4':'bill_ju', 'BILL_AMT5':'bill_m', 'BILL_AMT6':'bill_ap'}, inplace = True)

#rename PAY_AMT features to convey month
df.rename(columns = {'PAY_AMT1': 'pmt_s', 'PAY_AMT2':'pmt_ag', 'PAY_AMT3':'pmt_jy','PAY_AMT4':'pmt_ju', 'PAY_AMT5':'pmt_m', 'PAY_AMT6':'pmt_ap'}, inplace = True)
```
-->

<!--
### *Reorder Features*

```python
#define function to move/re-order columns (code lifted from towarddatascience)
def movecol(df, cols_to_move=[], ref_col='', place='After'):

    cols = df.columns.tolist()
    if place == 'After':
        seg1 = cols[:list(cols).index(ref_col) + 1]
        seg2 = cols_to_move
    if place == 'Before':
        seg1 = cols[:list(cols).index(ref_col)]
        seg2 = cols_to_move + [ref_col]

    seg1 = [i for i in seg1 if i not in seg2]
    seg3 = [i for i in cols if i not in seg1 + seg2]

    return(df[seg1 + seg2 + seg3])

#move LIMIT_BAL column next to other account/credit variables
df = movecol(df,
             cols_to_move=['LIMIT_BAL'],
             ref_col='AGE',
             place='After')

#re-order pay columns to follow calendar
df = movecol(df,
             cols_to_move=['pay_ap', 'pay_m', 'pay_ju', 'pay_jy', 'pay_ag', 'pay_s'],
             ref_col='LIMIT_BAL',
             place='After')

#re-order pmt columns  to follow calendar
df = movecol(df,
             cols_to_move=['pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s'],
             ref_col='LIMIT_BAL',
             place='After')

#re-order bill columns  to follow calendar
df = movecol(df,
             cols_to_move=['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s'],
             ref_col='LIMIT_BAL',
             place='After')
```


### *Export Cleaned Data*

```python
#export cleaned transformed data as .csv file
df.to_csv('cleaned_creditone_data.csv', index = False, header=True)
```
-->


&nbsp;
## Exploratory Data Analysis (EDA)

```python
#import packages
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
plt.style.use('fivethirtyeight')
import numpy as np
from pandas_profiling import ProfileReport

#import cleaned data
df = pd.read_csv('cleaned_transformed_credit_one_data.csv')
```

*The observations that follow are primarily based on my analysis of the HTML pandas profile report I generated from the data.*

<!--
&nbsp;
### *Univarite Analysis*

```python
#create in-line minimal pandas profile report
ProfileReport(df, minimal=True)

#profile = ProfileReport(df, title='Clean CreditOne Pandas Profiling Report', explorative = True)
```

>*Coding Notes*
>* The full pandas profile report took a *very long time to generate* and was *too big* to load without crashing my browser.
>* Two options to generate profile reports more quickly:
    * Generate a minimal report (which does not include correlations between vars) by adding a 'minimal=True' parameter to ProfileReport command.
    * Generate a report for only a *sample* of df by substituting ```df.sample(n=10000)``` for df in the command.
>* To output the report as a separate html file use:  ```prof = ProfileReport(df)
prof.to_file(output_file='output.html')```
-->



&nbsp;
#### Demographic Variables

*Age* & *Sex*
{{< figure src="/images/creditone_age_sex_dist_plots.png" >}}

* Age range 21 - 79, Mean 35, Median 34
* There are slightly more female customers (60.4% to 39.6%)
<!--* Consider binnning: young, middle age, 65+-->


&nbsp;
*Education* & *Marriage*
{{< figure src="/images/creditone_edu_mar_count_plots.png" >}}

* HS 16.4%, University 46.8%, Graduate School 35.3%, Other 1.6%
* A disproportionate number of customers have university & graduate degrees (compared to US population)
* Married (1) 45.5%, Single (2) 53.6%, Divorced (3) 1.1%, Other 0.2%

<!--* May consider combining university and graduate school categories later.-->


&nbsp;
#### Account Variables

*Credit Limit*: the $ amount of credit each customer currently has
{{< figure src="/images/creditone_crlim_dist.png" >}}

* Mean: 167,484; Mode: 50k (11.2%)
* Distribution significantly skewed toward low limits
* Wide range: 10k - 1M
* Low variability: only 81 distinct values


*Default*: dummy variable, indicate whether the customer has or has not defaulted on their loans (we don't know the time frame determining this value, e.g. lifetime of account, at least once during last 7 yrs).
{{< figure src="/images/creditone_dflt_dist.png" >}}

* Not default 77.9%, default 22.1%


*Repayment Variables*


* -2 = no use, -1 = paid in full, 0 = use of revolving credit, 1= 1 month payment delay, 2= 2 month delay, ..., 9= 9+ month delay (at what point are they coded as defaulted?)
* All six variables have roughly the same, significantly skewed distributions (50%+ of values are 0, few values >= 1)


&nbsp;
#### Billing Variables
{{< figure src="/images/creditone_bill_dist.png" >}}

- The distributions for all six variables are roughly the same and just like the payment variables.  
- All variables have a non-trival number of negative values (this merits investigation for data integrity since negative values on bills presumably mean a customer was overcharged then issued a credit).


<!--
* Suspicious data points: some customers have bill amounts that exceed their balance limits. Does that mean credit one is not enforcing those limits?
* Consider removing or transforming negative values to improve model performance.
* Maybe I should engineer a logical var that is true iff the the customer's bill amount is bigger than their balance limit.
-->

<!--
```python
#filter out the rows where the value for at least one of the billing vars is less than zero at least one of the bill
df.query('bill_ap < 0 or bill_m < 0 or bill_ju < 0 or bill_jy < 0 or bill_ag or bill_s < 0')
```

>*Coding Notes*
>* I first tried doing the filter using the ``.loc()``` function, but all my attempts threw errors.
>  * ```df.loc[ df[['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']]<0]```Threw a ValueError: cannot index with multidimensional key.
>  * ```df[['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].loc[lambda x: x<0]``` same error as above.


```python
#filter out the rows where the value for all the billing vars is less than zero
df.query('bill_ap < 0 & bill_m < 0 & bill_ju < 0 & bill_jy < 0 & bill_ag & bill_s < 0')

#sort df by april bill amount from highest to lowest, then show first 10 rows
df.sort_values(by=['bill_ap'], ascending=False).head(10)
```


```python
#calculate median values for all billing variables
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].median()

#calculate mean absolute deviation (MAD) for all billing vars
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].mad()

#calculate descriptive stats for all billing variables
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].describe()
```

>*Coding Notes*
>* The median, mad, and describe stats are included in profile report. I was just practicing and testing if I could generate stats on a *list of vars* (rather than one at a time).
-->
&nbsp;
#### Payment Variables
{{< figure src="/images/creditone_pmt_dist.png" >}}

* There are 6 continuous payment variables - each represents the amount the customer paid towards their bill for the months of April to September 2005.
* The distributions of the payment and billing variables have similar shapes.
* Notable Differences:
  * The payment variables contain a higher percentage of zero values.
  * The payment variables don't contain any negative values.



&nbsp;
### Correlation Analysis
If any of the features are highly correlated, that may be because they are actually tracking/representing the same information. Since high correlation between dependent variables can harm the performance of ML models, it needs to be handled.  

&nbsp;
#### *Are the 6 billing variables correlated?*

<!--
```python
#calculate kendall's tau correlation between billing vars
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].corr(method='kendall')
```

>*Coding Notes*
>* By default corr function calculates pearson's correlation co-efficient.
>* But it's only appropriate to use Pearson's if the data is normally distributed and we know from the profile report that the distributions for the billing variables are highly skewed.
>* I think there's a way to output only the values below the diagonal.
-->

```python
#calculate kendall's tau correlation between billing vars
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].corr(method='kendall')<!

# name a sample of 1,000 data points selected from the df
smpl=df.sample(n=1000)

#create pairwise plots of bill features using smpl
sns.pairplot(smpl, hue='DEFAULT', kind='scatter', vars=['bill_ap','bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s'])

#save plot as png
plt.savefig('creditone_bill_pairplot.png')
```

{{< figure src="/images/creditone_bill_pairplot.png" >}}

* The billing variables are highly correlated with each other. Max: 0.8112, Min: 0.5853, which is not surprising since spending doesn't change from month to month.
* In general, the correlation is highest between each monthly billing variable and the variable representing the month after. The correlation declines with each proceeding month (e.g. the correlation is highest between the April (bill_ap) and May (bill_m) billing variables, and declines for each subsequent month.)  
    * May impact model performance. Consider only using one of the billing variables in model.



&nbsp;
#### *Are the 6 payment variables correlated?*
<!--
```python
#look at kendall's tau correlations b/t payment variables
df[['pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s']].corr(method='kendall')

#create a pairplot of the payment features using smpl (of 1,000)
sns.pairplot(smpl, hue='DEFAULT', kind='scatter', vars=['pmt_ap','pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s'])
```
-->
{{< figure src="/images/creditone_pmt_pairplot.png" >}}

* One reason to think the payment variables might be highly correlated is that presumably the amount customers pay each month is related to the amount they were billed during the previous month. If true, since the billing variables are highly correlated,  the payment variables would be too.
* The kendall's correlation coefficients are between 0.348 and 0.4126. But the scatter plots don't show any relationship.
* This suggests, somewhat surprising, that the amount customers pay each month is not really related to the amount they are billed.



&nbsp;
#### *Are the payment variables correlated with the billing variables?*

```python
#look at kendall's tau correlations b/t payment variables
df[['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s','pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s']].corr(method='kendall')
```

* I expected to find a moderate to high correlation between the billing and payment variables for reasons outlined abouve.
* The strongest correlations are around 0.51.


&nbsp;
## Credit Limit Analysis
In this section I perform statistical tests to determine whether any of the dependent variables are related to the limit_bal target variable.


&nbsp;
### Age, Sex, & Credit Limit
<!--
```python
sns.boxplot(data=df, x='SEX',y='LIMIT_BAL' )

# save plot as png
plt.savefig("creditone_sex_credit_box1.png")
```
-->
{{< figure src="/images/creditone_age_sex_plots.png" >}}

- There doesn't appear to be any sort of meaningful relationship between age and credit limit.
- The similarity between the box plots for female and male indicates there is no difference in credit limits between men and women. That means SEX is unlikely to be a helpful variable to include in a model prediting LIMIT_BAL.


&nbsp;
### Education, Marriage, & Credit Limit
<!--
```python
sns.catplot(data=df, kind='box', x='EDUCATION', y='LIMIT_BAL').set_xticklabels(rotation=45)

# save plot as png
plt.savefig("creditone_edu_credit_box.png")
```
-->
{{< figure src="/images/creditone_edu_marriage_box.png" >}}

- The box plots for all education levels are very similar. Customers with a graduate education enjoy credit limits that are on average higher than the other groups. It also appears the credit limits for customers with a high school education are slightly skewed to the lower end. The differences aren't dramatic, but a customer's credit limit does appear to be somewhat related to their level of education  
- Although there are some small differences in the plots, it does not appear that a customer's credit limit is related to their marital status.




&nbsp;
## Credit Default Analysis
In this section I examine whether there is a meaningful relationship between any of the dependent variables--demographic, account, billing, & payment variables--and whether or not a customer has defaulted on their loans.
<!--
```python
#obtain correlation co-efficients for all features and target
df.corr().to_markdown
---
    <bound method DataFrame.to_markdown of                  ID  MARRIAGE       AGE  LIMIT_BAL   bill_ap    bill_m  \
    ID         1.000000 -0.029079  0.018678   0.026179  0.016730  0.016705   
    MARRIAGE  -0.029079  1.000000 -0.414170  -0.108139 -0.021207 -0.025393   
    AGE        0.018678 -0.414170  1.000000   0.144713  0.047613  0.049345   
    LIMIT_BAL  0.026179 -0.108139  0.144713   1.000000  0.290389  0.295562   
    bill_ap    0.016730 -0.021207  0.047613   0.290389  1.000000  0.946197   
    bill_m     0.016705 -0.025393  0.049345   0.295562  0.946197  1.000000   
    bill_ju    0.040351 -0.023344  0.051353   0.293988  0.900941  0.940134   
    bill_jy    0.024354 -0.024909  0.053710   0.283236  0.853320  0.883910   
    bill_ag    0.017982 -0.021602  0.054283   0.278314  0.831594  0.859778   
    bill_s     0.019389 -0.023472  0.056239   0.285430  0.802650  0.829779   
    pmt_ap     0.003000 -0.006641  0.019478   0.219595  0.115494  0.164184   
    pmt_m      0.000652 -0.001205  0.022850   0.217202  0.307729  0.141574   
    pmt_ju     0.007793 -0.012659  0.021379   0.203242  0.250237  0.293118   
    pmt_jy     0.039151 -0.003541  0.029247   0.210167  0.233770  0.252305   
    pmt_ag     0.008406 -0.008093  0.021785   0.178408  0.172663  0.181246   
    pmt_s      0.009742 -0.005979  0.026147   0.195236  0.199965  0.217031   
    pay_ap    -0.020270  0.034345 -0.048773  -0.235195  0.285091  0.290894   
    pay_m     -0.022199  0.035629 -0.053826  -0.249411  0.262509  0.269783   
    pay_ju    -0.002735  0.033122 -0.049722  -0.267460  0.239154  0.242902   
    pay_jy    -0.018494  0.032688 -0.053048  -0.286123  0.222327  0.225145   
    pay_ag    -0.011215  0.024199 -0.050148  -0.296382  0.219403  0.221348   
    pay_s     -0.030575  0.019917 -0.039447  -0.271214  0.176980  0.180635   

                bill_ju   bill_jy   bill_ag    bill_s  ...    pmt_ju    pmt_jy  \
    ID         0.040351  0.024354  0.017982  0.019389  ...  0.007793  0.039151   
    MARRIAGE  -0.023344 -0.024909 -0.021602 -0.023472  ... -0.012659 -0.003541   
    AGE        0.051353  0.053710  0.054283  0.056239  ...  0.021379  0.029247   
    LIMIT_BAL  0.293988  0.283236  0.278314  0.285430  ...  0.203242  0.210167   
    bill_ap    0.900941  0.853320  0.831594  0.802650  ...  0.250237  0.233770   
    bill_m     0.940134  0.883910  0.859778  0.829779  ...  0.293118  0.252305   
    bill_ju    1.000000  0.923969  0.892482  0.860272  ...  0.130191  0.300023   
    bill_jy    0.923969  1.000000  0.928326  0.892279  ...  0.143405  0.130011   
    bill_ag    0.892482  0.928326  1.000000  0.951484  ...  0.147398  0.150718   
    bill_s     0.860272  0.892279  0.951484  1.000000  ...  0.158303  0.156887   
    pmt_ap     0.177637  0.182326  0.174256  0.179341  ...  0.157834  0.162740   
    pmt_m      0.160433  0.179712  0.157957  0.167026  ...  0.151830  0.159214   
    pmt_ju     0.130191  0.143405  0.147398  0.158303  ...  1.000000  0.216325   
    pmt_jy     0.300023  0.130011  0.150718  0.156887  ...  0.216325  1.000000   
    pmt_ag     0.207564  0.316936  0.100851  0.099355  ...  0.180107  0.244770   
    pmt_s      0.233012  0.244335  0.280365  0.140277  ...  0.199558  0.252191   
    pay_ap     0.266356  0.241181  0.226924  0.207373  ...  0.019018  0.005834   
    pay_m      0.271915  0.243335  0.226913  0.206684  ... -0.058299  0.009062   
    pay_ju     0.245917  0.244983  0.225816  0.202812  ... -0.043461 -0.069235   
    pay_jy     0.227202  0.227494  0.237295  0.208473  ... -0.046067 -0.053311   
    pay_ag     0.222237  0.224146  0.235257  0.234887  ... -0.046858 -0.055901   
    pay_s      0.179125  0.179785  0.189859  0.187068  ... -0.064005 -0.070561   

                 pmt_ag     pmt_s    pay_ap     pay_m    pay_ju    pay_jy  \
    ID         0.008406  0.009742 -0.020270 -0.022199 -0.002735 -0.018494   
    MARRIAGE  -0.008093 -0.005979  0.034345  0.035629  0.033122  0.032688   
    AGE        0.021785  0.026147 -0.048773 -0.053826 -0.049722 -0.053048   
    LIMIT_BAL  0.178408  0.195236 -0.235195 -0.249411 -0.267460 -0.286123   
    bill_ap    0.172663  0.199965  0.285091  0.262509  0.239154  0.222327   
    bill_m     0.181246  0.217031  0.290894  0.269783  0.242902  0.225145   
    bill_ju    0.207564  0.233012  0.266356  0.271915  0.245917  0.227202   
    bill_jy    0.316936  0.244335  0.241181  0.243335  0.244983  0.227494   
    bill_ag    0.100851  0.280365  0.226924  0.226913  0.225816  0.237295   
    bill_s     0.099355  0.140277  0.207373  0.206684  0.202812  0.208473   
    pmt_ap     0.157634  0.185735 -0.025299 -0.023027 -0.026565 -0.035861   
    pmt_m      0.180908  0.148459 -0.046434 -0.033337 -0.033590 -0.035863   
    pmt_ju     0.180107  0.199558  0.019018 -0.058299 -0.043461 -0.046067   
    pmt_jy     0.244770  0.252191  0.005834  0.009062 -0.069235 -0.053311   
    pmt_ag     1.000000  0.285576 -0.005223 -0.003191 -0.001944 -0.066793   
    pmt_s      0.285576  1.000000 -0.001496 -0.006089 -0.009362  0.001295   
    pay_ap    -0.005223 -0.001496  1.000000  0.816900  0.716449  0.632684   
    pay_m     -0.003191 -0.006089  0.816900  1.000000  0.819835  0.686775   
    pay_ju    -0.001944 -0.009362  0.716449  0.819835  1.000000  0.777359   
    pay_jy    -0.066793  0.001295  0.632684  0.686775  0.777359  1.000000   
    pay_ag    -0.058990 -0.080701  0.575501  0.622780  0.662067  0.766552   
    pay_s     -0.070101 -0.079269  0.474553  0.509426  0.538841  0.574245   

                 pay_ag     pay_s  
    ID        -0.011215 -0.030575  
    MARRIAGE   0.024199  0.019917  
    AGE       -0.050148 -0.039447  
    LIMIT_BAL -0.296382 -0.271214  
    bill_ap    0.219403  0.176980  
    bill_m     0.221348  0.180635  
    bill_ju    0.222237  0.179125  
    bill_jy    0.224146  0.179785  
    bill_ag    0.235257  0.189859  
    bill_s     0.234887  0.187068  
    pmt_ap    -0.036500 -0.058673  
    pmt_m     -0.037093 -0.058190  
    pmt_ju    -0.046858 -0.064005  
    pmt_jy    -0.055901 -0.070561  
    pmt_ag    -0.058990 -0.070101  
    pmt_s     -0.080701 -0.079269  
    pay_ap     0.575501  0.474553  
    pay_m      0.622780  0.509426  
    pay_ju     0.662067  0.538841  
    pay_jy     0.766552  0.574245  
    pay_ag     1.000000  0.672164  
    pay_s      0.672164  1.000000  

    [22 rows x 22 columns]>
```
-->


&nbsp;
#### Sex & Default
A quick and intuitive way to examine the relationship between two categorical variables is to compare how sex is distributed in the entire sample to how it's distributed in customers who have defaulted vs. not-defaulted. If whether or not a customer defaults on their loans is related to their gender, the distributions will be different.
<!--
```python
sns.displot(data=df, x="SEX", stat='density', shrink=.8)
```


```python
sns.displot(data=df, x="SEX", col='DEFAULT', stat='density', shrink=.8)

#save plot as png
plt.savefig('creditone_sex_default_notdefault.png')
```
-->
{{< figure src="/images/creditone_sex_default_notdefault.png" >}}

*Observations*
* When we compare the male/female breakdown in the entire sample to the breakdowns among customers who have and have not defaulted on their loans, they are all very similar, suggesting whether or not a customer defaults on their loans is unrelated to their gender.
<!--
* Consequently, sex is unlikely to be useful feature (dependent variable)  predicting whether a customer will default on their loans.
-->

&nbsp;
#### Education & Default
<!--
```python
#create probability chart for the EDU var
sns.displot(data=df, x ="EDUCATION", discrete=True, shrink = .8, stat='probability').set_xticklabels(rotation=45)
```


```python
#create subplots showing the distribution for the EDU var conditional on the DEFUALT var
sns.displot(data=df, x ="EDUCATION", col = 'DEFAULT', shrink = .8, stat='probability').set_xticklabels(rotation=45)
```
-->
{{< figure src="/images/creditone_edu.png" >}}


*Observations*
* Since higher levels of education are correlated with higher incomes, and presumably higher incomes are correlated with lower rates of credit default, I was expecting to see some kind of relationship. However, the distributions of education levels for customers who have and haven't defaulted on their loans are similar to one another and similar to the distribution in the entire sample.

<!--
```python
# this code generates a messed up plot
g = sns.FacetGrid(data=df, col="DEFAULT", sharey=False)
g.map_dataframe(sns.histplot, x="EDUCATION", stat='probability')
g.set_xticklabels(rotation=45)
```

>*Coding Notes*
>* I created the g subpolots at fisrt and thought they revealed an interesting relationship between level of edu and whether a customer had defaulted on their loans. It looked like relative to other groups, those with a high school edu level are less likely to default on their loans, and those with the other edu level are more likely.
>* However, I continued to mess around and try out different types of plots and use different code, and none of the subsequent plots I generated showed the pattern. They all showed the pattern we would expect to see if there is no significant relationship between level of education and whether a customer defaulted on their loan.
>* I investigated the issue extensively on my own and tried to figure it out with David Schwab, but it is still a mystery.
  >* David pointed out that in the weird charts it looks like bars for the "high school" and "other" levels the bars have been swapped. That sounds plausible to me, but we could not figure out how that could have happened, or how to get rid of it.  
-->

&nbsp;
#### Age & Default
<!--
```python
sns.catplot(data=df, kind='box', x='DEFAULT', y='AGE')

#save plot as png
plt.savefig('creditone_age_default.png')
```
-->
{{< figure src="/images/creditone_age_full_dflt_box.png" >}}

*Observations*
* Comparing the above boxplots, we can see the mean, IQR (middle 50% of data points), and range for age is nearly identical in all three plots. As before, this suggests that whether or not customers default on their loans is unrelated to their age (which I find surprising).  



&nbsp;
#### Marriage & Default

<!--
```python
#filter the df to return only rows where value for DEFAULT == default
dflt = df.loc[df['DEFAULT']=='default']
dflt.head().to_markdown
---

    <bound method DataFrame.to_markdown of     ID     SEX        EDUCATION  MARRIAGE  AGE  LIMIT_BAL  bill_ap  bill_m  \
    0    1  female       university         1   24      20000        0       0   
    1    2  female       university         2   26     120000     3261    3455   
    13  14    male       university         2   30      70000    36894   36137   
    16  17    male  graduate school         2   24      20000    19104   17905   
    21  22  female       university         1   39     120000      316     632   

        bill_ju  bill_jy  ...  pmt_jy  pmt_ag  pmt_s  pay_ap  pay_m  pay_ju  \
    0         0      689  ...       0     689      0      -2     -2      -1   
    1      3272     2682  ...    1000    1000      0       2      0       0   
    13    66782    65701  ...    3000       0   3200       2      0       0   
    16    18338    17428  ...    1500       0   3200       2      2       2   
    21        0      316  ...       0     316    316      -1     -1      -1   

        pay_jy  pay_ag  pay_s  DEFAULT  
    0       -1       2      2  default  
    1        0       2     -1  default  
    13       2       2      1  default  
    16       2       0      0  default  
    21      -1      -1     -1  default  

    [5 rows x 25 columns]>
```



```python
#filter the df to return only rows where value for DEFAULT == not default
nodflt = df.loc[df['DEFAULT']=='not default']
nodflt.head().to_markdown
---
    <bound method DataFrame.to_markdown of    ID     SEX        EDUCATION  MARRIAGE  AGE  LIMIT_BAL  bill_ap  bill_m  \
    2   3  female       university         2   34      90000    15549   14948   
    3   4  female       university         1   37      50000    29547   28959   
    4   5    male       university         1   57      50000    19131   19146   
    5   6    male  graduate school         2   37      50000    20024   19619   
    6   7    male  graduate school         2   29     500000   473944  483003   

       bill_ju  bill_jy  ...  pmt_jy  pmt_ag  pmt_s  pay_ap  pay_m  pay_ju  \
    2    14331    13559  ...    1000    1500   1518       0      0       0   
    3    28314    49291  ...    1200    2019   2000       0      0       0   
    4    20940    35835  ...   10000   36681   2000       0      0       0   
    5    19394    57608  ...     657    1815   2500       0      0       0   
    6   542653   445007  ...   38000   40000  55000       0      0       0   

       pay_jy  pay_ag  pay_s      DEFAULT  
    2       0       0      0  not default  
    3       0       0      0  not default  
    4      -1       0     -1  not default  
    5       0       0      0  not default  
    6       0       0      0  not default  

    [5 rows x 25 columns]>
```

-->

```python
fig, axes = plt.subplots(1, 3, figsize=(13,5), sharey=True)
axes[0].set_title('All Customers', fontsize=14)
axes[1].set_title('Customers who defualted', fontsize=14)
axes[2].set_title('Customers who did not default', fontsize=14)

# To iterate over all items in a multidimensional numpy array, use the `flat` attribute
for ax in axes.flat:
    # set xticks...
    ax.set(xticks=[0, 1, 2, 3])

fig.suptitle(t='Is there a relationship between customers marital status & whether they defaulted?', verticalalignment='baseline', fontsize=18)
labels=['Other', 'Married', 'Single', 'Divorced']


sns.histplot(ax=axes[0], data=df, x ='MARRIAGE', stat='probability', discrete=True, shrink=.8)
axes[0].set_xlabel('')
axes[0].set_xticklabels(labels=labels, rotation=45)

sns.histplot(ax=axes[1], data=dflt, x ='MARRIAGE', stat='probability', discrete=True, shrink=.8)
axes[1].set_xlabel('')
axes[1].set_xticklabels(labels=labels, rotation=45)

sns.histplot(ax=axes[2], data=nodflt, x ='MARRIAGE', stat='probability', discrete=True, shrink=.8)
axes[2].set_xticklabels(labels=labels, rotation=45)
axes[2].set_xlabel('')
```

{{< figure src="/images/creditone_marriage_default_notdefault.png" >}}

*Observations*
*  Comparing the above plots, it appears that single customers may be slightly less likely to default on their loans than customers in the other three categories. However, the difference is small. The larger story is that customer's maritial status appears to have very little to do with whether or not they default on their loans.






&nbsp;
#### Billing, Payment, & Credit Default
<!--
```python
fig, axes = plt.subplots(2, 3, figsize=(20,18), sharey=True)

fig.suptitle('Monthly Bill Amounts Grouped by DEFAULT')

sns.boxplot(ax=axes[0, 0], data=df, x='DEFAULT', y='bill_ap')
sns.boxplot(ax=axes[0, 1], data=df, x='DEFAULT', y='bill_m')
sns.boxplot(ax=axes[0, 2], data=df, x='DEFAULT', y='bill_ju')
sns.boxplot(ax=axes[1, 0], data=df, x='DEFAULT', y='bill_jy')
sns.boxplot(ax=axes[1, 1], data=df, x='DEFAULT', y='bill_ag')
sns.boxplot(ax=axes[1, 2], data=df, x='DEFAULT', y='bill_s')

```
-->
{{< figure src="/images/creditone_billing_default.png" >}}

*Observations*
* I only included the plots for the billing variables here, but the boxplots for the payment variables are nearly identical.
* The distributions for all the payment and billing variables are very similar all the groups of interest. However, I am not confident this warrants drawing the same conclusions. I have more questions than observations:
  * We saw earlier that the billing variables are highly correlated with each other. How should I account for that in both my analysis and model building?
  * There are some significant outliers. How can I identify and remove them? Is it worth it?
  * Other than some differences in outliers and a noticable difference in the length of the wiskers in bill_ag, all the box plots are basically the same. What, if anything, can we conclude from that?


<!--
&nbsp;
### Payment & Default

```python
fig, axes = plt.subplots(2, 3, figsize=(20,25), sharey=True)

fig.suptitle('Monthly Payment Variables Grouped by DEFAULT')

sns.boxplot(ax=axes[0, 0], data=df, x='DEFAULT', y='pmt_ap')
sns.boxplot(ax=axes[0, 1], data=df, x='DEFAULT', y='pmt_m')
sns.boxplot(ax=axes[0, 2], data=df, x='DEFAULT', y='pmt_ju')
sns.boxplot(ax=axes[1, 0], data=df, x='DEFAULT', y='pmt_jy')
sns.boxplot(ax=axes[1, 1], data=df, x='DEFAULT', y='pmt_ag')
sns.boxplot(ax=axes[1, 2], data=df, x='DEFAULT', y='pmt_s')

```
-->







<!--
### Additional Stuff

```python
#convert DEFUALT into numerical feature
df['DEFAULT'] =df['DEFAULT'].astype('category').cat.codes
"""
I read that it is less memory intensive to convert non-numerical variables into numerical vars if the vars are
have the category data type instead of the obj dtype.
"""

#obtain correlation co-efficients for all features and bill_ap
df[df.columns[1:]].corr()['bill_ap'][:]

#filter df to exclude cases where bill_ap value is 0, then print last 20 cases in df
df.loc[df.bill_ap !=0].tail(20)

#filter the df to return only the rows where the april bill var is not 0
nzbill=df.loc[df['bill_ap'] !=0]

# filter the data to return only rows where june bill var is greater than 0
nzbill2=df['bill_ju'].loc[lambda x: x > 0]

nzbill2.head(20)
---
    1       3272
    2      14331
    3      28314
    4      20940
    5      19394
    6     542653
    7        221
    8      12211
    10      2513
    11      8517
    12      6500
    13     66782
    14     59696
    15     28771
    16     18338
    17     70074
    20     20616
    22     44006
    23       560
    24      5398
    Name: bill_ju, dtype: int64
```


```python
#filter data to return only rows where the april AND june bill vars are not zero
nzbill3=df.query('bill_ap !=0 & bill_ju !=0')
nzbill3.shape
---
    (25140, 25)
```


>*Coding Notes*
>* You can filter data using the ```df.query()``` function with string statements composed of var names and boolean operators.
>* The first time I ran ```nzbill3.shape()``` it didn't work because I ended it with (). ```.shape``` is not a function that you want to perform on the data. It's more like asking for some kind of description of the data. That's why it shouldn't have parentheses at the end.


```python
#get the corr between the filtered data and DEFAULT
nzbill3[nzbill3.columns[1:]].corr()['DEFAULT'][:]
---

    MARRIAGE     0.024541
    AGE         -0.006730
    LIMIT_BAL    0.178940
    bill_ap      0.002718
    bill_m       0.003591
    bill_ju      0.008230
    bill_jy      0.011548
    bill_ag      0.010313
    bill_s       0.015756
    pmt_ap       0.053151
    pmt_m        0.058686
    pmt_ju       0.058949
    pmt_jy       0.059988
    pmt_ag       0.060119
    pmt_s        0.074500
    pay_ap      -0.252730
    pay_m       -0.269089
    pay_ju      -0.276352
    pay_jy      -0.291087
    pay_ag      -0.316754
    pay_s       -0.363338
    DEFAULT      1.000000
    Name: DEFAULT, dtype: float64
```


>*Notes*
>* Because I am concerned that the high percentage of 0 values in the billing variables is negatively impacting the analysis, I wanted to see if the correlations with DEFAULT would improve if I excluded the data where one or more of the billing vars has a value of 0.
>* However, it occured to me after getting the above .corr and thinking about the math behind correlation that excluding the cases with 0 values wouldn't impact the co-efficients.


```python
#filter the data to return only rows where the April bill var is less than 100,000
df.loc[df['bill_ap']<= -100000]

#get number of columens and rows
nzbill3.shape
---
    (25140, 25)
```
-->

&nbsp;
## Predicting Customer Credit Limits
In this section I try building a machine learning model that can accurately predict each customer's credit limit (i.e. limit_bal) from their demographic, account, billing, and payment data.


```python
import pandas as pd
import numpy as np
from math import sqrt

# for one hot encoding categorical variables
from sklearn.preprocessing import LabelEncoder

# ML algorithms
from sklearn.ensemble import RandomForestRegressor
from sklearn.linear_model import LinearRegression
from sklearn.svm import SVR
from sklearn import linear_model

# model metrics
from sklearn.metrics import mean_squared_error
from sklearn.metrics import r2_score
from sklearn.model_selection import cross_val_score

# for splitting data into training and test sets
from sklearn.model_selection import train_test_split
```




&nbsp;
#### *Data Preparation*

- For convenience, I moved LIMIT_BAL to the last column in my dataframe.
- Turn categorical variables into a set of dummy variables using *One-Hot-Encoding*.
- Identify which columns are the feature variables and which is the target.
- Split the data into a training data set and a testing data set (80/20 split).

<!--
## Import & Verify Data

```python
#import numerical only data
df= pd.read_csv('num_only_credit_one_data.csv')

df.head()
```


```python
df.shape
---
    (30000, 31)
```


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 30000 entries, 0 to 29999
    Data columns (total 31 columns):
     #   Column               Non-Null Count  Dtype
    ---  ------               --------------  -----
     0   ID                   30000 non-null  int64
     1   SEX                  30000 non-null  int64
     2   AGE                  30000 non-null  int64
     3   LIMIT_BAL            30000 non-null  int64
     4   bill_ap              30000 non-null  int64
     5   bill_m               30000 non-null  int64
     6   bill_ju              30000 non-null  int64
     7   bill_jy              30000 non-null  int64
     8   bill_ag              30000 non-null  int64
     9   bill_s               30000 non-null  int64
     10  pmt_ap               30000 non-null  int64
     11  pmt_m                30000 non-null  int64
     12  pmt_ju               30000 non-null  int64
     13  pmt_jy               30000 non-null  int64
     14  pmt_ag               30000 non-null  int64
     15  pmt_s                30000 non-null  int64
     16  pay_ap               30000 non-null  int64
     17  pay_m                30000 non-null  int64
     18  pay_ju               30000 non-null  int64
     19  pay_jy               30000 non-null  int64
     20  pay_ag               30000 non-null  int64
     21  pay_s                30000 non-null  int64
     22  DEFAULT              30000 non-null  int64
     23  edu_graduate school  30000 non-null  int64
     24  edu_high school      30000 non-null  int64
     25  edu_other            30000 non-null  int64
     26  edu_university       30000 non-null  int64
     27  mar_0                30000 non-null  int64
     28  mar_1                30000 non-null  int64
     29  mar_2                30000 non-null  int64
     30  mar_3                30000 non-null  int64
    dtypes: int64(31)
    memory usage: 7.1 MB


Everything with my dataset looks as I expect. So I will move on to specifying the variables I was to use as features (IVs) and the one I want to use as target for my model predicting credit limit.
-->
<!--
```python
#step 1: create a variabled to hold the LIMIT_BAL var
lim=df['LIMIT_BAL']

#step 2: drop the LIMIT_BAL column
df.drop(labels='LIMIT_BAL', axis=1, inplace=True)

#step 3: insert 'lim' as last column of df
df.insert(loc=30, column='limit', value=lim)

df.head()
```
-->


<!--
# Transform Non-numeric Values

```python
# select columns with non-numeric data and create a new df with only those columns (so changes aren't in original df)
obj_df = df.select_dtypes(include=['object']).copy()
obj_df.head()
```


```python
# change data types from object to category to make transforminmg faster
cat_df = obj_df[['SEX', 'EDUCATION', 'DEFAULT']].astype('category')

"""
I read that it's more efficient to transform non-numerical data into numerical data
if the vars to be transformed have the 'category data type instead of 'object' dtype
To find out how long things take to run use the %timeit magic
"""
cat_df.dtypes
---
    SEX          category
    EDUCATION    category
    DEFAULT      category
    dtype: object
```



```python
#import label encoder from sklearn
from sklearn.preprocessing import LabelEncoder

#instiate the label encoder function, call it le
le = LabelEncoder()
```


```python
# use label encoder to  encode values in SEX column as numbers and add new column to df to preserve encoded values
cat_df['sex_le']=le.fit_transform(cat_df['SEX'])

cat_df.head()
```



```python
# use label encoder to  encode values in SEX column as numbers and add new column to df to preserve encoded values
cat_df['dflt_le']=le.fit_transform(cat_df['DEFAULT'])

cat_df.head()
```


```python
pd.get_dummies(cat_df, columns=['EDUCATION'], prefix=['edu'])

cat_df.head()
```

>*Coding Notes*
>* The first time I executed the ```get_dummies``` code it didn't do anything.
>* I had to add ```cat_df=``` before ```pd.get_dummies``` for the dummy variables to show up in the df.
>* I asked why I needed the ```cat_df=``` before get_dummies by not other things (e.g. df.head)? Mike said the reason is that without the assignment (```cat_df=```) at the beginning of the line, pandas doesn't assign the output of the get_dummies operation to anything, and so it isn't preserved. To put it differently, putting ```cat_df=``` at the front of the line does to same thing as adding an ```inplace=True``` parameter would do (Except the get_dummies function doesn't accept an ```inplace=``` parameter). (from Slack exchange on 1/4/2020)


```python
cat_df=pd.get_dummies(cat_df, columns=['EDUCATION'], prefix=['edu'])

cat_df.head()
```
-->

<!--
## Perform Transformations on Original df
To avoid issues arising from having to merge 2 df, I am going to create a copy of the original df and transform the original non-numerical variables, instead of adding new variables to the df.


```python
#create a copy of the dataframe to retain transformations
num_df= df.copy()

# use label encoder to change values in SEX into numbers
num_df['SEX']=le.fit_transform(num_df['SEX'])

num_df.head()
```



```python
# use label encoder to change values in DEFUALT into numbers
num_df['DEFAULT']=le.fit_transform(num_df['DEFAULT'])

# transform EDU var into 4 dummy vars (1 var for each category)
num_df=pd.get_dummies(num_df, columns=['EDUCATION'], prefix=['edu'])

#transform MARRIAGE var in 4 dummy vars
num_df=pd.get_dummies(num_df, columns=['MARRIAGE'], prefix=['mar'])

#verify changes
num_df.head()
```


```python
#Export Numerical df
num_df.to_csv('num_only_credit_one_data.csv', index = False, header=True)
```
-->





<!--
### Specify the Target Variable

```python
#filter the data to return my dependent var (the limit column)
y = df.loc[:,'limit']

#verify that the first 5 values in y match the values in the limit column above
y.head()
```




    0     20000
    1    120000
    2     90000
    3     50000
    4     50000
    Name: limit, dtype: int64




```python
#verify that y has 30,000 rows
y.shape
```




    (30000,)


&nbsp;
### *Model 1: All Dependent Variables*
-->
I included all demographic, account, billing, and payment variables as features in my initial model.

<!--
```python
# filter df to specify the feature variables
X = df.iloc[:, 1:30]

#verify that all but the ID and limit columns were selected
print('Summary of Feature Variables')
X.head()
```



```python
#create a dictionary to hold the algos
algosClass = [ ]

#add the randomn forest regressor to the dictionary
algosClass.append(('Random Forest Regressor',RandomForestRegressor()))

#add the liner regression algo to the dictionary
algosClass.append(('Linear Regression',LinearRegression()))

#add the SV regressor to the dictionary
algosClass.append(('Support Vector Regression',SVR()))
```



&nbsp;
### Train-Test Split

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2, random_state=123)
```
-->


&nbsp;
#### Train Model & Make Predictions

```python
#create two empty lists to hold the names and cross validation results from each model
results = []
names = []

#create a for-loop that will
for name, model in algosClass:
    result = cross_val_score(model, X,y, cv=3, scoring='r2')
    names.append(name)
    results.append(result)
```



&nbsp;
#### Model Performance

```python
#run the for loop and print the mean cross validation score for each model
for i in range(len(names)):
    print(names[i],results[i].mean())
---

    Random Forest Regressor 0.46700202592747336
    Linear Regression 0.35871695390197167
    Support Vector Regression -0.05039128320492573
```

The model built using the random forest regressor performed slightly better than the other two models. However, did little better than chance at predicting customer's credit limits (i.e.limit_bal).



<!--
&nbsp;
### *Model 2: Excluding Demographic Variables*

```python
## Select Feature Variables: filter the data to return only columns for bill amount, payment amount, payment history, and default var
X2 = df.iloc[:, 3:22]

print('Summary of Model 2 Features')
X2.head()
```



```python
#create two empty lists to hold the names and cross validation results from each model
results2 = []
names2 = []

#get cross validation scores for model 2
for name, model in algosClass:
    result = cross_val_score(model, X2,y, cv=3, scoring='r2')
    names2.append(name)
    results2.append(result)

#run the for loop and print the mean cross validation score for each model
print("Summary of Model2 Cross Validation Scores")

for i in range(len(names2)):
    print(names[i],results[i].mean())
---

    Summary of Model2 Cross Validation Scores
    Random Forest Regressor 0.46700202592747336
    Linear Regression 0.35871695390197167
    Support Vector Regression -0.05039128320492573
```



&nbsp;
#### Train & Test the Model

```python
#instantiate the random forest regression algo
rfr = RandomForestRegressor()
```


```python
#train the rfr model on X_train and y_train
# model = rfr.fit(X_train, y_train)

#use trained model to predict credit limit amounts for the X_test data
# y_predict = model_1.predict(X_test)
```
-->



<!--
>*Coding Notes*
>* The ```test_size =``` parameter specifies what percent of you dataset will be used for training and training. If you don't specify an amount, the default is 0.25.
>* The ```random_state =``` parameter can be set to any number. The purpose of it is to make sure that every time the model is run,  the same observations are used in the training and testing sets. If this parameter is not included, every time you run the model it may select a slightly different set of observations.


```python
cross_val_score(rfr, X, y, cv=3)
```


```python
r2_score(y_test, y_predict)
```


```python
mean_squared_error(y_test, y_predict)
```
-->


[Link to GitHub Repository](https://github.com/kpiatti/creditone-project)
