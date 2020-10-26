# Predicting Cryptoasset Price Movements.

## Overview

The goal of this project is to see how the price and volume of Bitcoin affect the price of Litecoin and Ethereum. Additionally, daily returns for each asset will be forecasted.

## Data Wrangling

Data for this project was pulled from Kraken Digital Asset Exchange. There is a third-party Python library that uses the Kraken API to pull data and is formatted as a Pandas dataframe with the date-time as an index. This is much simpler than other API’s that were tried and leads to less errors. 

Initial data pulls showed an error where a certain date would have 0 values for certain columns. However, re-running the data pull would solve this error on it’s own.

All the code for the API data pull and additional data wrangling can be seen here:

[API Data Pull](https://github.com/hank-butler/Third-Capstone/blob/master/Third_Capstone_Data_Wrangling.ipynb)

## Exploratory Data Analysis

The initial plot of the Bitcoin, Ethereum, and Litecoin Volume Weight Average Price (VWAP) isn’t very insightful due to the massive difference in scale between Bitcoin’s price and the other assets.

[Link to EDA File](https://github.com/hank-butler/Third-Capstone/blob/master/Third_Capstone_EDA.ipynb)

To overcome the scaling issue, a log transformation of each price was conducted and plotted. After the log transformation, there appears to be a relationship between the assets at first glance. 

Next the Daily VWAP returns and Daily Log VWAP Returns were plotted for each asset and then all plotted together.

The Daily and Log Daily Returns appear to move similarly.

Histograms and KDE plots were plotted for Daily VWAP Returns and Log Daily Returns. Each histogram was fairly symmetric with slight left skew, indicating some large negative returns.

Daily volume had one of the more interesting graphs. Ethereum volume is the one that jumps out due to the various large spikes. Volume will be factored into later models.

Lastly, a correlation matrix for daily return was created. 

## Data Pre-processing

Pulling in the data with the Pandas .read_csv() method has the index in reverse order. The index was reversed with the .reindex() method. Next, a check for any NA values was run and yielded 0 NA’s.

The next step of pre-processing was creating the daily and log daily return columns. This was done with the .pct_change() method from Pandas.

## Modeling

[Link to Pre-Processing and Modeling](https://github.com/hank-butler/Third-Capstone/blob/master/Third_Capstone_Modeling.ipynb)

Before doing any time-series analysis, the Augmented Dickey Fuller Test (ADFuller Test) was conducted on the VWAP, Log VWAP, Daily VWAP Returns, and Log Daily VWAP Returns. The purpose of the ADFuller Test is to see if the time-series is stationary, the properties do not depend on the time at which the series is observed. The expectation is that the price series will be non-stationary, due to trends in the price, and that the returns series will be stationary. The expectation for the returns being stationary is from their plots and histograms not showing an identifiable trend.

After running the ADFuller Tests, the expectations were generally accurate. The only price series that was stationary was Ethereum, although its log price was non-stationary. All returns series were stationary.

The ADFuller Tests results were done to establish whether or not the series needed to be differenced to make it stationary and model. The Auto_Arima package was used to model each price and return series and provide an expectation for the Vector AutoRegression of Returns. Each model had a Moving Average (MA) component that ranged from 1 to 4. Only the price models were integrated (meaning they were ARIMA models). The most interesting observation was that ETH and BTC did not have an Autoregressive component which suggests that yesterday’s price does not have an influence on today’s price. However, LTC had an AR component of 4, which indicates the lagged prices from one to four days ago affect today’s price.

After the ADFuller Tests, a Vector AutoRegression was done to forecast returns. Below is the correlation matrix of residuals from the VAR.

Correlation matrix of residuals
                    BTC_daily_return  ETH_daily_return  LTC_daily_return

BTC_daily_return            1.000000          0.822321          0.773713

ETH_daily_return            0.822321          1.000000          0.832074

LTC_daily_return            0.773713          0.832074          1.000000

After the forecast of returns, the next step is building a regression model to see the effects of Bitcoin and Litecoin prices on the price of Ethereum and the effects of Bitcoin and Ethereum prices on the price of Litecoin. The second iteration of models will see the effects of Bitcoin price, Litecoin price, and volume of all assets on Ethereum price and the effects of Bitcoin price, Ethereum price, and volume on Litecoin price. 

The models used were linear regression, random forest regression, and gradient boosting regression from the Sci-Kit Learn package.

## Results and Next Steps

The best model for Ethereum was a random forest regression factoring Bitcoin price, Litecoin Price, and Volume. The R-squred on the training set was 0.98 and 0.96 on the test set. It is a very well fit model that generalizes well.

The best model for Litecoin was a random forest regression factoring Bitcoin price and Ethereum price. The R-squared on the training set was 0.92 and 0.83. It is slightly overfit but generalizes best out of all Litecoin models.

When compared to the baseline of just using yesterday’s price to predict today’s price, each model is better than the baseline. The baseline Ethereum R-squared was only 0.639, compared to the 0.96 from our best model. However the baseline Litecoin R-squared was marginally lower than our model’s R-squared, 0.801 and 0.83 respectively.

We know that there is a relationship between the assets that can be quantified. This could be used as the foundation for a mean reversion trading strategy. 

Further analysis could be used to analyze the effects of volatility on prices. Bitcoin and other crypto-assets have received significant attention for their price swings. If there is a significant trend in volatility, it could also be used as a trading strategy.
