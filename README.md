
# Managing Time Series Data - Lab

## Introduction

In the previous lecture, you learned that time series data are everywhere and understanding time series data is an important skill for data scientists!

In this lab, you'll practice your previously learned techniques to import/load, clean and manipulate time series data.

The lab will cover how to perform time series analysis while working with large datasets. The dataset can be memory intensive so your computer will need at least 2GB of memory to perform some of the calculations.


## Objectives

You will be able to:

* Load time-series data using Pandas and perform time series indexing
* Perform index based slicing to create subsets of a time-series
* Change the granularity of a time series 
* Perform basic data cleasing operations on a time series
* Getting time series data ready for further analysis



## Let's get started!

We will start the lab by loading the required libraries 

* `pandas` for data wrangling and manipulations  
* `matplotlib` for visualising timeseries data 
* `statsmodels` primarily for bundled datasets 


```python
# Load required libraries
import pandas as pd
import pandas.tseries
import statsmodels.api as sm
import matplotlib.pyplot as plt
```

## Loading time series data
The `StatsModels` library comes bundled with built-in datasets for experimentation and practice. A detailed description of these datasets can be found [here](http://www.statsmodels.org/dev/datasets/index.html). Using `StatsModels`, the time series datasets can be loaded straight into memory. 

In this lab, we'll use the **"Atmospheric CO2 from Continuous Air Samples at Mauna Loa Observatory, Hawaii, U.S.A."**, containing CO2 samples from March 1958 to December 2001. Further details on this dataset are available [here](http://www.statsmodels.org/dev/datasets/generated/co2.html).

We can bring in this data using the `load_pandas()`-method, which will allow us to read this data into a pandas dataframe by using `dataset.data`.  


```python
# Load the "co2" dataset from sm.datasets
CO2_dataset = sm.datasets.co2.load_pandas()
CO2 = CO2_dataset.data
```

Let's check the type of CO2 and also first 15 entries of CO2 dataframe as our first exploratory step.


```python
# Print the datatype of CO2 and check first 15 values

print(type(CO2))
print(CO2.head(15))

# datatype of CO2 is <class 'pandas.core.frame.DataFrame'>

#               co2
# 1958-03-29  316.1
# 1958-04-05  317.3
# 1958-04-12  317.6
# 1958-04-19  317.5
# 1958-04-26  316.4
# 1958-05-03  316.9
# 1958-05-10    NaN
# 1958-05-17  317.5
# 1958-05-24  317.9
# 1958-05-31    NaN
# 1958-06-07    NaN
# 1958-06-14    NaN
# 1958-06-21    NaN
# 1958-06-28    NaN
# 1958-07-05  315.8
```

    <class 'pandas.core.frame.DataFrame'>
                  co2
    1958-03-29  316.1
    1958-04-05  317.3
    1958-04-12  317.6
    1958-04-19  317.5
    1958-04-26  316.4
    1958-05-03  316.9
    1958-05-10    NaN
    1958-05-17  317.5
    1958-05-24  317.9
    1958-05-31    NaN
    1958-06-07    NaN
    1958-06-14    NaN
    1958-06-21    NaN
    1958-06-28    NaN
    1958-07-05  315.8


With all the required packages imported and the CO2 dataset as a Dataframe ready to go, we can move on to indexing our data.

## Data Indexing

You may have noticed that by default, the dates have been set as the index of our pandas DataFrame. While working with time series data in Python, it's important to always ensure that dates are used as index values and are set as a `timestamp` object. Timestamp is the pandas equivalent of python’s `Datetime` and is interchangeable with it in most cases. It’s the type used for the entries that make up a `DatetimeIndex`, and other timeseries oriented data structures in pandas. Further details can be found [here](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Timestamp.html).

We can confirm these assumption in python by checking index values of a pandas dataframe with `DataFrame.index`. 


```python
# Confirm that date values are used for indexing purpose in the CO2 dataset by using DataSet.index property.
print(CO2.index)


# DatetimeIndex(['1958-03-29', '1958-04-05', '1958-04-12', '1958-04-19',
#                '1958-04-26', '1958-05-03', '1958-05-10', '1958-05-17',
#                '1958-05-24', '1958-05-31',
#                ...
#                '2001-10-27', '2001-11-03', '2001-11-10', '2001-11-17',
#                '2001-11-24', '2001-12-01', '2001-12-08', '2001-12-15',
#                '2001-12-22', '2001-12-29'],
#               dtype='datetime64[ns]', length=2284, freq='W-SAT')
```

    DatetimeIndex(['1958-03-29', '1958-04-05', '1958-04-12', '1958-04-19',
                   '1958-04-26', '1958-05-03', '1958-05-10', '1958-05-17',
                   '1958-05-24', '1958-05-31',
                   ...
                   '2001-10-27', '2001-11-03', '2001-11-10', '2001-11-17',
                   '2001-11-24', '2001-12-01', '2001-12-08', '2001-12-15',
                   '2001-12-22', '2001-12-29'],
                  dtype='datetime64[ns]', length=2284, freq='W-SAT')


The output above shows that our dataset clearly fulfills the indexing requirements. Look at the last line:


### **dtype='datetime64[ns]', length=2284, freq='W-SAT'**


* `dtype=datetime[ns]` field confirms that the index is made of timestamp objects.
* `length=2284` shows the total number of entries in our timeseries data.
* `freq='W-SAT'` tells us that we have 2,284 weekly (W) date stamps starting on Saturdays (SAT).

Depending on the nature of analytical question, the resolution of timestamps can also be changed to hourly, daily, weekly, monthly, yearly etc. For our dataset, we can perform the analysis on monthly CO2 consumption values. This can be obtained by using the [`resample()` function](http://pandas.pydata.org/pandas-docs/stable/timeseries.html). Let's perform following data manipulations on our time series:

* Group the time-series into buckets representing 1 month using `resample()` function.
* Apply a `mean()`function on each group (i.e. get monthly average).
* Combine the result as one row per monthly group.


```python
# Group the timeseries into monthly buckets using resample('MS')
# 'MS' refers to monthly start 
# Take the mean of each group 
# get the first 10 elements of resulting timeseries

CO2_monthly= CO2['co2'].resample('MS')
CO2_monthly_mean = CO2_monthly.mean()

CO2_monthly_mean.head(10)

# 1958-03-01    316.100000
# 1958-04-01    317.200000
# 1958-05-01    317.433333
# 1958-06-01           NaN
# 1958-07-01    315.625000
# 1958-08-01    314.950000
# 1958-09-01    313.500000
# 1958-10-01           NaN
# 1958-11-01    313.425000
# 1958-12-01    314.700000
# Freq: MS, Name: co2, dtype: float64
```




    1958-03-01    316.100000
    1958-04-01    317.200000
    1958-05-01    317.433333
    1958-06-01           NaN
    1958-07-01    315.625000
    1958-08-01    314.950000
    1958-09-01    313.500000
    1958-10-01           NaN
    1958-11-01    313.425000
    1958-12-01    314.700000
    Freq: MS, Name: co2, dtype: float64



Looking at the index values, we can see that our timeseries now carries aggregated data on monthly terms, shown as `Freq: MS`. 

### Time-series Index Slicing for Data Selection

Pandas carries the ability to handle date stamp indices allowing quick and handy way of slicing data. For example, we can slice our dataset to only retrieve data points that come after the year 1990.


```python
# Slice the timeseries to contain data after year 1990. 

CO2_monthly_mean['1990':]

# 1990-01-01    353.650
# 1990-02-01    354.650
#                ...   
# 2001-11-01    369.375
# 2001-12-01    371.020
# Freq: MS, Name: co2, Length: 144, dtype: float64

```




    1990-01-01    353.650
    1990-02-01    354.650
    1990-03-01    355.480
    1990-04-01    356.175
    1990-05-01    357.075
    1990-06-01    356.080
    1990-07-01    354.675
    1990-08-01    352.900
    1990-09-01    350.940
    1990-10-01    351.225
    1990-11-01    352.700
    1990-12-01    354.140
    1991-01-01    354.675
    1991-02-01    355.650
    1991-03-01    357.200
    1991-04-01    358.600
    1991-05-01    359.250
    1991-06-01    358.180
    1991-07-01    356.050
    1991-08-01    353.860
    1991-09-01    352.125
    1991-10-01    352.250
    1991-11-01    353.740
    1991-12-01    355.025
    1992-01-01    355.900
    1992-02-01    356.680
    1992-03-01    357.900
    1992-04-01    359.075
    1992-05-01    359.540
    1992-06-01    359.125
                   ...   
    1999-07-01    369.000
    1999-08-01    366.700
    1999-09-01    364.675
    1999-10-01    365.140
    1999-11-01    366.650
    1999-12-01    367.900
    2000-01-01    369.020
    2000-02-01    369.375
    2000-03-01    370.400
    2000-04-01    371.540
    2000-05-01    371.650
    2000-06-01    371.625
    2000-07-01    369.940
    2000-08-01    367.950
    2000-09-01    366.540
    2000-10-01    366.725
    2000-11-01    368.125
    2000-12-01    369.440
    2001-01-01    370.175
    2001-02-01    371.325
    2001-03-01    372.060
    2001-04-01    372.775
    2001-05-01    373.800
    2001-06-01    373.060
    2001-07-01    371.300
    2001-08-01    369.425
    2001-09-01    367.880
    2001-10-01    368.050
    2001-11-01    369.375
    2001-12-01    371.020
    Freq: MS, Name: co2, Length: 144, dtype: float64



Similarly, we can also slice our timeseries to only retrieve values for a given time interval. Let's try to retrieve data starting from Jan 1990 to Jan 1991.


```python
# Retrieve the data between 1st Jan 1990 to 1st Jan 1991
CO2_monthly_mean['1990-01-01':'1991-01-01']

# 1990-01-01    353.650
# 1990-02-01    354.650
# 1990-03-01    355.480
# 1990-04-01    356.175
# 1990-05-01    357.075
# 1990-06-01    356.080
# 1990-07-01    354.675
# 1990-08-01    352.900
# 1990-09-01    350.940
# 1990-10-01    351.225
# 1990-11-01    352.700
# 1990-12-01    354.140
# 1991-01-01    354.675
# Freq: MS, Name: co2, dtype: float64
```




    1990-01-01    353.650
    1990-02-01    354.650
    1990-03-01    355.480
    1990-04-01    356.175
    1990-05-01    357.075
    1990-06-01    356.080
    1990-07-01    354.675
    1990-08-01    352.900
    1990-09-01    350.940
    1990-10-01    351.225
    1990-11-01    352.700
    1990-12-01    354.140
    1991-01-01    354.675
    Freq: MS, Name: co2, dtype: float64



### Missing Values

Its quite common for a timeseries dataset to have missing values as real world data tends to be messy and imperfect. Simplest way to detect missing values is either plotting the data and identifying disjoint areas of timeseries, or by using `DataFrame.isnull()` function to get list of all missing values. This function can be used with `sum()` to get a total count of all missing values. 


```python
# Get the total number of missing values in the timeseries
CO2_monthly_mean.isnull().sum()

# 5
```




    5



For the monthly timeseries we have created above, this value tells us that we are missing data values for 5 months in total. Missing values can be handled in a multitude of ways. 
* Drop the data elements with missing values (may result as low accuracy and loss of valueable information)
* Fill in the missing values under a defined criteria 
* Use advanced machine learning methods to predict the missing values. 

For our experiment, we shall try to fill in the missing values using pandas `DataFrame.fillna()` function. We shall use The `bfill()` method as an argument/crietria for filling in Null values . `bfill()` (backward filling) looks for the next valid entry in the timeseries and fills the gaps with this value. 


```python
# Use fillna() with bfill() argument to perform backward filling of missing values
# Use isnull() to check for missing values 

CO2_final = CO2_monthly_mean.fillna(CO2_monthly_mean.ffill())
CO2_final.isnull().sum()

# 0
```




    0



After performing above steps, out timeseries is now ready for visualisation and further analysis.

## Summary

In this introductory lab, we learnt how to create a time-series object in Python using Pandas. We learnt to fulfil all the requirements for a dataset to be classified as a time-series by ensuring  timestamp values as data index. Basic data handling techniques for getting time-series data ready for further analysis are also introduced. Students are advised to apply these techqniques to other time-series datasets as some may require much more cleansing than the one we saw in this lab. 