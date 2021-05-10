---
date: 2020-12-13T11:00:59-04:00
description: ""
featured_image: "/images/credit_feature.jpg"
title: "Credit One Project"
show_reading_time: true
---


```python
#import required packages
from sqlalchemy import create_engine
import pymysql
import pandas as pd
import numpy as np
```

## *Obtain Data*

The CreditOne data is on the MySQL sever, so I need to connect to the server, query the database for the data, get the data (formatted as dataframe), and finally export the data as a .csv file.

Even though it's not necessary, I prefer to export the raw data as a .csv before doing any processing/wrangling. This way I have a untouched copy of data that I can use for comparisons and verification.

```python
#connect python to MySQL server (database?)
db_connection_str = 'mysql+pymysql://deepanalytics:Sqltask1234!@34.73.222.197/deepanalytics'
```


```python
#create engine for SQLalchemy to interface with the database API (see notes below)
db_connection = create_engine(db_connection_str)

#query the credit one data and extract it into a pd df
sql_query = pd.read_sql('SELECT * FROM credit', con=db_connection)

#create pandas df from SQL query
df = pd.DataFrame(sql_query)
```

*Coding Note*
* Initially missed the ```db_connection = create_engine(db_connection_str)``` command. Resulted in a ```nameerror: 'db_connection' not defined```. I removed the _str from the first line, re-ran the cell, and was able to successfully execute the query. So I'm unsure why (or if) the ```create_engine()``` command is needed. (See my SQL collection in Zotero).


```python
#verify data was properly extracted
df.head().to_markdown
---

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
```

* *Data Source* — academic paper found [here](https://bradzzz.gitbooks.io/ga-seattle-dsi/content/dsi/dsi_05_classification_databases/2.1-lesson/assets/datasets/DefaultCreditCardClients_yeh_2009.pdf)
* *Dats Supplement* — for variable definitions see creditone_data_info.pdf ([here](/folder/creditone_data_info1.pdf)).


```python
#get info about df variables
df.info()
---
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 30204 entries, 0 to 30203
    Data columns (total 25 columns):
     #   Column           Non-Null Count  Dtype
    ---  ------           --------------  -----
     0   MyUnknownColumn  30204 non-null  object
     1   X1               30204 non-null  object
     2   X2               30204 non-null  object
     3   X3               30204 non-null  object
     4   X4               30204 non-null  object
     5   X5               30204 non-null  object
     6   X6               30204 non-null  object
     7   X7               30204 non-null  object
     8   X8               30204 non-null  object
     9   X9               30204 non-null  object
     10  X10              30204 non-null  object
     11  X11              30204 non-null  object
     12  X12              30204 non-null  object
     13  X13              30204 non-null  object
     14  X14              30204 non-null  object
     15  X15              30204 non-null  object
     16  X16              30204 non-null  object
     17  X17              30204 non-null  object
     18  X18              30204 non-null  object
     19  X19              30204 non-null  object
     20  X20              30204 non-null  object
     21  X21              30204 non-null  object
     22  X22              30204 non-null  object
     23  X23              30204 non-null  object
     24  Y                30204 non-null  object
    dtypes: object(25)
    memory usage: 5.8+ MB
```

### *Export Raw Data As .csv*

```python
#export df as .csv file
df.to_csv ('raw_creditone_data.csv', index = False)
```





# Data Wrangling

```python
#import exported csv file with raw data
df = pd.read_csv('raw_credit_one_data.csv')

#verify imported data looks as expected
df.head()

# number of rows and columns in df as benchmark before making any changes
df.shape
---
    (30204, 25)
```

*Data Stats*
  - 23 features/independent variables
  - 1 dependent variable
  - 1 ID column
  - 30,204 rows


### *Change Column Headings*

```python
#change column headings to values in index (row) 0
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
```
*Coding Notes*
* See [Stack Overflow](https://stackoverflow.com/questions/26147180/convert-row-to-column-header-for-pandas-dataframe) for more info.
* ```df.iloc[pd.RangeIndex(len(df)).drop(0)]``` is for situations where df either doesn't have or you don't know if it has **unique index numbers** for each row. Otherwise use ```df.drop(df.index[0])```.


```python
#drop row 0 that contains column headings
df.drop(df.index[0], inplace = True)

#verify df is -1 row
len(df)

#verify column headings row was dropped
df.head()
```


### *Remove Duplicate Data*

```python
#find and print all duplicate rows in the df
duplicate_rows = df[df.duplicated()]
print(duplicate_rows)
---

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
```


```python
#drop duplicate rows from the df
df.drop_duplicates(inplace = True)  #inplace=True is to make change to df permanent

#verify rows were dropped
len(df)
---
    30002
```


### *Handle Missing Values*

```python
#check for missing values
print(df.isnull().sum())
---
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


```python
#drop the row with missing ID
df.dropna(inplace=True)

#verify 1 row dropped
len(df)
---
    30001
```



### *Invalid Data*

```python
#locate rows 201-206 (not inclusive)
df.loc[201:206]

#drop row 203 containing original column headings
df.drop(203, inplace=True)

#get number of rows in df
len(df)
---
    30000
```


```python
#only row 201 should appear b/c others were dropped above
df.loc[201:210].to_markdown
---
    <bound method DataFrame.to_markdown of 0     ID LIMIT_BAL     SEX        EDUCATION MARRIAGE AGE PAY_0 PAY_2 PAY_3  \
    201  201    180000  female  graduate school        1  38    -2    -2    -2   

    0   PAY_4  ... BILL_AMT4 BILL_AMT5 BILL_AMT6 PAY_AMT1 PAY_AMT2 PAY_AMT3  \
    201    -2  ...         0         0         0        0        0        0   

    0   PAY_AMT4 PAY_AMT5 PAY_AMT6 default payment next month  
    201        0        0        0                not default  

    [1 rows x 25 columns]>
```
*Coding Notes*
* Dropping rows doesn't stick unless you add ```inplace = True``` parameter to command. Read [this](https://stackabuse.com/python-with-pandas-dataframe-tutorial-with-examples/) to understand why parameter is needed.
* After dropping index, that index number is dropped from the df (e.g. if you drop index 0, when you use df.head() the first entry will be index 1). In other words, the data from below does not move up and fill the dropped indices as it would in an Excel file..



### *Change Variable Data Types*

```python
#look at data types for each variable
df.dtypes
---
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
```


### *Continuous Variables*

```python
df.LIMIT_BAL.value_counts()
---
    50000      3365
    20000      1976
    30000      1610
    80000      1567
    200000     1528
               ...
    740000        2
    690000        1
    327680        1
    760000        1
    1000000       1
    Name: LIMIT_BAL, Length: 81, dtype: int64
```


```python
#change data type of LIMIT_BAL var from object to int
df["LIMIT_BAL"] = df['LIMIT_BAL'].astype('int')

#change data type of AGE var from object to int
df["AGE"] = df['AGE'].astype('int')
```


```python
#get number of obs for each value in PAY_0
df.PAY_0.value_counts()
---
    0     14737
    -1     5686
    1      3688
    -2     2759
    2      2667
    3       322
    4        76
    5        26
    8        19
    6        11
    7         9
    Name: PAY_0, dtype: int64
```
It turns out the values in this variable represent if and how long an account has been delinquent. It is probably better treated as a categorical variable.

```python
#get number of obs for each value in BILL_AM1
df.BILL_AMT1.value_counts()
---
    0         2008
    390        244
    780         76
    326         72
    316         63
              ...
    216          1
    192727       1
    4434         1
    38579        1
    96644        1
    Name: BILL_AMT1, Length: 22723, dtype: int64
```



```python
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
```


```python
df.PAY_AMT4.value_counts()
---
    0        6408
    1000     1394
    2000     1214
    3000      887
    5000      810
             ...
    12164       1
    17133       1
    2466        1
    3187        1
    3859        1
    Name: PAY_AMT4, Length: 6937, dtype: int64
```



```python
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

#verify change
df.dtypes
---
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
#get counts of male and female customers in df
df.SEX.value_counts()
---
    female    18112
    male      11888
    Name: SEX, dtype: int64
```



```python
#get counts for education categories
df.EDUCATION.value_counts()
---
    university         14030
    graduate school    10585
    high school         4917
    other                468
    Name: EDUCATION, dtype: int64
```



```python
#get counts for marriage categories
df.MARRIAGE.value_counts()
---
    2    15964
    1    13659
    3      323
    0       54
    Name: MARRIAGE, dtype: int64
```

```python
# get counts for the target var, i.e. DEFAULT
df.DEFAULT.value_counts()
---
    not default    23364
    default         6636
    Name: DEFAULT, dtype: int64
```


### *Rename Features*

```python
#rename PAY features to convey month
df.rename(columns = {'PAY_0': 'pay_s', 'PAY_2':'pay_ag', 'PAY_3':'pay_jy','PAY_4':'pay_ju', 'PAY_5':'pay_m', 'PAY_6':'pay_ap'}, inplace = True)

#rename BILL_AMT features to convey month
df.rename(columns = {'BILL_AMT1': 'bill_s', 'BILL_AMT2':'bill_ag', 'BILL_AMT3':'bill_jy','BILL_AMT4':'bill_ju', 'BILL_AMT5':'bill_m', 'BILL_AMT6':'bill_ap'}, inplace = True)

#rename PAY_AMT features to convey month
df.rename(columns = {'PAY_AMT1': 'pmt_s', 'PAY_AMT2':'pmt_ag', 'PAY_AMT3':'pmt_jy','PAY_AMT4':'pmt_ju', 'PAY_AMT5':'pmt_m', 'PAY_AMT6':'pmt_ap'}, inplace = True)

#verify features were renamed
df.head().to_markdown
---

    <bound method DataFrame.to_markdown of 0 ID  LIMIT_BAL     SEX   EDUCATION MARRIAGE  AGE pay_s pay_ag pay_jy pay_ju  \
    1  1      20000  female  university        1   24     2      2     -1     -1   
    2  2     120000  female  university        2   26    -1      2      0      0   
    3  3      90000  female  university        2   34     0      0      0      0   
    4  4      50000  female  university        1   37     0      0      0      0   
    5  5      50000    male  university        1   57    -1      0     -1      0   

    0  ... bill_ju bill_m  bill_ap  pmt_s  pmt_ag  pmt_jy  pmt_ju  pmt_m  pmt_ap  \
    1  ...       0      0        0      0     689       0       0      0       0   
    2  ...    3272   3455     3261      0    1000    1000    1000      0    2000   
    3  ...   14331  14948    15549   1518    1500    1000    1000   1000    5000   
    4  ...   28314  28959    29547   2000    2019    1200    1100   1069    1000   
    5  ...   20940  19146    19131   2000   36681   10000    9000    689     679   

    0      DEFAULT  
    1      default  
    2      default  
    3  not default  
    4  not default  
    5  not default  

    [5 rows x 25 columns]>
```


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
```


```python
#move LIMIT_BAL column next to other account/credit variables
df = movecol(df,
             cols_to_move=['LIMIT_BAL'],
             ref_col='AGE',
             place='After')
```





```python
#re-order pay columns
df = movecol(df,
             cols_to_move=['pay_ap', 'pay_m', 'pay_ju', 'pay_jy', 'pay_ag', 'pay_s'],
             ref_col='LIMIT_BAL',
             place='After')

#re-order pmt columns
df = movecol(df,
             cols_to_move=['pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s'],
             ref_col='LIMIT_BAL',
             place='After')

#re-order bill columns
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

# Exploratory Data Analysis (EDA)

```python
#import packages
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
plt.style.use('fivethirtyeight')
import numpy as np

from pandas_profiling import ProfileReport
```


```python
#import cleaned data
df = pd.read_csv('cleaned_transformed_credit_one_data.csv')

#verify imported data looks as expected
df.head()
```

* Cleaned data Dimensions: 30,000 obervations & 25 variables

## Variable Info


```python
#get variable data types
df.dtypes
---
    ID            int64
    SEX          object
    EDUCATION    object
    MARRIAGE      int64
    AGE           int64
    LIMIT_BAL     int64
    bill_ap       int64
    bill_m        int64
    bill_ju       int64
    bill_jy       int64
    bill_ag       int64
    bill_s        int64
    pmt_ap        int64
    pmt_m         int64
    pmt_ju        int64
    pmt_jy        int64
    pmt_ag        int64
    pmt_s         int64
    pay_ap        int64
    pay_m         int64
    pay_ju        int64
    pay_jy        int64
    pay_ag        int64
    pay_s         int64
    DEFAULT      object
    dtype: object
```


1. Demographic Variables
    * Sex—categorical, 2 levels: male, female
    * Education— ordinal, 6 levels?
    * Marital Status - categorical, 4 levels
    * Age- continuous

2. Account Variables
    * ID- customer ID- unique number assigned to each observation in dataset (exclude from analysis)
    * Balance Limit - continuous
    * Default Status - categorical 2 level   
    * Repayment Status in April  - categorical, 9+ levels
    * Repayment Status in May
    * Repayment Status in June
    * Repayment Status in July
    * Repayment Status in August
    * Repayment Status in September

3. Billing Variables
    * April bill amount - continuous
    * May bill amount
    * June bill amount
    * July Bill amount
    * August bill amount
    * September bill amount

4. Payment Variables
    * April Payment amount- continuous
    * May Payment Amount
    * June Payment Amount
    * July Payment Amount
    * August Payment Amount
    * September Payment Amount

* Target Features
    * Balance Limit- if we are trying to predict how much credit customers should be extended.
    * Default- if we are trying to predict which customers are most likely to dafault on their loans.

# Univarite Analysis


```python
#create in-line minimal pandas profile report
#ProfileReport(df, minimal=True)

#profile = ProfileReport(df, title='Clean CreditOne Pandas Profiling Report', explorative = True)
```

*Coding Notes*
* The full pandas profile report took a *very long time to generate* and was *too big* to load without crashing my browser.
* Two options to generate profile reports more quickly:
    * Generate a minimal report (which does not include correlations between vars) by adding a 'minimal=True' parameter to ProfileReport command.
    * Generate a report for only a *sample* of df by substituting ```df.sample(n=10000)``` for df in the command.
* To output the report as a separate html file use:  ```prof = ProfileReport(df)
prof.to_file(output_file='output.html')```

## Demographic Variables

*Observations (based on profile report)*
* Sex
    * 60.4% female, 39.6% male
* Education
    * HS 16.4%, University 46.8%, Graduate School 35.3%, Other 1.6%
    * Compared to US population, sample has a lot of customers with university and graduate school degrees, and a small number with high school degrees.
    * May consider combining university and graduate school categories later.
* Marital Status
    * Married (1) 45.5%, Single (2) 53.6%, Divorced (3) 1.1%, Other 0.2%
* Age
    * Range 21 - 79.
    * Mean 35, Median 34
    * Consider binnning: young, middle age, 65+

## Account Variables

*Observations (based on profile report)*

* ***Credit/Balance Limit*** (target for model predicting amount of credit to give customers)
  * Continuous variable, range - 10k to 1 million
  * Only 81 distinct values (consider 8 bins)
  * Mean: 167,484
  * Distribution significantly skewed toward low end
  * Mode: 50k limit (11.2%)
  * Only 1 customer with 1 million limit  
  * Potential problem: this variable represents the amount of credit given to the customer, including both the individual consumer credit and "his/her family (supplemental) credit". I'm not exactly sure what family credit is, but it may be an amount of credit extended to a family group that the customer can use because he/she is part of the family. This potentially muddies the waters because  we don't know how much of the credit limit was given to the customer as an individual vs. how much of the credit limit they were given as a member of a family. At the very least this means, at best the model will be able to accurately predict the amount of individual and family credit each customer should be given.


* ***Default*** (target for model predicting which customers are likely to default)
  * Not defult 77.9%, default 22.1%
  * Do I need to use special techniques |when trying to model/predict extremely unlikely events?


* Repayment Variables
  * -2 = no use, -1 = paid in full, 0 = use revolving credit, 1 = payment delay 1 month, 2 = payment delay 2 months, ..., 9 = payment delay 9+ months
  * All repayment variables have roughly same distribution and characteristics: over 50% zeros, significantly more -2 & -1 values than values greater than or equal to 1

## Billing Variables

*Observations (from profile report)*  
* All billing variables have negative minimum values. If valid data points, it perhaps CreditOne is overcharging customers and having to issue credits.
* Distributions for all billing variables look roughly the same.
    * Very significant left skew.
    * Lots of zeros.
    * All months contain a small number of negative values. Don't know how to interpret.
        * ◘ Consider removing or transforming negative values to improve model performance.


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




*Observations*
* Suspicious data point:some customers have bill amounts that exceed their balance limits. Does that mean credit one is not enforcing those limits?
    * Maybe I should engineer a feature that is true iff the the customer's bill amount is ever bigger than their balance limit.


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

## Payment Variables

* There are 6 continuous payment variables - each represents the amount the customer paid towards their bill for the months of April to September 2005.

*Observations (based on profile report)*
* The distributions of the payment and billing variables have similar shapes.
* Notable Differences:
  * The payment variables contain a higher percentage of zero values.
  * The number and spread of positive values when it comes to the payment variables (which makes sense if customers are using revolving credit and/or not paying their bill.
  * The payment variables don't contain any negative values.

# Relationships Between Features (IVs)  
In this section, I will examine whether any of the features (independent variables) are related to one another in a significant way. If any of the features  are highly correlated, that may be because they are actually tracking/representing the same information. Since high corfelation between depedent variables can harm ML models, it needs to be handled.  

## Are billing variables correlated with each other?


```python
#calculate kendall's tau correlation between billing vars
df[ ['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s']].corr(method='kendall')
```



>*Coding Notes*
>* By default corr function calculates pearson's correlation co-efficient.
>* But it's only appropriate to use Pearson's if the data is normally distributed and we know from the profile report that the distributions for the billing variables are highly skewed.
>* I think there's a way to output only the values below the diagonal.


```python
# name a sample of 1,000 data points selected from the df
smpl=df.sample(n=1000)

#create pairwise plots of bill features using smpl
sns.pairplot(smpl, hue='DEFAULT', kind='scatter', vars=['bill_ap','bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s'])

#save plot as png
plt.savefig('creditone_bill_pairplot.png')
```


{{< figure src="/images/creditone_bill_pairplot.png" >}}

*Observations*
* It's reasonable to suspect the billing variables might be correlated, since it's not uncommon for individuals' spending to be similar from month to month.
* Yes, the billing variables are highly correlated with each other. Max: 0.8112, Min: 0.5853.
* In general, the correlation is highest between each monthly billing variable and the variable representing the month after. The correlation declines with each proceeding month (e.g. the correlation is highest between the April (bill_ap) and May (bill_m) billing variables, and declines for each subsequent month.)  
    * ◘ May impact model performance. Consider only using one of the billing variables in model.

## Are the payment variables correlated with each other?


```python
#look at kendall's tau correlations b/t payment variables
df[['pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s']].corr(method='kendall')
```






```python
#create a pairplot of the payment features using smpl (of 1,000)
sns.pairplot(smpl, hue='DEFAULT', kind='scatter', vars=['pmt_ap','pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s'])

#save plot as png
plt.savefig('creditone_pmt_pairplot.png')
```
{{< figure src="/images/creditone_pmt_pairplot.png" >}}

*Observtions*
* I wanted to look at whether the payment variables on related because I thought the amount customers pay each month  might be related to the amount they were billed during the previous month. If that's true and the billing variables are highly correlated, I would expect the payment variables to also be highly correlated.
* When we look at the kendall's correlation coefficients between the payment vars they are modest (between 0.348 and 0.4126).
* However, when we look at the scatter plots, there doesn't appear to be any obvious relationship.
* This suggests, somewhat surprising, that the amount customers pay each month is not really related to the amount they are billed.
  * I wonder if the lack of a clear relationship is because I need to look at the relationships between billing vars and payment vars offset by one month (e.g. the relationship between the bill amount for April and the payment amount for May, and so on).

## Are the payment variables correlated with the billing variables?


```python
#look at kendall's tau correlations b/t payment variables
df[['bill_ap', 'bill_m', 'bill_ju', 'bill_jy', 'bill_ag', 'bill_s','pmt_ap', 'pmt_m', 'pmt_ju', 'pmt_jy', 'pmt_ag', 'pmt_s']].corr(method='kendall')
```




*Observations*
* I expected to find a moderate to high correlation between the billing and payment variables, since presumably the amount customers pay each month is at least somewhat related to how much they are billed.
* More specifically, we should expect that the payment amount is most highly correlated with the bill amount of the previous month (e.g. the july payment amount is has the strongest correlation with the bill amount for June.
* The strongest correlations are around 0.51.

## Are the repayment variables related to the billing or payment variables?

The values in the repayment history variables should, in some sense, be related the values in the payment variables because the the repayment history variable is representing whether the customer has been using the credit, if they are paying their bills, and if their payments are for the full amount or leave a monthly balance (revolving credit).
* However, it's not clear how or what kind of relationship I can look for. One challenge is that the payment variables are continuous, and the repayment variables are categorical. It shouldf be the case that if a customer's monthly payment is not zero (i.e. they paid some amount) for some month, the value for their repayment history variable the next month should be -1 or zero.
* I have no idea how I could test for that.

# Relationships Between Feature & Target Variables

## Credit Limit

In this section I will perform statistical tests to determine whether any of the DVs are related to the limit_bal target variable.

### Demographic Variables & Credit Limit


```python
# sns.heatmap(data=df, annot=True, fmt="d", linewidths=.5)

"""
Above code won't work with non-numeric data
"""
```




    "\nAbove code won't work with non-numeric data\n"



### Sex & Credit Limit


```python
sns.boxplot(data=df, x='SEX',y='LIMIT_BAL' )

# save plot as png
plt.savefig("creditone_sex_credit_box.png")
```
{{< figure src="/images/creditone_sex_credit_box.png" >}}

The similarity between the box plots for female and male indicates there is no difference in credit limits between men and women. That means SEX is unlikely to be a helpful variable to include in a model prediting LIMIT_BAL.

### Education & Credit Limit


```python
sns.catplot(data=df, kind='box', x='EDUCATION', y='LIMIT_BAL').set_xticklabels(rotation=45)

# save plot as png
plt.savefig("creditone_edu_credit_box.png")
```
{{< figure src="/images/creditone_edu_credit_box.png" >}}

The box plots for all education levels are very similar. Customers with a graduate education enjoy credit limits that are on average higher than the other groups. It also appears the credit limits for customers with a high school education are slightly skewed to the lower end. The differences aren't dramatic, but a customer's credit limit does appear to be somewhat related to their level of education  

### Marriage & Credit Limit


```python
sns.boxplot(data=df, x='MARRIAGE', y='LIMIT_BAL')

# save plot as png
plt.savefig("creditone_marriage_credit_box.png")
```
{{< figure src="/images/creditone_marriage_credit_box.png" >}}

Although there are some small differences in the plots, it does not appear that a customer's credit limit is related to their marital status.

### Age & Credit Limit


```python
sns.scatterplot(data=df, x='AGE', y='LIMIT_BAL')

# save plot as png
plt.savefig("creditone_age_credit_box.png")
```
{{< figure src="/images/creditone_age_credit_box.png" >}}

There doesn't appear to be any sort of meaningful relationship between age and credit limit.

## Credit Default


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


### Demographic Features & Defualt Target

#### Sex & Default


```python
sns.displot(data=df, x="SEX", stat='density', shrink=.8)
```




    <seaborn.axisgrid.FacetGrid at 0x23c43023370>




```python
sns.displot(data=df, x="SEX", col='DEFAULT', stat='density', shrink=.8)

#save plot as png
plt.savefig('creditone_sex_default_notdefault.png')
```
{{< figure src="/images/creditone_sex_default_notdefault" >}}

*Observations*
* From the first plot, we can see that CreditOne's customers are slightly more likely to be female than male.
* From the second set of plots, when we compare the data for customers who have defualted on their loans to the data for the customers who have not defaulted on their loans, it looks like customers who have defaulted on their loans are slightly more likely to be male than customers overall--however, it's unlikely this difference is statistically significant.
* The fact that the relative probabilities of being female vs. male is almost the same among the customers who have defaulted and the customers who have not defualted suggests that sex will not be an important feature in a model predicting whether a customer will defualt on their loans.

#### Education & Default


```python
#create probability chart for the EDU var
sns.displot(data=df, x ="EDUCATION", discrete=True, shrink = .8, stat='probability').set_xticklabels(rotation=45)
```





```python
#create subplots showing the distribution for the EDU var conditional on the DEFUALT var
sns.displot(data=df, x ="EDUCATION", col = 'DEFAULT', shrink = .8, stat='probability').set_xticklabels(rotation=45)
```





*Observations*
* Since higher levels of education are highly correlated with higher levels of income, and higher levels of income are presumably correlated with lower rates of defaulting on loans. I was anticipating that customers with university and graduate school levels of education to be relatively less likely to default on their loans.
* However, comparing the distributions of education levels for customers who have defaulted to the ebb


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

#### Age & Default


```python
sns.catplot(data=df, kind='box', x='DEFAULT', y='AGE')

#save plot as png
plt.savefig('creditone_age_default.png')
```
{{< figure src="/images/creditone_age_default.png" >}}

*Observations*
* We can see in the above boxplot the the mean, IQR (middle 50% of data points), and range for the age variable are nearly identical for customers who defaulted on their loans (blue box) and for customers who did not defualt on their loans (orange box).
* This suggests there is no significant relationship between a customer's age and whether or not she defaulted on her loans, which means AGE will not be an important feature in a model predicting whether a customer will default.  

#### Marriage & Default


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




>*Coding Notes*
>* It look several hours to get the above row of plots.
>* Problems I had to solve:
  * Because the number of defualts is much smaller than not defaults, to compare sex across the two categories I couldn't use count as the y axis, I needed to plot a normalized stat like probability or density.
  * Using hue: When I tried to use hue to represent the DEFAULT categories and plot them on one set of axes, the resulting graph had 4 bars (2 for female, 2 male) whose probabilities summed to 1. Which meant the bars for male an female in the default category were much smaller than the bars for male and female in the not default category.
  * I also couldn


*Observations*
* From a comparison on the above plots, there appears that single customers are relatively less likely to default on their loans.

### Billing Variables & Credit Default


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




>*Coding Notes*
* Before finding the code to create the above 2x6 grid of subplots, I created the boxplots for each billing variable separately. For future refernce here is the code I used:  
  * ```sns.catplot(data=df, kind='box', x='DEFAULT', y='bill_ap')```
  * ```sns.catplot(data=df, kind='box', x='DEFAULT', y='bill_m', height=6)```
  * ```sns.catplot(data=df, kind='box', x='DEFAULT', y='bill_ju', height=6, aspect=.5)```
  * ```sns.catplot(data=df, kind='box', x='DEFAULT', y='bill_ag', height=8, aspect=.5)```
  * ```sns.catplot(data=df, kind='box', x='DEFAULT', y='bill_s', height=8, aspect=.5)```

*Observations*
* I have more questions than observations.
  * It looks like there are some significant outliers. How can we find those data points in our df.
  * Other than some differences in outliers and a noticable difference in the length of the wiskers in bill_ag, all the box plots are basically the same. What, if anything, can we conclude from that?

### Payment Variables & Default


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




*Observations*
* I assume the boxplots for the payment variables look like this because there is a high % of zeros in these variables.
* I also seem some signigicant outliers.


* Questions:
  * I don't have a good sense of how to interpret/understand what I'm seeing when I compare visualizations of DVs conditional on a target variable. Am I just looking for significant differences? What conclusions would you draw from seeing the above plots for the payment variables? Are there any rules of thumb?
  * How do you go about figuring out whether it would improve the situation to bin the values, remove outliers, aggregate the variables, etc? There are too many possibilities to test them all. How do you narrow things down? Is there a way to determine which changes are most likely to have an impact so you can focus your efforts?

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

#use get.dummies function to transform EDU var into 4 dummy vars (1 var for each category)
num_df=pd.get_dummies(num_df, columns=['EDUCATION'], prefix=['edu'])

num_df.head()
```


```python
#use get.dummies function to transform MAR var into 4 dummy vars (1 var for each category)
num_df=pd.get_dummies(num_df, columns=['MARRIAGE'], prefix=['mar'])
"""
The values in the MARRIAGE var were numerical to start. I had to transform it into 4 dummy variables
because I am going to use this data to train and test a regression ML model, and categorical vars are
treated as ordinal (i.e. the algo would treat/interpret category labeled 2 as the average of categories labeled 1 & 3)
"""
```





```python
#Export Numerical df
num_df.to_csv('num_only_credit_one_data.csv', index = False, header=True)
```

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


# Modeling

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
#the inline magic line is needed to make some data visualizations come out right when you are using jupyter notebooks
%matplotlib inline
import seaborn as sns

from math import sqrt
```


```python
#ML algorithms (estimators)
from sklearn.ensemble import RandomForestRegressor
from sklearn.linear_model import LinearRegression
from sklearn.svm import SVR
from sklearn import linear_model

#model metrics
from sklearn.metrics import mean_squared_error
from sklearn.metrics import r2_score
from sklearn.model_selection import cross_val_score

#cross validation
from sklearn.model_selection import train_test_split
```





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

# Predicting Customer Credit Limits
In this notebook I am only going to create and test models that attempt to predict customer credit limits. Which means the target variable in all models will be 'limit' (i.e. limit_bal).

#### Re-ordering df Columns
To make things easier, before doing anything I'm going to move LIMIT_BAL to the last column in my df and rename it.


```python
#step 1: create a variabled to hold the LIMIT_BAL var
lim=df['LIMIT_BAL']

#step 2: drop the LIMIT_BAL column
df.drop(labels='LIMIT_BAL', axis=1, inplace=True)

#step 3: insert 'lim' as last column of df
df.insert(loc=30, column='limit', value=lim)

df.head()
```




### Specify the Target Variable
Now I will specify the dependent or target variable for models attempting to predict customer credit limits.


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



# Model 1: Predicting Credit Limit Using All Variables
None of the variables stood out as ones that should be included or excluded from the model, so in this first model I'm going to include them all.

### Specify the Feature Variables


```python
#filter the data to return only columns for my feature vars
X = df.iloc[:, 1:30]

#verify that all but the ID and limit columns were selected
print('Summary of Feature Variables')
X.head()
```



## Train & Test Model 1 Using 3 Different Regression Algorithms
Because credit limit is a continuous variable, I know I will have to use a regression model to predict it. But I don't have a principled reason to choose one regression algorithm over others, so I'll compare the predictions of models using Random Forest p9idifferent regression algorithms to find the one that works best on this task.
* I will run each model through a 3-fold cross validation test. Using R squared to score the models.
* The model with the highest mean CV sore will be labeled best.

### Setup


```python
#create a dictionary to hold the algos
algosClass = [ ]

#add the randomn forest regressor to the dictionary
algosClass.append(('Random Forest Regressor',RandomForestRegressor()))

#add the liner regression algo to the dictionary
algosClass.append(('Linear Regression',LinearRegression()))

#add the SV regressor to the dictionary
algosClass.append(('Support Vector Regression',SVR()))

#verify every algo was added to the list
print(algosClass)
```

    [('Random Forest Regressor', RandomForestRegressor()), ('Linear Regression', LinearRegression()), ('Support Vector Regression', SVR())]


### Train & Test Models


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

### Performance Results


```python
#run the for loop and print the mean cross validation score for each model
for i in range(len(names)):
    print(names[i],results[i].mean())
```

    Random Forest Regressor 0.46700202592747336
    Linear Regression 0.35871695390197167
    Support Vector Regression -0.05039128320492573


The model built using the random forest regressor performed somewhat less badly than the other two models. However, even the randomn forest model does little better than chance at predicting customer's credit limits (i.e.limit_bal).

# Credit Limit Model 2: Excluding the Demographic Variables

## Select the Feature Variables


```python
#filter the data to return only columns for bill amount, payment amount, payment history, and default var
X2 = df.iloc[:, 3:22]

print('Summary of Model 2 Features')
X2.head()
```

    Summary of Features for Model 2





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
      <th>bill_ap</th>
      <th>bill_m</th>
      <th>bill_ju</th>
      <th>bill_jy</th>
      <th>bill_ag</th>
      <th>bill_s</th>
      <th>pmt_ap</th>
      <th>pmt_m</th>
      <th>pmt_ju</th>
      <th>pmt_jy</th>
      <th>pmt_ag</th>
      <th>pmt_s</th>
      <th>pay_ap</th>
      <th>pay_m</th>
      <th>pay_ju</th>
      <th>pay_jy</th>
      <th>pay_ag</th>
      <th>pay_s</th>
      <th>DEFAULT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>689</td>
      <td>3102</td>
      <td>3913</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>689</td>
      <td>0</td>
      <td>-2</td>
      <td>-2</td>
      <td>-1</td>
      <td>-1</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3261</td>
      <td>3455</td>
      <td>3272</td>
      <td>2682</td>
      <td>1725</td>
      <td>2682</td>
      <td>2000</td>
      <td>0</td>
      <td>1000</td>
      <td>1000</td>
      <td>1000</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>-1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15549</td>
      <td>14948</td>
      <td>14331</td>
      <td>13559</td>
      <td>14027</td>
      <td>29239</td>
      <td>5000</td>
      <td>1000</td>
      <td>1000</td>
      <td>1000</td>
      <td>1500</td>
      <td>1518</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29547</td>
      <td>28959</td>
      <td>28314</td>
      <td>49291</td>
      <td>48233</td>
      <td>46990</td>
      <td>1000</td>
      <td>1069</td>
      <td>1100</td>
      <td>1200</td>
      <td>2019</td>
      <td>2000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>19131</td>
      <td>19146</td>
      <td>20940</td>
      <td>35835</td>
      <td>5670</td>
      <td>8617</td>
      <td>679</td>
      <td>689</td>
      <td>9000</td>
      <td>10000</td>
      <td>36681</td>
      <td>2000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-1</td>
      <td>0</td>
      <td>-1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create two empty lists to hold the names and cross validation results from each model
results2 = []
names2 = []

#get cross validation scores for model 2
for name, model in algosClass:
    result = cross_val_score(model, X2,y, cv=3, scoring='r2')
    names2.append(name)
    results2.append(result)
```


```python
#run the for loop and print the mean cross validation score for each model
print("Summary of Model2 Cross Validation Scores")

for i in range(len(names2)):
    print(names[i],results[i].mean())
```

    Summary of Model2 Cross Validation Scores
    Random Forest Regressor 0.46700202592747336
    Linear Regression 0.35871695390197167
    Support Vector Regression -0.05039128320492573


## Train & Test the Model


```python
#instantiate the random forest regression algo
# rfr = RandomForestRegressor()
```


```python
#train the rfr model on X_train and y_train
# model = rfr.fit(X_train, y_train)

#use trained model to predict credit limit amounts for the X_test data
# y_predict = model_1.predict(X_test)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-15-966390a2adc9> in <module>
          1 #train the rfr model on X_train and y_train
    ----> 2 model = rfr.fit(X_train, y_train)
          3
          4 #use trained model to predict credit limit amounts for the X_test data
          5 y_predict = model_1.predict(X_test)


    NameError: name 'X_train' is not defined


## Split Data into Training and Testing Sets


```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2, random_state=123)
```

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


```python

```


[Link to GitHub Repository](https://github.com/kpiatti/creditone-project)
