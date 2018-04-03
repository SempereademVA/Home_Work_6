

```python
# READ THIS BEFORE RUNNING THIS PROGRAM
#
#Install the following python libraries using pip install as shown below:
# pip install matplotlib
# pip install pandas
# pip install requests
# pip install citipy
# pip install seaborn
# pip install plotly   <------------Don't forget this one.  You will need it to see a map with coordinates using this library

# Dependencies

%matplotlib inline

from random import uniform
import os
import time
import pandas as pd
import numpy as np
import requests
import json
from citipy import citipy
import matplotlib.pyplot as plt
import plotly
from citipy import citipy
import seaborn as sns

from config import APIKEY


#Define constants

base_url= 'http://api.openweathermap.org/data/2.5/weather?&q='
APPID ='&APPID=' + APIKEY
units = '&units=imperial'
```


```python
get_url = base_url + 'bluff' + units + APPID
```


```python
weather = requests.get(get_url)
weather_j= weather.json()
```


```python
#Check the randomness of the uniform function

np.random.seed(42)   #Keep this the same for development and debugging purposes
N = 10000  # no of samples
x = range(N)
y = [np.random.uniform(-180,180) for i in x]
plt.plot(x, y)
plt.show()


```


```python
np.random.seed(42)   #Keep this the same for development and debugging purposes
N = 10000  # no of samples
x = range(N)
y = [np.random.uniform(-90,90) for i in x]
plt.plot(x, y)
plt.show()
```


```python
#Plots are consistent with a uniform distribution
```


```python
import time
now = time.strftime("%c")
print ("Current time %s"  % now )
print(now + ' url')
the_date = time.strftime("%x")
print(the_date)
```


```python
log_api_call_f = open('apilog.txt', 'w')        #This file contains a log of the API calls to the weather server
city_f = open('citydatax000.csv', 'w')          #This file is the CSV for the weather data provided by the weather server

#Write the column headings to CSV file
city_f.write('City,' +  'Latitude,' + 'Longitude,' + 'Temperature (F),' + 
             'Humidity (%),' + 'Cloudiness (%),' + 'Winds (mph),'+'\n')

np.random.seed(20)     #Set the seed for the pseudorandom number generator for debugging purposes

for i in range(2000):
    lat, lon = np.random.uniform(-90, 90), np.random.uniform(-180,180)
    location = (lat, lon)
    city = citipy.nearest_city(lat, lon)     #Get the nearest city from the random latitude and longitude
    
    
    get_url = base_url + city.city_name + units + APPID     #Assemble the API call with the city
    
    now = time.strftime("%c")
    
    no_api_url = base_url + city.city_name + units + 'XXXXXXXXXXXXXXXXXXXX'
    print(now + ' ' + no_api_url)
    log_api_call_f.write(now + ' ' + no_api_url +'\n')
    
    weather = requests.get(get_url)                         #Make the call to the server for the city's weather
    weather_j= weather.json()                               #Read th JSON information returned from the server
    if weather_j['cod'] == 200:                 #Make sure the server has weather information on the city,  Check the cod code 
        city_f.write(city.city_name + ',' + f'{lat:.6f}' + ',' + f'{lon:.6f}' + ',' + str(weather_j['main']['temp_max']) +',' + 
                     str(weather_j['main']['humidity']) +',' + str(weather_j['clouds']['all']) +',' + 
                     str(weather_j['wind']['speed']) +'\n')
#Processing Complete.  Close the file        
city_f.close()
log_api_call_f.close()
```


```python
now = time.strftime("%c")
print ("End time: %s"  % now)
```


```python
weather_df = pd.read_csv('citydatax000.csv')
weather_df=weather_df.drop(weather_df.columns[weather_df.columns.str.contains('unnamed',case = False)],axis = 1)
weather_df.tail()
```


```python
weather_df.shape
```


```python
weather_df.info()
```


```python
weather_df['City'].value_counts(normalize=True)
```


```python
weather_dedup_df=weather_df.drop_duplicates(['City'])
weather_dedup_df['City'].value_counts(normalize=True)
```


```python
weather_dedup_df.info()
```


```python
df = weather_dedup_df
```


```python
df.head()
```


```python


scl = [ [0,"rgb(5, 10, 172)"],[0.35,"rgb(40, 60, 190)"],[0.5,"rgb(70, 100, 245)"],\
    [0.6,"rgb(90, 120, 245)"],[0.7,"rgb(106, 137, 247)"],[1,"rgb(220, 220, 220)"] ]

data = [ dict(
        type = 'scattergeo',
        lon = df['Longitude'],
        lat = df['Latitude'],
        text = df['City'],
        mode = 'markers',
        marker = dict(
            size = 8,
            opacity = 0.8,
            reversescale = True,
            autocolorscale = False,
            symbol = 'square',
            line = dict(
                width=1,
                color='rgba(102, 102, 102)'
            ),
            colorscale = scl,
            cmin = 0,
           
        ))]

layout = dict(
        title = 'Nearest City Locations for Weather Data',
        colorbar = True,
        geo = dict(
            scope='world',
            showland = True,
            landcolor = "rgb(250, 250, 250)",
            subunitcolor = "rgb(217, 217, 217)",
            countrycolor = "rgb(217, 217, 217)",
            countrywidth = 0.5,
            subunitwidth = 0.5
        ),
    )

fig = dict( data=data, layout=layout )


plotly.offline.plot( fig, validate=False )                  #Check your directory for an html figure: temp-plot

plotly.offline.init_notebook_mode()

plotly.offline.iplot( fig, validate=False, image='png')     #Save the image

#Move cursor over each point to get latitude and longitude
```


```python
sns.set(font_scale=3)
ax, fig = plt.subplots(figsize=(20,10))
plt.xlim(-190, 190)
sns.set_style("whitegrid", {'axes.grid' : True})
sns.regplot(x=df['Longitude'], y=df['Latitude'], fit_reg=False)
ax.suptitle('City Coordinates ' + the_date)
plt.savefig('lat_long.png')
```


```python
sns.set(font_scale=3)
ax, fig = plt.subplots(figsize=(20,10))
sns.set_style("whitegrid", {'axes.grid' : True})
sns.regplot(x=df['Latitude'], y=df['Temperature (F)'], fit_reg=False)
ax.suptitle('Max. Temperature vs. City Latitude ' + the_date)
plt.savefig('temp_lat.png')
plt.show()
```


```python
sns.set(font_scale=3)
ax, fig = plt.subplots(figsize=(20,10))
sns.set_style("whitegrid", {'axes.grid' : True})
sns.regplot(x=df['Latitude'], y=df['Humidity (%)'], fit_reg=False)
ax.suptitle("Humidity vs. City Latitude " + the_date)
plt.savefig('humid_lat.png')
plt.show()
```


```python
sns.set(font_scale=3)
ax, fig = plt.subplots(figsize=(20,10))
sns.regplot(x=df['Latitude'], y=df['Cloudiness (%)'], fit_reg=False)
ax.suptitle("Cloudiness vs. City Latitude " + the_date)
plt.savefig('cloud_lat.png')
plt.show()
```


```python
sns.set(font_scale=3)
ax, fig = plt.subplots(figsize=(20,10))
sns.regplot(x=df['Latitude'], y=df['Winds (mph)'], fit_reg=False)
ax.suptitle("Wind Speed vs. City Latitude " + the_date)
plt.savefig('winds_lat.png')
plt.show()
```


```python
#End of Program
```
