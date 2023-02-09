# OpenForecast

The data that used in this work is environmental data, consist of 14 measurement: temperature, humidity, ... <br>
The first part of this work we visualise the environmental data using Python and Python libraries, to create a dynamic interactive heat map based on the measurement (temperature data), as we can see in the `heatmap` folder.

For creating the heat map, we used `OpenStreetMap` which is a collaborative project for creating a free editable map of the world. This project creates and distributes free geographic data of the world. The geographic data underlying this map is considered as major output of this project. On top of `OpenStreetMap`, we used the opensource JavaScript library `Leaflet` to create the interactive heat map. The enviromental data in our case need to be processed to a certain structure in order to visalise them using the above mentioned libraries.

### DataFlow

Filter and clean the data
Transform the time-series data into hourly data
Feed the transformed data into the libraries and create the heat map


