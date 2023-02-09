# OpenForecast

The data that used in this work is environmental data, consist of 14 measurement: temperature, humidity, ... <br>
The first part of this work we visualise the environmental data using Python and Python libraries, to create a dynamic interactive heat map based on the measurement (temperature data), as we can see in the `heatmap` folder.

For creating the heat map, we used `OpenStreetMap` which is a collaborative project for creating a free editable map of the world. This project creates and distributes free geographic data of the world. The geographic data underlying this map is considered as major output of this project. On top of `OpenStreetMap`, we used the opensource JavaScript library `Leaflet` to create the interactive heat map. The enviromental data in our case need to be processed to a certain structure in order to visalise them using the above mentioned libraries.

### DataFlow - heat map

Filter and clean the data <br>
Transform the time-series data into hourly data <br>
Feed the transformed data into the libraries and create the heat map <br>

<div id="header" align="center">
  <img src="https://github.com/dingzehu/OpenForecast/blob/main/heatmap/interactive_map.gif" width="600"/>
</div>

### Outlier detection

#### Z-score
The number of standard deviations above and below the mean value of what is being observed or measured.

#### IQR (interquartile range)
A measurement of statistical dispersion. It means the observed data point would be the difference between the 25th and the 75th percentile, or between the lower and upper quartiles, IQR = Q3 - Q1. The upper limit is Q3 + 1.5 × IQR, and the lower limit is Q3 - 1.5 × IQR.

#### Isolation Forest algorithm
An unsupervised learning algorithm used for anomaly detection. In order to isolate the data points, this algorithm generates partitions recursively on the observed dataset, by randomly selecting a feature and then randomly selecting a value to split the minimum and maximum values. In our case, Isolation Forest detects more anomalous data.
