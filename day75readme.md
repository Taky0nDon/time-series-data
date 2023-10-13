---
tags:
  - pandas
  - resample
  - isna
  - time-series
  - data
---
## Combine google Trends with other Time Series Data

What can the poularity of search terms tel us about the wrold? Google Trends gives us access to the popularity of Google Search terms. Let's investigate:

* How search volume for "Bitcoin" relates to the price of Bitcoin
* How search volume for a hot stock like Telsa relates to that stock's price
* How searches for "unemployment Benefits" vary with the actual unemployment rate in the United States*

## What we'll learn today

* How to make time-series data comprable by resampling and converting to the same periodcity (e.g., from daily data to monthly data)
* Fine-tuning the styling of Matplotlib charts by using limits, labels, line-styles, markers, colors, and the chart's resolution
* Using grids to help visually identify seasonality in a time series
* Finding the number of missing and NaN values and how to locate NaN values in a DataFrame
* How to work with Locators to better style the time axis on a chart.
* Review the concepts learned in the previous three days and apply them to new datasets

## Data Exploration

```python
df_tesla = pd.read_csv('TESLA Search Trend vs Price.csv')

df_btc_search = pd.read_csv('Bitcoin Search Trend.csv')
df_btc_price = pd.read_csv('Daily Bitcoin Price.csv')

df_unemployment = pd.read_csv('UE Benefits Search vs UE Rate 2004-19.csv')

dataframes = [df_tesla, df_btc_search, df_btc_price, df_unemployment]
```

1. What are the shapes of the dataframes?
	df_tesla: (124, 3)
	df_btc_search: (73, 2)
	df_btc_price: (2204, 3)
	df_unemployment(181, 3)

2. How many rows & columns do they have?
3. What are the column names?
4. What is the largest number in the search data column? Try using the [`.describe()`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.describe.html) function.
	```python
	print(f'Largest value for Tesla in Web Search: ')
	print(df_tesla.describe()['TSLA_WEB_SEARCH']['max'])
	print(f'Smallest value for Tesla in Web Search: ')
	print(df_tesla.describe()['TSLA_WEB_SEARCH']['min'])
	df_tesla.describe().loc(0)['max']
	```
5. what is the periodicity of the time series data (daily, weekly, monthly)?

|question|df_tesla|df_btc_search| df_btc_price | df_unemployment |
| --- |--- | --- |--- |--- |
|shape|(124,3) |(73, 2) |(2204, 3) | (181, 3) 
|rows|124 |73 |2204 |181 
|columns|3 |2 |73 |181 
|largest num|31 | 100| | 100
|periodicity| monthly|monthly |daily |monthly 

## Data Cleaning

### Missing Values

`.isna()` returns a series of booleans (False if the value is not NaN, true if it is.)
Chain `.values.any()` to see if any value in the series is `True`
```python
print(f'Missing values for BTC price?: ')
df_btc_price[df_btc_price.isna().any(axis=1)]
```

returns a dataframe with all the rows containing NaN values

## Number of NaN values

```pythong
print(f'Number of missing values: ')
(df_btc_price.shape[0] - df_btc_price.count()).sum()
# returns a dataframe where the rows are the original columns and the column is the number of NaN values.
# Sum the columns to get the total number of NaN values
df_btc_price.isna().any().sum()
```

## [Resampling](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.resample.html)

use `.resample()` to convert daily data (df_btc_price) to monthly data (df_btc_search)
* Specify column to use
* Specify sample frequency (the 'rule')
`df_btc_end_month_price = df_btc_price.resample(rule='M', on='DATE').last()`
avg `df_btc_end_month_price = df_btc_price.resample(rule='M', on='DATE').mean()`

## Data Visualization

* Plot the Tesla stock price against the Tesla search volume using a line chart and two different axes.
[[day74readme#Let's use two separate axes]]

```python
df_tesla.head()
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.set_xlabel("Months")
ax1.set_ylabel("Tesla Web Searches")
ax2.set_ylabel("Tesla Stock Price")
ax1.plot(df_tesla.MONTH, df_tesla.TSLA_WEB_SEARCH, color="purple")
ax2.plot(df_tesla.MONTH, df_tesla.TSLA_USD_CLOSE)
```


```python
# Create locators for ticks on the time axis
chart_figure = plt.figure(figsize=(14, 8), dpi=200)
plt.title("Tesla Web Search vs Price", fontsize=16)
plt.xticks(fontsize=14, rotation=45)
ax1 = plt.gca()
ax1.set_xlim(df_tesla.MONTH[0], df_tesla.MONTH[len(df_tesla.MONTH) - 1])
ax1.set_ylim(0, df_tesla.TSLA_WEB_SEARCH.max())
ax2 = ax1.twinx()
ax2.set_ylim(0, df_tesla.TSLA_USD_CLOSE.max())
ax1.set_xlabel("Months", fontsize=14)
ax1.set_ylabel("Tesla Web Searches", color="purple", fontsize=14)
ax2.set_ylabel("Tesla Stock Price", color="#FF0000", fontsize=14)
ax1.plot(df_tesla.MONTH, df_tesla.TSLA_WEB_SEARCH, color="purple", linewidth=3)
ax2.plot(df_tesla.MONTH, df_tesla.TSLA_USD_CLOSE, color="#FF0000", linewidth=3)
chart_figure.get_dpi()
plt.show()
```

## Using Locators and DateFormatter

Locators are useful for getting your tick marks to appear the way you want them to when working with time series data

Import `matplotlib.dates`
use `YearLocator()` and `MonthLocator()`
`DateFormatter()` will help us specify how we want to display the dates

```python
ax1 = plt.gca()
ax1.xaxis.set_major_locator(years)
ax1.xaxis.set_major_formatter(years_fmt)
ax1.xaxis.set_minor_locator(months)
ax1.set_xlim(df_tesla.MONTH[0], df_tesla.MONTH[len(df_tesla.MONTH) - 1])
ax1.set_ylim(0, df_tesla.TSLA_WEB_SEARCH.max())
ax2 = ax1.twinx()
```

### Plotting btc price vs search volume

#grid
#locators


```python
chart_figure = plt.figure(figsize=(14, 8), dpi=200)
resampled_btc_price = df_btc_price.resample(rule="M", on="DATE")

plt.title("Bitcoin News Search vs Resampled Price", fontsize=16)
plt.xticks(fontsize=14, rotation=45)
ax1 = plt.gca()
ax1.xaxis.set_major_locator(years)
ax1.xaxis.set_major_formatter(years_fmt)
ax1.xaxis.set_minor_locator(months)

ax1.set_xlim(df_btc_search.MONTH.min(), resampled_btc_price.last().index.max())
ax1.set_ylim(0, resampled_btc_price.mean().CLOSE.max() + 2000)

ax2 = ax1.twinx()

ax2.set_ylim(0, df_btc_search.BTC_NEWS_SEARCH.max())
ax1.set_xlabel("Months", fontsize=14)
ax1.set_ylabel("BTC Price", color="purple", fontsize=14)
ax2.set_ylabel("BTC News Web Searches", color="#FF0000", fontsize=14)

ax1.plot(resampled_btc_price.mean().index, resampled_btc_price.mean().CLOSE, color="purple", linewidth=3, linestyle="dashed")
ax2.plot(df_btc_search.MONTH[:74], df_btc_search.BTC_NEWS_SEARCH, color="#FF0000", linewidth=1, marker="o")

plt.show()
```


### Multiple axes refresher

```python
df_ue_search_vs_rate = pd.read_csv("UE Benefits Search vs UE Rate 2004-20.csv")
df_ue_search_vs_rate.MONTH = pd.to_datetime(df_ue_search_vs_rate.MONTH)
x1 = plt.gca()
x2 = x1.twinx()

x1.plot(x, df_ue_search_vs_rate.UNRATE)
x2.plot(x, df_ue_search_vs_rate.UE_BENEFITS_WEB_SEARCH)

```

## Learning Points and Summary

* Use `.describe()` to quickly see some descriptive statistics at a glance
* How to use resample() to make time-series data comparable to another by changing the periodicity
* How to work with matplotlib.dates Locators to better style a timeline (e.g., an axis on a chart)
* How to find the number of NaN values with isna().values.sum()
* How to change the resolution of a chart using the figures dpi
* How to created dashed and dotted lines using `linestyles`
* How to use different kinds of markers (e.g., `o` or `^`) on charts
* fine-tuning the styling of matplotlib charts by using limits, labels, linewidth and colours
* using .grid() to help visually identify seasonality in a time series
* 