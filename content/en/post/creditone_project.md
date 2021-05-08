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

{{< figure src="/images/gen_spending_plot.png" title="Data Vizualization" >}}

[Link to GitHub Repository](https://github.com/kpiatti/creditone-project)
