# Time Series Forecast of Brent and WTI

Stan Chen Dec 14, 2019

---
### Introduction
This project is created to showcase the possibilities of performing time-series market prediction with Python scripts. Python is a general purpose programming language which in this case I'm using it to train time series predictive models to fill in the missing piece in corporate process solutions: **Forecasting time series analysis**.


### Why use this python procedure to perform this process?

While it's nice to use specialized software such as SAS or IBM SPSS, licensing these softwares could be expensive. 
No matter what tools or technologies are used in this process, it doesn't change the business process logic **as long as the process is done right**. I personally find it **flexible,cost effective and faster** to perform this process with python scripts. 

---
### Processes involved in this project:
1. Data Ingestion
2. Time series modeling
3. Perform prediction using trained models
---

###  CRISP-DM Framework
1. Business Understanding
2. Data Understanding
3. Data Preparation
4. Modeling
5. Evaluation
6. Deployment

Data source:
https://datahub.io/core/oil-prices#python



# Data Ingestion

```python
!pip install datapackage
from datapackage import Package
import matplotlib.pyplot as plt 
import pandas as pd

dp_url='https://datahub.io/core/oil-prices/datapackage.json'
dp = Package(dp_url)

# print list of all resources:
print(dp.resource_names)

# print processed tabular data (if exists any)
for resource in dp.resources:
    if resource.descriptor['datahub']['type'] == 'derived/csv':
        print(resource.read())
        
# to load only tabular data
src1=dp.get_resource('brent-monthly_csv').read(keyed=True)
src2=dp.get_resource('wti-monthly_csv').read(keyed=True)

# Loading the data into a Panda dataframe
# df1_bre : Data Frame for Brent
df1_bre=pd.DataFrame(src1)
# df2_wti : Data Frame for WTI
df2_wti=pd.DataFrame(src2)
```

# Visualizing Data

```python
plt.plot(df1_bre['Date'], df1_bre['Price'], label = 'Brent')
plt.title('Brent Price')
plt.ylabel('Price ($)');
plt.legend();
plt.show()

plt.plot(df2_wti['Date'], df2_wti['Price'], label = 'WTI')
plt.title('WTI Price')
plt.ylabel('Price ($)');
plt.legend();
plt.show();

plt.plot(df1_bre['Date'], df1_bre['Price'], label = 'Brent')
plt.plot(df2_wti['Date'], df2_wti['Price'], label = 'WTI')
plt.xlabel('Date');
plt.ylabel('Price ($)');
plt.legend();
plt.show();
```

 ![brent_plot](README.assets/brent_plot.png) 

 ![wti_plot](README.assets/wti_plot.png) 

 ![img](README.assets/2019003033.png) 



# Modeling & Forecasting
While there are several viable options to choose from Prophet, ARIMA, LTMS & other algorithms, in this case we're using Prophet.

```python
import fbprophet
# Prophet requires columns ds (Date) and y (value)
df1_bre = df1_bre.rename(columns={'Date': 'ds', 'Price': 'y'})
bre_pp = fbprophet.Prophet(changepoint_prior_scale=0.0225)

bre_pp.add_seasonality(name='Monthly', period=30.5, fourier_order=8)
bre_pp.fit(df1_bre)

# Make a future dataframe for 2 years
bre_forecast = bre_pp.make_future_dataframe(periods=12 * 2, freq='M')

# Make predictions
bre_forecast = bre_pp.predict(bre_forecast)

# Plot Projection
bre_pp.plot(bre_forecast, xlabel = 'Date', ylabel = 'Price ($)', uncertainty = True)
plt.title('Brent Price Forecast');
```

![brent_forecast](README.assets/brent_forecast.png)

```python

df2_wti = df2_wti.rename(columns={'Date': 'ds', 'Price': 'y'})
wti_pp = fbprophet.Prophet(changepoint_prior_scale=0.0225)
wti_pp.add_seasonality(name='Monthly', period=30.5, fourier_order=8)
wti_pp.fit(df2_wti)

# Make a future dataframe for 2 years
wti_forecast = wti_pp.make_future_dataframe(periods=12 * 2, freq='M')

# Make predictions
wti_forecast = wti_pp.predict(wti_forecast)


# Plot Projection
wti_pp.plot(wti_forecast, xlabel = 'Date', ylabel = 'Price ($)', uncertainty = True)
plt.title('WTI Price Forecast');
```

![wti_forecast](README.assets/wti_forecast.png)

# Plotting Trends and patterns

```python
print("Brent Forecast Trends and Patterns:")
bre_pp.plot_components(bre_forecast, uncertainty = True);
```



Brent Forecast Trends and Patterns:

![BrentForecastTrendsandPatterns](README.assets/BrentForecastTrendsandPatterns.png)

```python
print("WTI Forecast Trends and Patterns:")
wti_pp.plot_components(wti_forecast, uncertainty = True);	
```

 WTI Forecast Trends and Patterns: 

![WTIForecastTrendsandPatterns](README.assets/WTIForecastTrendsandPatterns.png)



# Extras
- [View the iPython Notebook on GitHub](https://github.com/stan587/Data-Science-Portfolio/blob/master/Time_Series_Oil_Price_Forecast/Time_Series_Forecast_of_Brent_and_WTI.ipynb)
- download the prediction in plain text
  - Brent 2 years forecast (download [bre_forecastOutput.csv](https://github.com/stan587/Data-Science-Portfolio/blob/master/Time_Series_Oil_Price_Forecast/bre_forecastOutput.csv) )
  - WTI 2 years forecast (download [wti_forecastOutput.csv](https://github.com/stan587/Data-Science-Portfolio/blob/master/Time_Series_Oil_Price_Forecast/wti_forecastOutput.csv) )



### References:

https://pypi.org/project/datapackage/#resource

https://datahub.io/docs/getting-started/getting-data

https://towardsdatascience.com/time-series-analysis-in-python-an-introduction-70d5a5b1d52a

https://towardsdatascience.com/time-series-forecasting-with-lstms-using-tensorflow-2-and-keras-in-python-6ceee9c6c651


https://www.tensorflow.org/tutorials/structured_data/time_series