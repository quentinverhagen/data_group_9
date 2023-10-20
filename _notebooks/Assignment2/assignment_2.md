## Assignment 2
Question 1: How many of the canal boats currently in use are diesel/fossil fuel driven and how many boats are electrical driven?

To answer this we use the passenger shipping register of the municipality of Amsterdam.


```python
#Scrape data from the online register passagiersvaart

import requests
from bs4 import BeautifulSoup

#Access the website containing the municipal passenger shipping register
URL = "https://www.amsterdam.nl/verkeer-vervoer/varen-amsterdam/varen-beroepsvaart/register-passagiersvaart-amsterdam/"
page = requests.get(URL)

#Retreive HTML data
soup = BeautifulSoup(page.content, "html.parser")
```


```python
#transform data into lists for every column

l1 = []
l2 = []
for td_tag in soup.find_all('td'):
    l1.append(td_tag.text) 
x = 0
count = 1
operators = []
name = []
amount = []
numberinuse = []
segment = []
enddate = []
area = []

for i in l1:
    if x%7 == 0:
        operators.append(i)
        l2.append(count)
        count = count + 1
    if x%7 == 1:
        name.append(i)
    if x%7 == 2:
        amount.append(i)
    if x%7 == 3:
        numberinuse.append(i)
    if x%7 == 4:
        segment.append(i)
    if x%7 == 5:
        enddate.append(i)
    if x%7 == 6:
        area.append(i)
    x = x + 1 
```


```python
#combine lists into dictionary

from itertools import zip_longest

canalboatsdic = {
    row[0]: row[1:]
    for row in zip_longest(l2,
                            operators,
                            name,
                            amount,
                            numberinuse,
                            segment,
                            enddate,
                            area)
}

#References: Stackoverflow
```


```python
#Transform the dictionary into a dataframe

import pandas as pd

df = pd.DataFrame(data=canalboatsdic)

df = (df.T)
df.rename(columns={0: "Operator", 1: "Name of boat(s)", 2: "Amount of boats", 3: "Number of boats in use", 4: "Segment", 5: "End date", 6: "Area"  }, inplace=True)

df.head()
```




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
      <th>Operator</th>
      <th>Name of boat(s)</th>
      <th>Amount of boats</th>
      <th>Number of boats in use</th>
      <th>Segment</th>
      <th>End date</th>
      <th>Area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Adam's Boats B.V.</td>
      <td>Adam 01 t/m Adam 10</td>
      <td>10</td>
      <td>10</td>
      <td>onbemand</td>
      <td>1 maart 2026</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ambolux Classic Ecoboats</td>
      <td>Ambolux 3</td>
      <td>1</td>
      <td>1</td>
      <td>bemand open</td>
      <td>1 maart 2030</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Amsterdam Boat Adventures vof</td>
      <td>Amsterdam Boat Adventures</td>
      <td>1</td>
      <td>1</td>
      <td>bemand open</td>
      <td>1 maart 2028</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Amsterdam Boat Center</td>
      <td>Amigo</td>
      <td>1</td>
      <td>1</td>
      <td>bemand open</td>
      <td>1 maart 2024</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Amsterdam Boat Center</td>
      <td>Salonboot Tzar</td>
      <td>1</td>
      <td>1</td>
      <td>bemand open</td>
      <td>1 maart 2028</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Select all the canal boats

df2 = df[df["Segment"]=="bemand groot"]
df2["numberinuse_int"] = pd.to_numeric(df2["Number of boats in use"])
numberofboats = sum(df2["numberinuse_int"])
print("The total of canal boats in Amsterdam is", numberofboats)
```

    The total of canal boats in Amsterdam is 145
    

    C:\Users\quent\AppData\Local\Temp\ipykernel_14496\1184196463.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      df2["numberinuse_int"] = pd.to_numeric(df2["Number of boats in use"])
    

The number of canal boats in Amsterdam is 145. The number of large electrical boats in Amsterdam is negligible. As such we assume that all of these are diesel driven.

Question 3: Try to compare the energy use of the canal boats to some other activities in the city.

To answer this question we have used the TNO report that researched the energy needs of electric canal boats. The numbers from this report are imported in the code below and the total energy demand for a future fleet of electric canal boats is calculated.


```python
energy_usehour = 21.1 #average energy use for a canal boat per hour, retreived from TNO

#adjust type
numberofboats = float(numberofboats)
energy_usehour = float(energy_usehour)

#calculate total energy demand
energy_demand_hour = numberofboats*energy_usehour
total_energy_demand = energy_demand_hour*2500

#compare to other activities
energy_use_tram = 44124174 #data from jaarrapport GVB
energy_use_bus = 1900731 #data from jaarrapport GVB, concers 31 electric busses

print("The energy demand for all canal boat per hour is", energy_demand_hour, "KWh")
print("The average canal boat operates 2500 hours a year. This results in a total energy demand of", int(total_energy_demand), "KWh per year")
print("This is", round(energy_use_tram/total_energy_demand,1), "less than yearly energy use of trams in Amsterdam")
print("Compared to an electric bus, an electric canal boat uses", round((energy_use_bus/31)/(energy_usehour*2500), 2), "times more energy per year.")
```

    The energy demand for all canal boat per hour is 3059.5 KWh
    The average canal boat operates 2500 hours a year. This results in a total energy demand of 7648750 KWh per year
    This is 5.8 less than yearly energy use of trams in Amsterdam
    Compared to an electric bus, an electric canal boat uses 1.16 times more energy per year.
    

Question 2: Are there peak times for the canal boats?

We use the information from de grachtmonitor 2022 report to visualize the most and least crowded points in Amsterdams waterways.
This reveals that the area around the Singelgracht and Amstel are the most crowded and that the inner canals have less boat traffic. Aditionally, the peak time for boat traffic is between 15.00PM and 20.00PM. The mornings are relatively quiet.


```python
import matplotlib.pyplot as plt
import numpy as np

#import the data from grachtmonitor
df_canalboats = pd.read_excel("data/gracht.xlsx")
df_canalboats = df_canalboats.iloc[0:14, 0:12]
df_canalboats = df_canalboats.set_index(df_canalboats.columns[0])

#sort the different measuring points by highest average value
average_values = df_canalboats.mean()
sorted_columns = average_values.sort_values(ascending=False).index

#Make a graph in which the busiest points are represented by red and opposite by green
colormap = plt.get_cmap('RdYlGn')
line_colors = [colormap(i) for i in np.linspace(0, 1, len(sorted_columns))]

x_values = np.arange(len(df_canalboats.index)) 
x_labels = df_canalboats.index

ax = df_canalboats[sorted_columns].plot(figsize=(20, 5), color=line_colors, linewidth='3.5')

ax.set_xticks(x_values)
ax.set_xlabel("Time periods during the day")
ax.set_ylabel("Number of boats passing by")
ax.set_xticklabels(x_labels, rotation=45) 
plt.show()

df_canalboats.head()
```


    
![png](assignment_2_files/assignment_2_10_0.png)
    





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
      <th>Berlagebrug</th>
      <th>Haarlemersluis</th>
      <th>Korte Niezel</th>
      <th>Korte Prinsengracht/Brouwersgracht</th>
      <th>Nieuwe Meersluis</th>
      <th>Prinsengracht/Leidsegracht</th>
      <th>Prinsengracht/Reguliersgracht</th>
      <th>Prinsengracht/Leliegracht</th>
      <th>Reguliersgracht, zicht op 7 bogen</th>
      <th>Singelgracht-Amstel</th>
      <th>St. Antoniesluis/Snoekjesgracht</th>
    </tr>
    <tr>
      <th>Tijd (duur)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9:00-10:00</th>
      <td>52</td>
      <td>5</td>
      <td>4</td>
      <td>20</td>
      <td>7</td>
      <td>33</td>
      <td>16</td>
      <td>17</td>
      <td>19</td>
      <td>37</td>
      <td>16</td>
    </tr>
    <tr>
      <th>10:00-11:00</th>
      <td>72</td>
      <td>24</td>
      <td>12</td>
      <td>44</td>
      <td>14</td>
      <td>65</td>
      <td>50</td>
      <td>44</td>
      <td>38</td>
      <td>85</td>
      <td>42</td>
    </tr>
    <tr>
      <th>11:00-12:00</th>
      <td>108</td>
      <td>40</td>
      <td>24</td>
      <td>68</td>
      <td>29</td>
      <td>112</td>
      <td>82</td>
      <td>82</td>
      <td>72</td>
      <td>139</td>
      <td>72</td>
    </tr>
    <tr>
      <th>12:00-13:00</th>
      <td>141</td>
      <td>44</td>
      <td>38</td>
      <td>91</td>
      <td>54</td>
      <td>140</td>
      <td>116</td>
      <td>91</td>
      <td>88</td>
      <td>180</td>
      <td>82</td>
    </tr>
    <tr>
      <th>13:00-14:00</th>
      <td>179</td>
      <td>57</td>
      <td>46</td>
      <td>102</td>
      <td>62</td>
      <td>165</td>
      <td>142</td>
      <td>84</td>
      <td>74</td>
      <td>237</td>
      <td>99</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Match the measuring points with geometry data and transform it into a dataframe

listnames = []
listgeo = []
df_measuringpoints = pd.read_excel("data/meetpunten.xlsx") #coordinates of measuring points, data retreived from google maps

for i in df_measuringpoints['Geometry']:
    listgeo.append(i)

for i in df_canalboats.columns:
    listnames.append(i)

dict_geo = {key: value for key, value in zip(listnames, listgeo)}
df_geo = pd.DataFrame(list(dict_geo.items()), columns=['Name', 'Geometry'])
df_geo[['Latitude', 'Longitude']] = df_geo['Geometry'].str.split(', ', expand=True)

df_geo
```




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
      <th>Name</th>
      <th>Geometry</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Berlagebrug</td>
      <td>52.34731041620543, 4.912464234049668</td>
      <td>52.34731041620543</td>
      <td>4.912464234049668</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Haarlemersluis</td>
      <td>52.378855408032734, 4.894052455733411</td>
      <td>52.378855408032734</td>
      <td>4.894052455733411</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Korte Niezel</td>
      <td>52.3747813856251, 4.899337459619198</td>
      <td>52.3747813856251</td>
      <td>4.899337459619198</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Korte Prinsengracht/Brouwersgracht</td>
      <td>52.381504406085575, 4.889320914469436</td>
      <td>52.381504406085575</td>
      <td>4.889320914469436</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nieuwe Meersluis</td>
      <td>52.3390697100906, 4.84682573678532</td>
      <td>52.3390697100906</td>
      <td>4.84682573678532</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Prinsengracht/Leidsegracht</td>
      <td>52.36619139823907, 4.882766486124969</td>
      <td>52.36619139823907</td>
      <td>4.882766486124969</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Prinsengracht/Reguliersgracht</td>
      <td>52.361609137238545, 4.896049060525268</td>
      <td>52.361609137238545</td>
      <td>4.896049060525268</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Prinsengracht/Leliegracht</td>
      <td>52.37593571879353, 4.884250522824771</td>
      <td>52.37593571879353</td>
      <td>4.884250522824771</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Reguliersgracht, zicht op 7 bogen</td>
      <td>52.36481430860875, 4.895862174664486</td>
      <td>52.36481430860875</td>
      <td>4.895862174664486</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Singelgracht-Amstel</td>
      <td>52.3592314826586, 4.904608930896866</td>
      <td>52.3592314826586</td>
      <td>4.904608930896866</td>
    </tr>
    <tr>
      <th>10</th>
      <td>St. Antoniesluis/Snoekjesgracht</td>
      <td>52.370111810214794, 4.9016282699582</td>
      <td>52.370111810214794</td>
      <td>4.9016282699582</td>
    </tr>
  </tbody>
</table>
</div>




```python
import geopandas as gpd
from shapely.geometry import Point
import contextily as ctx
import matplotlib.pyplot as plt


#transform dataframe into a GeoDataFrame
df_geo = gpd.GeoDataFrame(df_geo, geometry=gpd.points_from_xy(df_geo['Longitude'], df_geo['Latitude']), crs="EPSG:4326")
ax = df_geo.to_crs(epsg=3857).plot(figsize=(10, 10), alpha=0.5, edgecolor="k", color="red", markersize=100)
ctx.add_basemap(ax, zoom=15, source=ctx.providers.OpenStreetMap.Mapnik) 
plt.show()

df_geo
```


    
![png](assignment_2_files/assignment_2_12_0.png)
    





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
      <th>Name</th>
      <th>Geometry</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Berlagebrug</td>
      <td>52.34731041620543, 4.912464234049668</td>
      <td>52.34731041620543</td>
      <td>4.912464234049668</td>
      <td>POINT (4.91246 52.34731)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Haarlemersluis</td>
      <td>52.378855408032734, 4.894052455733411</td>
      <td>52.378855408032734</td>
      <td>4.894052455733411</td>
      <td>POINT (4.89405 52.37886)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Korte Niezel</td>
      <td>52.3747813856251, 4.899337459619198</td>
      <td>52.3747813856251</td>
      <td>4.899337459619198</td>
      <td>POINT (4.89934 52.37478)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Korte Prinsengracht/Brouwersgracht</td>
      <td>52.381504406085575, 4.889320914469436</td>
      <td>52.381504406085575</td>
      <td>4.889320914469436</td>
      <td>POINT (4.88932 52.38150)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nieuwe Meersluis</td>
      <td>52.3390697100906, 4.84682573678532</td>
      <td>52.3390697100906</td>
      <td>4.84682573678532</td>
      <td>POINT (4.84683 52.33907)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Prinsengracht/Leidsegracht</td>
      <td>52.36619139823907, 4.882766486124969</td>
      <td>52.36619139823907</td>
      <td>4.882766486124969</td>
      <td>POINT (4.88277 52.36619)</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Prinsengracht/Reguliersgracht</td>
      <td>52.361609137238545, 4.896049060525268</td>
      <td>52.361609137238545</td>
      <td>4.896049060525268</td>
      <td>POINT (4.89605 52.36161)</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Prinsengracht/Leliegracht</td>
      <td>52.37593571879353, 4.884250522824771</td>
      <td>52.37593571879353</td>
      <td>4.884250522824771</td>
      <td>POINT (4.88425 52.37594)</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Reguliersgracht, zicht op 7 bogen</td>
      <td>52.36481430860875, 4.895862174664486</td>
      <td>52.36481430860875</td>
      <td>4.895862174664486</td>
      <td>POINT (4.89586 52.36481)</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Singelgracht-Amstel</td>
      <td>52.3592314826586, 4.904608930896866</td>
      <td>52.3592314826586</td>
      <td>4.904608930896866</td>
      <td>POINT (4.90461 52.35923)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>St. Antoniesluis/Snoekjesgracht</td>
      <td>52.370111810214794, 4.9016282699582</td>
      <td>52.370111810214794</td>
      <td>4.9016282699582</td>
      <td>POINT (4.90163 52.37011)</td>
    </tr>
  </tbody>
</table>
</div>




```python
import os
import gc

#format data to be suitable to join together
df_canalboats_pivot = df_canalboats.T
measurepoints = df_geo
df_canalboats_pivot['name_measurepoint'] = df_canalboats_pivot.index
df_canalboats_pivot = df_canalboats_pivot.reset_index(drop=True)

#join data 
joinData = measurepoints.set_index('Name').join(df_canalboats_pivot.set_index('name_measurepoint'))
outputpath = "output/"

hours = []
for x in range (4, 18):
    hours.append(joinData.columns[x])

#determine details for the legend
vmin, vmax = 0, 300
x = 1

#create image for all the time periods
for hour in hours:
    fig, ax = plt.subplots(1,1,figsize=(20,10))
    map = joinData.plot(ax = ax, column = hour, cmap = 'RdYlGn_r', vmin = vmin, vmax = vmax, markersize=1250, alpha = 0.8, marker='s', legend=True)
    map.set_title(hour, fontsize = 25)
    ctx.add_basemap(ax, crs=joinData.crs, source=ctx.providers.CartoDB.Positron)
    
    filepath = os.path.join(outputpath, str(x) + 'boats_at_time.png')
    mapfig = map.get_figure()
    mapfig.savefig(filepath)
    plt.close('all')
    mapfig.clf()
    gc.collect()
    x = x+1

#References: Stackoverflow, Youtube

```


```python
#combine all the images into one GIF
from PIL import Image
import glob

frames = []
images = glob.glob(outputpath + '/*.png')

for i in images:
    image_frame = Image.open(i)
    frames.append(image_frame)

frames[0].save(outputpath + '/boatsattime.gif', format = 'GIF', append_images = frames[1:], save_all = True, duration = 1000, loop = 0)     
```

Question 4: Would you consider it economically feasible?

Based on the data from the TNO report, we have plotted the period it takes to recover the initial investment needed to electrify a canal boat. In the most optimistic scenario this takes little over 7 years to a maximum of about 8.5 years in the most pessimistic scenario. 


```python
from matplotlib.patches import Ellipse
from matplotlib.text import OffsetFrom

#state variables
costs_to_electric_low = 70000
costs_to_electric_high = 85000
costs_operational_loss = 50000

operational_hours_year = 3000
price_electricity_kwh = 0.39
price_fuel_liter = 1.6
price_fuel_kwh = price_fuel_liter/2.5

#calculate business case
profit_from_elec = (price_fuel_kwh-price_electricity_kwh)*energy_usehour*operational_hours_year

business_case = -(costs_to_electric_high+costs_operational_loss)+profit_from_elec

profit_low = []
for year in range(0, 15):
    business_case = -(costs_to_electric_high+costs_operational_loss)+profit_from_elec*year
    profit_low.append(business_case)

profit_high = []
for year in range(0, 15):
    business_case = -(costs_to_electric_low+costs_operational_loss)+profit_from_elec*year
    profit_high.append(business_case)

#calculate payback-time
total_cost_high = costs_to_electric_high + costs_operational_loss
total_cost_low = costs_to_electric_low + costs_operational_loss

year_high = total_cost_high/profit_from_elec
year_low = total_cost_low/profit_from_elec
print(year_high, year_low)

#plot the return on investment
fig, ax = plt.subplots()
plt.plot(profit_low, label='High cost estimate', color = 'red' )
plt.plot(profit_high, label='Low cost estimate', color = 'green')
plt.axhline(y=0, color='black', linestyle='--')
plt.xlabel('Time (years)')
plt.ylabel('Profit (€)')
plt.title('Return on investment period for the electrification of a canal boat', fontsize = 9)

#annotate return on investment
ax.annotate('It takes a maximum of 8,5 years\nto earn back the investment',
            xy=(year_high, 0), xycoords='data',
            xytext=(8, -22), textcoords='offset points',
            bbox=dict(boxstyle="round",
                      fc=(0.5, 0.7, 0.7), ec="none"),
            fontsize = 8
)




plt.legend()
plt.show()
print(profit_from_elec)
```

    8.53080568720379 7.582938388625592
    


    
![png](assignment_2_files/assignment_2_16_1.png)
    


    15825.000000000002
    

Question 5: If only clean energy can be used, how many solar panels or wind turbines are needed?
Our calculations show that the total yearly energy demand for all canal boats is 7,6 million KWh. This is the equivelant of the annual energy production of 1,2 wind turbines or about 22.500 solar panels.

Question 6: Is it possible to accommodate these facilities within the city?
Yes, this easily fits in within the city limits. The total flat roof available in Amsterdam could accommodate a total of 9,7 million solar panels. Currently, there are about 1 million solar panels. This leaves plenty of room to fit in the 22.500 solar panels. Concerning wind energy, there is only need for one turbine to deliver most of the demanded energy. This fits well within the current municipal plans.


```python
energy_production_pv = 340 #https://www.vattenfall.nl/zonnepanelen/opbrengst/
energy_production_wind = 6500000 #https://www.gaslicht.com/energie-informatie/windmolen#:~:text=Vermogen%20van%20een%20windmolen,-Hoe%20zit%20dat&text=Een%20windmolen%20met%20een%20vermogen,huishouden%20van%20stroom%20kan%20voorzien.

solarpanels = int(total_energy_demand/energy_production_pv)
windturbines = round((total_energy_demand/energy_production_wind),1)
print(windturbines)

df_platdak = pd.read_excel("data/Plat_dak_gemeente_20210823.xlsx")
amsterdam_data = df_platdak[df_platdak['Gemeentenaam 2021'] == "Amsterdam"]
last_index = amsterdam_data.index[-1]

get_next_rows = df_platdak.iloc[last_index + 1:last_index + 13]
df_platdak_total = pd.concat([amsterdam_data, get_next_rows])
area = 0 
for row in df_platdak_total["Wel plat dak (m2)"]:
    #print(row)
    area = area + row

print(area/1.94) #possible solar panels
print("There is", area, "m2 of flat roof space in Amsterdam. This is enough space for", int(area/1.94), "solar panels. More then sufficient then the", solarpanels, "solar panels that are required. To produce the energy with windturbines,", windturbines, "windturbines are needed. This also fits in the cuurent plans of the municipality to build wind turbines within the municipal boundaries")


```

    1.2
    9753130.927835051
    There is 18921074 m2 of flat roof space in Amsterdam. This is enough space for 9753130 solar panels. More then sufficient then the 22496 solar panels that are required. To produce the energy with windturbines, 1.2 windturbines are needed. This also fits in the cuurent plans of the municipality to build wind turbines within the municipal boundaries
    


```python
#References
#https://www.youtube.com/watch?v=eBnryZKLbcE&t=594s
#https://realpython.com/beautiful-soup-web-scraper-python/
```

For question 5 and 8 we could not find data to provide a properly substantiated answer.
