# Feature Engineering with Pyspark

{% embed url="https://datacamp-community-prod.s3.amazonaws.com/65076e3c-9df1-40d5-a0c2-36294d9a3ca9" %}

{% embed url="https://towardsdatascience.com/include-these-spark-window-functions-in-your-data-science-workflow-c6bac5824475" %}

{% embed url="https://towardsdatascience.com/the-hitchhikers-guide-to-handle-big-data-using-spark-90b9be0fe89a" %}

{% embed url="https://towardsdatascience.com/5-ways-to-add-a-new-column-in-a-pyspark-dataframe-4e75c2fd8c08" %}

{% embed url="https://towardsdatascience.com/100x-faster-randomized-hyperparameter-searching-framework-with-pyspark-4de19e44f5e6" %}



## EDA

Pyspark 更新很快，所以每次都要记得查版本，不然都不知道documentation不对死在哪里了！

```python
# return spark version
spark.version

# return python version
import sys 
sys.version_info

# JSON
spark.read.json('example.json') 
# CSV or delimited files
spark.read.csv('example.csv') 
# Parquet
spark.read.parquet('example.parq')

# print column names
print(df.columns)
```

在前期定义问题时，要知道自己数据的limitation，比如预测salescloseprice，如果只有一个district住宅楼的价格，无法预测更大面积；如果只有2017年的数据，无法判断时间演变规律。虽然parquet file定义好了自己的schema，但如果我们不是定义这个schema的人，最好自己检查一遍。

```python
df.count()
df.columns()
len(df.colomns)
df.dtypes

# Select our dependent variable
Y_df = df.select(['SALESCLOSEPRICE'])

# Display summary statistics
Y_df.describe().show()
```

### Verify Data Types

```python
def check_load(df, num_records, num_columns):
  # Takes a dataframe and compares record and column counts to input
  # Message to return if the critera below aren't met
  message = 'Validation Failed'
  # Check number of records
  if num_records == df.count():
    # Check number of columns
    if num_columns == len(df.columns):
      # Success message
      message = 'Validation Passed'
  return message

# Print the data validation message
print(check_load(df, 5000, 74))



# create list of actual dtypes to check
actual_dtypes_list = df.dtypes
print(actual_dtypes_list)

# Iterate through the list of actual dtypes tuples
for attribute_tuple in actual_dtypes_list:
  
  # Check if column name is dictionary of expected dtypes
  col_name = attribute_tuple[0]
  if col_name in validation_dict:

    # Compare attribute types
    col_type = attribute_tuple[1]
    if col_type == validation_dict[col_name]:
      print(col_name + ' has expected dtype.')
```

### Visually Inspect data

`df.describe(['LISTPRICE']).show()`观察某一个column的分布，除此之外spark自己也有内置的mean、skewness、minimum、covariance、correlation

```python
#Mean
pyspark.sql.functions.mean(col)
df.agg({'SALESCLOSEPRICE': 'mean'}).collect()
from pyspark.sql.functions import mean, stddev

#Skewness
pyspark.sql.functions.skewness(col)
from pyspark.sql.functions import skewness
print(df.agg({'LISTPRICE': 'skewness'}).collect())

#Minimum
pyspark.sql.functions.min(col)

#Covariance
cov(col1, col2)
df.cov('SALESCLOSEPRICE', 'YEARBUILT')

#Correlation
corr(col1, col2)

# Name and value of col with max corr
corr_max = 0
corr_max_col = columns[0]

# Loop to check all columns contained in list
for col in columns:
    # Check the correlation of a pair of columns
    corr_val = df.corr(col, 'SALESCLOSEPRICE')
    # Logic to compare corr_max with current corr_val
    if corr_max < corr_val:
        # Update the column name and corr value
        corr_max = corr_val
        corr_max_col = col

print(corr_max_col)
```

如果想要用seaborn之类的包画图，要转成pands dataframe，所以要注意先做sampling，sample with replacement

```python
# Sample 50% of the PySpark DataFrame and count rows
df.sample(False, 0.5, 42).count()

import seaborn as sns
# Sample the dataframe
sample_df = df.select(['SALESCLOSEPRICE']).sample(False, 0.5, 42) # Convert the sample to a Pandas DataFrame
pandas_df = sample_df.toPandas()
# Plot it
sns.distplot(pandas_df)
```

还可以用lmplot\(\) 来画两个column name的linear model plot. If they do they are good candidates to include in our analysis. If they don't it doesn't mean that we should throw them out, it means we may have to process or wrangle them before they can be used.

```python
import seaborn as sns
# Select columns
s_df = df.select(['SALESCLOSEPRICE', 'SQFTABOVEGROUND']) # Sample dataframe
s_df = s_df.sample(False, 0.5, 42)
# Convert to Pandas DataFrame
pandas_df = s_df.toPandas()
# Plot it
sns.lmplot(x='SQFTABOVEGROUND', y='SALESCLOSEPRICE', data=pandas_df)
```

还可以用joint plot画R2，知道两个变量之间的关系。

![](https://cdn.mathpix.com/snip/images/9FwLbK3ZvDPkNdfffe35UCBXk7UW2N-QgFsT4ozD18Y.original.fullsize.png)

```python
# Create new feature by adding two features together
df = df.withColumn('Total_SQFT', df['SQFTBELOWGROUND'] + df['SQFTABOVEGROUND'])

# Create additional new feature using previously created feature
df = df.withColumn('BATHS_PER_1000SQFT', df['BATHSTOTAL'] / (df['Total_SQFT'] / 1000))
df[['BATHS_PER_1000SQFT']].describe().show()

# Pandas dataframe
pandas_df = df.sample(False, 0.5, 0).toPandas()

# Linear model plots
sns.jointplot(x='Total_SQFT', y='SALESCLOSEPRICE', data=pandas_df, kind="reg", stat_func=r2)
plt.show()
sns.jointplot(x='BATHS_PER_1000SQFT', y='SALESCLOSEPRICE', data=pandas_df, kind="reg", stat_func=r2)
plt.show()
```

![](https://cdn.mathpix.com/snip/images/I7iKcllVRltizRjOp0AOEKuT9kP0gxaORthPdjurvZw.original.fullsize.png)

还有上图这种叫countplot

```python
# Import needed functions
from pyspark.sql.functions import to_date, dayofweek

# Convert to date type
df = df.withColumn('LISTDATE', to_date('LISTDATE'))

# Get the day of the week
df = df.withColumn('List_Day_of_Week', dayofweek('LISTDATE'))

# Sample and convert to pandas dataframe
sample_df = df.sample(False, 0.5, 42).toPandas()

# Plot count plot of of day of week
sns.countplot(x="List_Day_of_Week", data=sample_df)
plt.show()
```

## Dropping Data

Where can data go bad?

* Recorded wrong 
* Unique events 
* Formatted incorrectly 
* Duplications
* Missing
* Not relevant

For this dataset, 'NO' auto-generated record number, 'UNITNUMBER' irrelevant data, 'CLASS' all constant. spark的dropna和pandas df的很像，except有个\*。代表unpack the list and apply to them all。

```python
# List of columns to drop
cols_to_drop = ['NO', 'UNITNUMBER', 'CLASS']
# Drop the columns
df = df.drop(*cols_to_drop)
```

也可以

### Text Filtering

`isin()`  is similar to `like()` but allows us to pass a list of values to use as a filter rather than a single one.

```python
df = df.where(~df['POTENTIALSHORTSALE'].like('Not Disclosed'))
```

```python
# Inspect unique values in the column 'ASSUMABLEMORTGAGE'
df.select(['ASSUMABLEMORTGAGE']).distinct().show()

# List of possible values containing 'yes'
yes_values = ['Yes w/ Qualifying', 'Yes w/No Qualifying']

# Filter the text values out of df but keep null values
text_filter = ~df['ASSUMABLEMORTGAGE'].isin(yes_values) | df['ASSUMABLEMORTGAGE'].isNull()
df = df.where(text_filter)

# Print count of remaining records
print(df.count())
```

### Outlier Filtering

![](https://cdn.mathpix.com/snip/images/y64_Kv8lcRElHBVmUOxwcKrrG_Kb1KUQCCWdASXZtis.original.fullsize.png)

```python
from pyspark.sql.functions import mean, stddev
# Calculate values used for filtering
std_val = df.agg({'SALESCLOSEPRICE': 'stddev'}).collect()[0][0] 
mean_val = df.agg({'SALESCLOSEPRICE': 'mean'}).collect()[0][0]
# Create three standard deviation (? ± 3?) upper and lower bounds for data
hi_bound = mean_val + (3 * std_val) 
low_bound = mean_val - (3 * std_val)
# Use where() to filter the DataFrame between values
df = df.where((df['LISTPRICE'] < hi_bound) & (df['LISTPRICE'] > low_bound))
```

### Drop NA / NULL

`DataFrame.dropna()`  
 `how` :‘any’or‘all’.If ‘any’,drop a record if it contains any nulls.  If ‘all’, drop a record only if all its values are null.  
`thresh` :int, default `None`. If speci ed, drop records that have less than thresh non-null values. This overwrites the how parameter.  
`subset` :optional list of column names to consider.

```python
# Drop any records with NULL values
df = df.dropna()
# drop records if both LISTPRICE and SALESCLOSEPRICE are NULL
df = df.dropna(how='all', subset['LISTPRICE', 'SALESCLOSEPRICE '])
# Drop records where at least two columns have NULL values
df = df.dropna(thresh=2)
```

### Drop Duplicates

因为spark的分布式存储，这个操作不一定drop第一个出现的dup。

```python
# Entire DataFrame
df.dropDuplicates()
# Check only a column list
df.dropDuplicates(['streetaddress'])
```

## Adjusting Data

"Data does not give up its secrets easily, it must be tortured to confess."——Jeff Hooper, Bell Labs.

### Min-Max Scaling

$$
x_{i, j}^{*}=\frac{x_{i, j}-x_{j}^{\min }}{x_{j}^{\max }-x_{j}^{\min }}
$$

![](https://cdn.mathpix.com/snip/images/Mhhn7JTFJpfae9Csk3FhyErN7kjXG__Ip8JEeTjazSg.original.fullsize.png)

```python
# define min and max values and collect them
max_days = df.agg({'DAYSONMARKET': 'max'}).collect()[0][0] min_days = df.agg({'DAYSONMARKET': 'min'}).collect()[0][0]
# create a new column based off the scaled data
df = df.withColumn("scaled_days",
(df['DAYSONMARKET'] - min_days) / (max_days - min_days))
df[['scaled_days']].show(5)
```

写一个wrapper

```python
def min_max_scaler(df, cols_to_scale):
  # Takes a dataframe and list of columns to minmax scale. Returns a dataframe.
  for col in cols_to_scale:
    # Define min and max values and collect them
    max_days = df.agg({col: 'max'}).collect()[0][0]
    min_days = df.agg({col: 'min'}).collect()[0][0]
    new_column_name = 'scaled_' + col
    # Create a new column based off the scaled data
    df = df.withColumn(new_column_name, 
                      (df[col] - min_days) / (max_days - min_days))
  return df
  
df = min_max_scaler(df, cols_to_scale)
# Show that our data is now between 0 and 1
df[['DAYSONMARKET', 'scaled_DAYSONMARKET']].show()
```

### Standardization - Z transform

![](https://cdn.mathpix.com/snip/images/8sy6PGC5nktbCwAQeibMwurSCO1yumRPYCLtXWTwrb8.original.fullsize.png)

```python
mean_days = df.agg({'DAYSONMARKET': 'mean'}).collect()[0][0] stddev_days = df.agg({'DAYSONMARKET': 'stddev'}).collect()[0][0]
# Create a new column with the scaled data
df = df.withColumn("ztrans_days",
(df['DAYSONMARKET'] - mean_days) / stddev_days)
df.agg({'ztrans_days': 'mean'}).collect()
df.agg({'ztrans_days': 'stddev'}).collect()
```

### Log Scaling

下图是典型的positive skewed

![](https://cdn.mathpix.com/snip/images/4IdUnK5AWmz6W3MlGotsYwAREjb5Kr_YFRLVXkv7oqo.original.fullsize.png)

```python
# import the log function
from pyspark.sql.functions import log
# Recalculate log of SALESCLOSEPRICE
df = df.withColumn('log_SalesClosePrice', log(df['SALESCLOSEPRICE']))
```

### User Defined Scaling

比如对于房价，也可以用百分比来表示它在市场的时间。

```python
# Define max and min values and collect them
max_days = df.agg({'DAYSONMARKET': 'max'}).collect()[0][0]
min_days = df.agg({'DAYSONMARKET': 'min'}).collect()[0][0]

# Create a new column based off the scaled data
df = df.withColumn('percentagesscaleddays', 
                  round((df['DAYSONMARKET'] - min_days) / (max_days - min_days)) * 100)

# Calc max and min for new column
print(df.agg({'percentagesscaleddays': 'max'}).show())
print(df.agg({'percentagesscaleddays': 'min'}).show())
```

用log后的值反映它的建成时间

```python
from pyspark.sql.functions import log

# Compute the skewness
print(df.agg({'YEARBUILT': 'skewness'}).collect())

# Calculate the max year
max_year = df.agg({'YEARBUILT': 'max'}).collect()[0][0]

# Create a new column of reflected data
df = df.withColumn('Reflect_YearBuilt', (max_year + 1) - df['YEARBUILT'])

# Create a new column based reflected data
df = df.withColumn('adj_yearbuilt', 1 / log(df['Reflect_YearBuilt']))
```

## Missing Value

How does data go missing in the digital age?

* Data Collection - Broken Sensors 
* Data Storage Rules - 2017-01-01 vs January 1st, 2017 
* Joining Disparate Data Monthly to Weekly 
* Intentionally Missing - Privacy Concerns

### Types of Missing 

Missing completely at random

* Missing Data is just a completely random subset

Missing at random

* Missing conditionally at random based on another observation

Missing not at random

* Data is missing because of how it is collected

### When to drop rows with missing data?

* Missing values are rare
* Missing Completely at Random

`isNull()    df.where(df['ROOF'].isNull()).count()`

True if the current expression is null.

{% code title="超过某个阈值的时候drop" %}
```python
def column_dropper(df, threshold):
  # Takes a dataframe and threshold for missing values. Returns a dataframe.
  total_records = df.count()
  for col in df.columns:
    # Calculate the percentage of missing values
    missing = df.where(df[col].isNull()).count()
    missing_percent = missing / total_records
    # Drop column if percent of missing is more than threshold
    if missing_percent > threshold:
      df = df.drop(col)
  return df

# Drop columns that are more than 60% missing
df = column_dropper(df, 0.6)
```
{% endcode %}

### Plotting Missing Values

```python
# Import library
import seaborn as sns # subset the dataframe
sub_df = df.select(['ROOMAREA1']) # sample the dataframe
sample_df = sub_df.sample(False, .5, 4) # Convert to Pandas DataFrame
pandas_df = sample_df.toPandas()
# Plot it
sns.heatmap(data=pandas_df.isnull())  #convert it DataFrame into True/False
```

![](https://cdn.mathpix.com/snip/images/fpzHq2uTrHG0BKqZnT6JpRITZLAj1BihIPpgQtkabb8.original.fullsize.png)

比如这张图里，可以看出来第二个column全都是missing

### Imputation of Missing Value

Process of replacing missing values 

Rule Based: Value based on business logic 

Statistics Based: Using mean, median, etc 

Model Based: Use model to predict value

### Imputation of Missing Values

\*\* `fillna(value, subset=None)`  
 `value` the value to replace missings with  
`subset` the list of column names to replace missings 

```python
# Replacing missing values with zero
df.fillna(0, subset=['DAYSONMARKET'])
# Replacing with the mean value for that column
col_mean = df.agg({'DAYSONMARKET': 'mean'}).collect()[0][0] 
df.fillna(col_mean, subset=['DAYSONMARKET'])
```

## Getting more data

### External DataSource

Thoughts on External Data Sets

| Pros | Cons |
| :--- | :--- |
| Add important predictors  | May 'bog' analysis down  |
| Supplement/replace values  | Easy to induce data leakage  |
| Cheap or easy to obtain | Become data set subject matter expert |

### Join - PySpark DataFrame

```python
DataFrame.join( other,
on=None, how=None)
# Other DataFrame to merge
# The keys to join on
# Type of join to perform (default is 'inner')

# Inspect dataframe head
hdf.show(2)
# Specify join conditon
cond = [df['OFFMARKETDATE'] == hdf['dt']]
# Join two hdf onto df
df = df.join(hdf, on=cond, 'left')
# How many sales occurred on bank holidays? df.where(~df['nm'].isNull()).count()
```

### Join - SparkSQL Join

```python
# Register the dataframe as a temp table
df.createOrReplaceTempView("df") 
hdf.createOrReplaceTempView("hdf")

# Write a SQL Statement
sql_df = spark.sql(""" SELECT
* FROM df
LEFT JOIN hdf
ON df.OFFMARKETDATE = hdf.dt """)
```

要注意的时候Join的时候需要注意数字的精确度，因为如果join的是longitude和latitude，那么精读不同会导致join完全不行

### Generate More Features

可以通过对已有feature的加减乘除take ratio、取平均等方式来增加新的feature，但是有时候这样造出来的feature可能也不会有什么好的结果。

Automation of Features: FeatureTools & TSFresh

## Date Time

### Date

Please keep in mind that PySpark's week starts on Sunday, with a value of 1 and ends on Saturday, a value of 7.

```python
from pyspark.sql.functions import to_date 
# Cast the data type to Date
df = df.withColumn('LISTDATE', to_date('LISTDATE'))

# Inspect the field
df[['LISTDATE']].show(2)

from pyspark.sql.functions import year, month 
# Create a new column of year number
df = df.withColumn('LIST_YEAR', year('LISTDATE')) 
# Create a new column of month number
df = df.withColumn('LIST_MONTH', month('LISTDATE'))

from pyspark.sql.functions import dayofmonth, weekofyear
# Create new columns of the day number within the month
df = df.withColumn('LIST_DAYOFMONTH', dayofmonth('LISTDATE'))
# Create new columns of the week number within the year
df = df.withColumn('LIST_WEEKOFYEAR', weekofyear('LISTDATE')) 

 from pyspark.sql.functions import datediff
# Calculate difference between two date fields
df.withColumn('DAYSONMARKET', datediff('OFFMARKETDATE', 'LISTDATE'))
```

### Lagging Features

`window()`Returns a record based off a group of records

`lag(col, count=1)`Returns the value that is offset by rows before the current row

```python
from pyspark.sql.functions import lag 
from pyspark.sql.window import Window
# Create Window
w = Window().orderBy(m_df['DATE']) 
# Create lagged column
m_df = m_df.withColumn('MORTGAGE-1wk', lag('MORTGAGE', count=1).over(w)) 

# Inspect results
m_df.show(3)
```

## Extract Features

### Extract with Text Match 

![](../.gitbook/assets/image%20%2840%29%20%281%29.png)

```python
from pyspark.sql.functions import when
# Create boolean filters
find_under_8 = df['ROOF'].like('%Age 8 Years or Less%') 
find_over_8 = df['ROOF'].like('%Age Over 8 Years%')
# Apply filters using when() and otherwise()
df = df.withColumn('old_roof', (when(find_over_8, 1) 
                                        .when(find_under_8, 0)
                                        .otherwise(None))) 

df[['ROOF', 'old_roof']].show(3, truncate=100)
```

### Split Columns

![](../.gitbook/assets/image%20%2869%29.png)

```python
from pyspark.sql.functions import split 
# Split the column on commas into a list
split_col = split(df['ROOF'], ',')
# Put the first value of the list into a new column
df = df.withColumn('Roof_Material', split_col.getItem(0)) 
# Inspect results
df[['ROOF', 'Roof_Material']].show(5, truncate=100)
```

### Explode

![](../.gitbook/assets/image%20%2864%29.png)

### Pivot

![](../.gitbook/assets/image%20%2853%29.png)

```python
# Explode & Pivot!

from pyspark.sql.functions import split, explode, lit, coalesce, first 

# Split the column on commas into a list
df = df.withColumn('roof_list', split(df['ROOF'], ', ')) 

# Explode list into new records for each value
ex_df = df.withColumn('ex_roof_list', explode(df['roof_list'])) 

# Create a dummy column of constant value
ex_df = ex_df.withColumn('constant_val', lit(1))

# Pivot the values into boolean columns
piv_df = ex_df.groupBy('NO').pivot('ex_roof_list')\ 
.agg(coalesce(first('constant_val')))
```

接下来，join回到原来的df

```python
# Pivot 
piv_df = ex_df.groupBy('NO').pivot('ex_garage_list').agg(coalesce(first('constant_val')))

# Join the dataframes together and fill null
joined_df = df.join(piv_df, on='NO', how='left')

# Columns to zero fill
zfill_cols = piv_df.columns

# Zero fill the pivoted values
zfilled_df = joined_df.fillna(0, subset=zfill_cols)
```

## Binarizing, Bucketing & Encoding

### Binarizing

把threshold以上的定为1，小于等于的0 注意被binarize的column的datatype需要是double

```python
from pyspark.ml.feature import Binarizer # Cast the data type to double
df = df.withColumn('FIREPLACES', df['FIREPLACES'].cast('double'))
# Create binarizing transformer
bin = Binarizer(threshold=0.0, inputCol='FIREPLACES', outputCol='FireplaceT') 
# Apply the transformer
df = bin.transform(df)
# Inspect the results
df[['FIREPLACES','FireplaceT']].show(3)
```

### Bucketing

设置一批区间，把数据bin在一起 比如当看到dist（）里后面的数据比较少，就可以考虑合并掉它们。

```python
from pyspark.ml.feature import Bucketizer # Define how to split data
splits = [0, 1, 2, 3, 4, float('Inf')]
# Create bucketing transformer
buck = Bucketizer(splits=splits, inputCol='BATHSTOTAL', outputCol='baths') 
# Apply transformer
df = buck.transform(df)
# Inspect results
df[['BATHSTOTAL', 'baths']].show(4)
```

![](../.gitbook/assets/image%20%2825%29.png)

### One Hot Encoding

Spark的onehotencoding分成两步，先用stringindexer标号，再用encoder来transform。

```python
from pyspark.ml.feature import OneHotEncoder, StringIndexer 
# Create indexer transformer
stringIndexer = StringIndexer(inputCol='CITY', outputCol='City_Index')
# Fit transformer
model = stringIndexer.fit(df)
# Apply transformer
indexed = model.transform(df)

# Create encoder transformer
encoder = OneHotEncoder(inputCol='City_Index', outputCol='City_Vec') 
# Apply the encoder transformer
encoded_df = encoder.transform(indexed)
# Inspect results
encoded_df[['City_Vec']].show(4)
```

## Choose Model

![](../.gitbook/assets/image%20%2830%29.png)

`ml.regression`

* `DecisionTreeRegression` 
* `GBTRegression` 
* `RandomForestRegression`
* `GeneralizedLinearRegression` 
* `IsotonicRegression` 
* `LinearRegression`

### Time Series Test and Train Splits

![](../.gitbook/assets/image%20%2815%29.png)

```python
# Create variables for max and min dates in our dataset
max_date = df.agg({'OFFMKTDATE': 'max'}).collect()[0][0] 
min_date = df.agg({'OFFMKTDATE': 'min'}).collect()[0][0]

# Find how many days our data spans
from pyspark.sql.functions import datediff 
range_in_days = datediff(max_date, min_date)

# Find the date to split the dataset on
from pyspark.sql.functions import date_add 
split_in_days = round(range_in_days * 0.8) 
split_date = date_add(min_date, split_in_days)

# Split the data into 80% train, 20% test
train_df = df.where(df['OFFMKTDATE'] < split_date) 
test_df = df.where(df['OFFMKTDATE'] >= split_date)\
.where(df['LISTDATE'] >= split_date)
```

写个wrapper

```python
def train_test_split_date(df, split_col, test_days=45):
  """Calculate the date to split test and training sets"""
  # Find how many days our data spans
  max_date = df.agg({split_col: 'max'}).collect()[0][0]
  min_date = df.agg({split_col: 'min'}).collect()[0][0]
  # Subtract an integer number of days from the last date in dataset
  split_date = max_date - timedelta(days=test_days)
  return split_date

# Find the date to use in spitting test and train
split_date = train_test_split_date(df, 'OFFMKTDATE')

# Create Sequential Test and Training Sets
train_df = df.where(df['OFFMKTDATE'] < split_date) 
test_df = df.where(df['OFFMKTDATE'] >= split_date).where(df['LISTDATE'] <= split_date) 
```

### Time Series Data Leakage

Data leakage will cause your model to have very optimistic metrics for accuracy but once real data is run through it the results are often very disappointing. 

`DAYSONMARKET` only reflects what information we have at the time of predicting the value. I.e., if the house is still on the market, we don't know how many more days it will stay on the market. We need to adjust our `test_df` to reflect what information we currently have as of 2017-12-10. 

NOTE: This example will use the `lit()` function. This function is used to allow single values where an entire column is expected in a function call.

Thinking critically about what information would be available at the time of prediction is crucial in having accurate model metrics and saves a lot of embarrassment down the road if decisions are being made based off your results!

```python
from pyspark.sql.functions import lit, datediff, to_date

split_date = to_date(lit('2017-12-10'))
# Create Sequential Test set
test_df = df.where(df['OFFMKTDATE'] >= split_date).where(df['LISTDATE'] <= split_date)

# Create a copy of DAYSONMARKET to review later
test_df = test_df.withColumn('DAYSONMARKET_Original', test_df['DAYSONMARKET'])

# Recalculate DAYSONMARKET from what we know on our split date
test_df = test_df.withColumn('DAYSONMARKET', datediff(split_date, 'LISTDATE'))

# Review the difference
test_df[['LISTDATE', 'OFFMKTDATE', 'DAYSONMARKET_Original', 'DAYSONMARKET']].show()
```

### Dataframe Columns to Feature Vectors

```python
from pyspark.ml.feature import VectorAssembler 
# Replace Missing values
df = df.fillna(-1)
# Define the columns to be converted to vectors
features_cols = list(df.columns)
# Remove the dependent variable from the list
features_cols.remove('SALESCLOSEPRICE')

# Create the vector assembler transformer
vec = VectorAssembler(inputCols=features_cols, outputCol='features') 
# Apply the vector transformer to data
df = vec.transform(df)
# Select only the feature vectors and the dependent variable
ml_ready_df = df.select(['SALESCLOSEPRICE', 'features']) 
# Inspect Results
ml_ready_df.show(5)
```

### Drop Columns with Low Observations

一般低于30就不要了，因为不具有统计意义。Removing low observation features is helpful in many ways. It can improve processing speed of model training, prevent overfitting by coincidence and help interpretability by reducing the number of things to consider.

```python
obs_threshold = 30
cols_to_remove = list()
# Inspect first 10 binary columns in list
for col in binary_cols[0:10]:
  # Count the number of 1 values in the binary column
  obs_count = df.agg({col:'sum'}).collect()[0][0]
  # If less than our observation threshold, remove
  if obs_count <= obs_threshold:
    cols_to_remove.append(col)
    
# Drop columns and print starting and ending dataframe shapes
new_df = df.drop(*cols_to_remove)

print('Rows: ' + str(df.count()) + ' Columns: ' + str(len(df.columns)))
print('Rows: ' + str(new_df.count()) + ' Columns: ' + str(len(new_df.columns)))
```

### Random Forest, Naively Handling Missing and Categorical Values

对missing value友好，不需要minmax scale，对skewness不敏感，不需要onehotencod。 Missing values are handled by Random Forests internally where they partition on missing values. As long as you replace them with something outside of the range of normal values, they will be handled correctly. 

Likewise, categorical features only need to be mapped to numbers, they are fine to stay all in one column by using a StringIndexer as we saw in chapter 3. OneHot encoding which converts each possible value to its own boolean feature is not needed.

```python
# Replace missing values
df = df.fillna(-1, subset=['WALKSCORE', 'BIKESCORE'])

# Create list of StringIndexers using list comprehension
indexers = [StringIndexer(inputCol=column, outputCol=column+"_IDX")\
            .setHandleInvalid("keep") for column in categorical_cols]
# Create pipeline of indexers
indexer_pipeline = Pipeline(stages=indexers)
# Fit and Transform the pipeline to the original data
df_indexed = indexer_pipeline.fit(df).transform(df)

# Clean up redundant columns
df_indexed = df_indexed.drop(*categorical_cols)
# Inspect data transformations
print(df_indexed.dtypes)
```

## Building Model

### Training /Predicting with a Random Forest

```python
from pyspark.ml.regression import RandomForestRegressor
# Initialize model with columns to utilize
 rf = RandomForestRegressor(featuresCol="features",
                            labelCol="SALESCLOSEPRICE",
                            predictionCol="Prediction_Price",
                            seed=42
                           )
# Train model
model = rf.fit(train_df)

# Make predictions
predictions = model.transform(test_df)
# Inspect results
predictions.select("Prediction_Price", "SALESCLOSEPRICE").show(5)
```

```python
from pyspark.ml.regression import GBTRegressor

# Train a Gradient Boosted Trees (GBT) model.
gbt = GBTRegressor(featuresCol='features',
                           labelCol="SALESCLOSEPRICE",
                           predictionCol="Prediction_Price",
                           seed=42
                           )

# Train model.
model = gbt.fit(train_df)
```

### Evaluate a Model 

```python
from pyspark.ml.evaluation import RegressionEvaluator
# Select columns to compute test error
evaluator = RegressionEvaluator(labelCol="SALESCLOSEPRICE", predictionCol="Prediction_Price")

# Create evaluation metrics
rmse = evaluator.evaluate(predictions, {evaluator.metricName: "rmse"}) 
r2 = evaluator.evaluate(predictions, {evaluator.metricName: "r2"})

# Print Model Metrics
print('RMSE: ' + str(rmse)) print('R^2: ' + str(r2))
```

写个wrapper

```python
from pyspark.ml.evaluation import RegressionEvaluator

# Select columns to compute test error
evaluator = RegressionEvaluator(labelcol='SALESCLOSEPRICE', 
                                predictionCol='Prediction_Price')
# Dictionary of model predictions to loop over
models = {'Gradient Boosted Trees': gbt_predictions, 'Random Forest Regression': rfr_predictions}
for key, preds in models.items():
  # Create evaluation metrics
  rmse = evaluator.evaluate(preds, {evaluator.metricName: 'rmse'})
  r2 = evaluator.evaluate(preds, {evaluator.metricName: 'r2'})
  
  # Print Model Metrics
  print(key + ' RMSE: ' + str(rmse))
  print(key + ' R^2: ' + str(r2))
```

R^2 is comparable across predictions regardless of dependent variable.

RMSE is comparable across predictions looking at the same dependent variable. 

RMSE is a measure of unexplained variance in the dependent variable.

### Interpret a Model 

```python
import pandas as pd
# Convert feature importances to a pandas column
fi_df = pd.DataFrame(model.featureImportances.toArray(), columns=['importance'])

# Convert list of feature names to pandas column
fi_df['feature'] = pd.Series(feature_cols)

# Sort the data based on feature importance
fi_df.sort_values(by=['importance'], ascending=False, inplace=True)

# Interpret results
fi_df.head(9)
```

### Save and Load the Model

```python
from pyspark.ml.regression import RandomForestRegressionModel

# Save model
model.save('rfr_no_listprice')

# Load model
loaded_model = RandomForestRegressionModel.load('rfr_no_listprice')
```

