## Inside Edinburgh Airbnb

### Inspiration and aim of this project

The Airbnb model, which connects travelers with individuals offering accommodations in their homes or properties, offers several advantages. It provides a wide range of lodging options, often at lower prices than traditional hotels, and allows hosts to earn income from their unused space. It fosters a sense of community and cultural exchange, enabling travelers to immerse themselves in local cultures. 

However, this model has its downsides, including concerns about safety and security, potential neighborhood disturbances caused by irresponsible guests, and issues related to housing shortages and the displacement of long-term residents in popular tourist destinations. Additionally, regulatory and tax compliance challenges have been a source of controversy in many regions, and some critics argue that Airbnb can contribute to rising rental costs and reduced housing availability in certain markets.

[Inside Airbnb](http://insideairbnb.com/about/) is a mission driven project that provides data and advocacy about Airbnb's impact on residential communities. I reviewed the available data and noted that Edinburgh, Scotland (a city I spent 14 years in) is included which inspired me to perform a data dive. I decided I would try to enrich my analysis by making use of spatial data that I used in another [recent project](https://github.com/Stephen137/Scottish-Index-of-Multiple-Deprivation) on the Scottish Index of Multiple Deprivation (SIMD).

My overall aim is to create a choropleth map of the area covered by the Edinburgh Airbnb listings to reveal the density of listings in each data zone. I hope to achieve this by deconstructing the problem into the following steps :


- `Step 1:` Establish which datazone each of the Airbnb listings falls within by performing a spatial join

- `Step 2:` Calculate the number of property listings within each data zone

- `Step 3:` Calculate the property listing density per data zone using the calculation above and the area metadata that is included with the data zone shape file

- `Step 4:` Plot a choropleth map with appropriate colour ramp to provide insights regarding the density of Airbnb properties across Edinburgh

### Import the required packages


```python
import os
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
```

### Load in our data

#### Spatial Data


```python
datazones = gpd.read_file('data/shapefiles/sc_dz_11.shp')
print(f"Number of rows: {len(datazones)}")
```

    Number of rows: 6976



```python
datazones.head()
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01006506</td>
      <td>Culter - 01</td>
      <td>872</td>
      <td>852</td>
      <td>424</td>
      <td>438.880218</td>
      <td>4.388801</td>
      <td>11801.872345</td>
      <td>4.388802e+06</td>
      <td>POLYGON ((-2.27748 57.09527, -2.27644 57.09521...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01006507</td>
      <td>Culter - 02</td>
      <td>836</td>
      <td>836</td>
      <td>364</td>
      <td>22.349739</td>
      <td>0.223498</td>
      <td>2900.406362</td>
      <td>2.217468e+05</td>
      <td>POLYGON ((-2.27354 57.10449, -2.27333 57.10449...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01006508</td>
      <td>Culter - 03</td>
      <td>643</td>
      <td>643</td>
      <td>340</td>
      <td>27.019476</td>
      <td>0.270194</td>
      <td>3468.761949</td>
      <td>2.701948e+05</td>
      <td>POLYGON ((-2.27443 57.10171, -2.27237 57.10046...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01006509</td>
      <td>Culter - 04</td>
      <td>580</td>
      <td>580</td>
      <td>274</td>
      <td>9.625426</td>
      <td>0.096254</td>
      <td>1647.461389</td>
      <td>9.625426e+04</td>
      <td>POLYGON ((-2.26611 57.10133, -2.26599 57.10092...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01006510</td>
      <td>Culter - 05</td>
      <td>644</td>
      <td>577</td>
      <td>256</td>
      <td>18.007657</td>
      <td>0.180076</td>
      <td>3026.111412</td>
      <td>1.800766e+05</td>
      <td>POLYGON ((-2.26013 57.10160, -2.26050 57.10134...</td>
    </tr>
  </tbody>
</table>
</div>



So, as you can our `datazones` dataset contains `6,976` rows which represent "datazones" as defined and included as part of the Scottish Government Index of Multiple Deprivation (SIMD) and we have some basic metadata associated with these datazones, in particular the area of each zone which we will be using to calculate the density of Airbnb property listings.


```python
type(datazones)
```




    geopandas.geodataframe.GeoDataFrame



Our `datazones` dataset has been loaded from a shapefile using geopandas and is a `GeoDataFrame` - we have a `geometry` column which includes the Polygon geomtery of each of the 6,976 data zones. We can visualize in just one line of code :


```python
datazones.plot()
```




    <AxesSubplot: >




    
![png](images/output_13_1.png)## Inside Edinburgh Airbnb

### Inspiration and aim of this project

The Airbnb model, which connects travelers with individuals offering accommodations in their homes or properties, offers several advantages. It provides a wide range of lodging options, often at lower prices than traditional hotels, and allows hosts to earn income from their unused space. It fosters a sense of community and cultural exchange, enabling travelers to immerse themselves in local cultures. 

However, this model has its downsides, including concerns about safety and security, potential neighborhood disturbances caused by irresponsible guests, and issues related to housing shortages and the displacement of long-term residents in popular tourist destinations. Additionally, regulatory and tax compliance challenges have been a source of controversy in many regions, and some critics argue that Airbnb can contribute to rising rental costs and reduced housing availability in certain markets.

[Inside Airbnb](http://insideairbnb.com/about/) is a mission driven project that provides data and advocacy about Airbnb's impact on residential communities. I reviewed the available data and noted that Edinburgh, Scotland (a city I spent 14 years in) is included which inspired me to perform a data dive. I decided I would try to enrich my analysis by making use of spatial data that I used in another [recent project](https://github.com/Stephen137/Scottish-Index-of-Multiple-Deprivation) on the Scottish Index of Multiple Deprivation (SIMD).

My overall aim is to create a choropleth map of the area covered by the Edinburgh Airbnb listings to reveal the density of listings in each data zone. I hope to achieve this by deconstructing the problem into the following steps :


- `Step 1:` Establish which datazone each of the Airbnb listings falls within by performing a spatial join

- `Step 2:` Calculate the number of property listings within each data zone

- `Step 3:` Calculate the property listing density per data zone using the calculation above and the area metadata that is included with the data zone shape file

- `Step 4:` Plot a choropleth map with appropriate colour ramp to provide insights regarding the density of Airbnb properties across Edinburgh

### Import the required packages


```python
import os
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
```

### Load in our data

#### Spatial Data


```python
datazones = gpd.read_file('data/shapefiles/sc_dz_11.shp')
print(f"Number of rows: {len(datazones)}")
```

    Number of rows: 6976



```python
datazones.head()
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01006506</td>
      <td>Culter - 01</td>
      <td>872</td>
      <td>852</td>
      <td>424</td>
      <td>438.880218</td>
      <td>4.388801</td>
      <td>11801.872345</td>
      <td>4.388802e+06</td>
      <td>POLYGON ((-2.27748 57.09527, -2.27644 57.09521...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01006507</td>
      <td>Culter - 02</td>
      <td>836</td>
      <td>836</td>
      <td>364</td>
      <td>22.349739</td>
      <td>0.223498</td>
      <td>2900.406362</td>
      <td>2.217468e+05</td>
      <td>POLYGON ((-2.27354 57.10449, -2.27333 57.10449...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01006508</td>
      <td>Culter - 03</td>
      <td>643</td>
      <td>643</td>
      <td>340</td>
      <td>27.019476</td>
      <td>0.270194</td>
      <td>3468.761949</td>
      <td>2.701948e+05</td>
      <td>POLYGON ((-2.27443 57.10171, -2.27237 57.10046...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01006509</td>
      <td>Culter - 04</td>
      <td>580</td>
      <td>580</td>
      <td>274</td>
      <td>9.625426</td>
      <td>0.096254</td>
      <td>1647.461389</td>
      <td>9.625426e+04</td>
      <td>POLYGON ((-2.26611 57.10133, -2.26599 57.10092...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01006510</td>
      <td>Culter - 05</td>
      <td>644</td>
      <td>577</td>
      <td>256</td>
      <td>18.007657</td>
      <td>0.180076</td>
      <td>3026.111412</td>
      <td>1.800766e+05</td>
      <td>POLYGON ((-2.26013 57.10160, -2.26050 57.10134...</td>
    </tr>
  </tbody>
</table>
</div>



So, as you can our `datazones` dataset contains `6,976` rows which represent "datazones" as defined and included as part of the Scottish Government Index of Multiple Deprivation (SIMD) and we have some basic metadata associated with these datazones, in particular the area of each zone which we will be using to calculate the density of Airbnb property listings.


```python
type(datazones)
```




    geopandas.geodataframe.GeoDataFrame



Our `datazones` dataset has been loaded from a shapefile using geopandas and is a `GeoDataFrame` - we have a `geometry` column which includes the Polygon geomtery of each of the 6,976 data zones. We can visualize in just one line of code :


```python
datazones.plot()
```




    <AxesSubplot: >




    
![png](output_13_1.png)
    


#### Non-spatial data


```python
### create a non-spatial DataFrame from the listings data
edin_listings = pd.read_csv('data/edinburgh_airbnb_listings.csv')
print(f"Number of rows: {len(edin_listings)}")
```

    Number of rows: 8044



```python
edin_listings.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



So we have 8044 rows which represent Edinburgh Airbnb listings as at 11 September 2023. The data was sourced from [Inside Airbnb](http://insideairbnb.com/get-the-data).


```python
type(edin_listings)
```




    pandas.core.frame.DataFrame



### Convert the `pandas.DataFrame` into a `geopandas.GeoDataFrame`

Although our Airbnb listings dataset has a `longitude` and a `latitude` column this is not enough for it to qualify as a GeoDataFrame. However we can make use of this information to create a `geometry` column.

We want to create a `shapely.geometry.Point` *for each row*, based on the columns `longitude` and `latitude`.
There are different approaches to this task. One such method is the [`apply()` method](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.apply.html) of the data frame, together with a *lambda function*.


```python
from shapely.geometry import Point

# Create a column of empty Point objects
edin_listings['point_geometry'] = [Point() for _ in range(len(edin_listings))]

# Use a lambda function to create Point objects and assign them to the 'point_geometry' column
edin_listings['point_geometry'] = edin_listings.apply(lambda row: Point(row['longitude'], row['latitude']), axis=1)

edin_listings.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>point_geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
      <td>POINT (-3.185293348616568 55.94498320836646)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
      <td>POINT (-3.19641 55.9548)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
  </tbody>
</table>
</div>




```python
# explicitly specify the geometry column and the coordinate system
edin_listings = gpd.GeoDataFrame(edin_listings, geometry=edin_listings['point_geometry'], crs="WGS84")
```


```python
type(edin_listings)
```




    geopandas.geodataframe.GeoDataFrame




```python
edin_listings.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
edin_listings
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>point_geometry</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
      <td>POINT (-3.18805 55.95759)</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
      <td>POINT (-3.185293348616568 55.94498320836646)</td>
      <td>POINT (-3.18529 55.94498)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
      <td>POINT (-3.18305 55.95072)</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
      <td>POINT (-3.19641 55.9548)</td>
      <td>POINT (-3.19641 55.95480)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.20863 55.94217)</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Rental unit in Leith · ★New · 1 bedroom · 2 be...</td>
      <td>131677538</td>
      <td>Conor</td>
      <td>NaN</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>Private room</td>
      <td>60</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>255</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1882747601883032 55.98675908383569)</td>
      <td>POINT (-3.18827 55.98676)</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Rental unit in Edinburgh · ★New · 4 bedrooms ·...</td>
      <td>146730114</td>
      <td>Adam</td>
      <td>NaN</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>Entire home/apt</td>
      <td>399</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9</td>
      <td>258</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1776279 55.9397141)</td>
      <td>POINT (-3.17763 55.93971)</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Rental unit in Edinburgh · ★New · 3 bedrooms ·...</td>
      <td>238856343</td>
      <td>Hany</td>
      <td>NaN</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>Entire home/apt</td>
      <td>268</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>20</td>
      <td>134</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.176967726675298 55.959687801369945)</td>
      <td>POINT (-3.17697 55.95969)</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Rental unit in Edinburgh · ★New · 1 bedroom · ...</td>
      <td>18970049</td>
      <td>Darcy</td>
      <td>NaN</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>Private room</td>
      <td>56</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>207</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1400765961073613 55.91798460572752)</td>
      <td>POINT (-3.14008 55.91798)</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Rental unit in Edinburgh · ★New · 2 bedrooms ·...</td>
      <td>284574597</td>
      <td>Luka</td>
      <td>NaN</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>Entire home/apt</td>
      <td>107</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.244545613737548 55.96690438333705)</td>
      <td>POINT (-3.24455 55.96690)</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 20 columns</p>
</div>



### Step 1 - Performing a spatial join

We have a Point geometry for each of the property listings. We have a Polygon geometry for each of the data zones. We are now ready to join our GeoDataFrames.


```python
airbnb_datazones = edin_listings.sjoin(datazones, how="left", predicate="within")
```


```python
airbnb_datazones.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>index_right</th>
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>2170</td>
      <td>S01008676</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>779</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>2174</td>
      <td>S01008680</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>631</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>2179</td>
      <td>S01008685</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>1025</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>2343</td>
      <td>S01008849</td>
      <td>New Town West - 01</td>
      <td>727</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>2145</td>
      <td>S01008651</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>965</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>



### Step 2: Calculate the number of property listings within each data zone

We can use `.groupby` to group the number of properties in each data zone :


```python
# group by DataZone and count # airbnb listings
count_listings = airbnb_datazones.groupby(['DataZone', 'Name', 'StdAreaKm2']).size().reset_index(name='Airbnb_Count')
count_listings.sort_values(by='Airbnb_Count',ascending=False)
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
      <th>DataZone</th>
      <th>Name</th>
      <th>StdAreaKm2</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>241</th>
      <td>S01008677</td>
      <td>Old Town, Princes Street and Leith Street - 04</td>
      <td>0.438908</td>
      <td>218</td>
    </tr>
    <tr>
      <th>429</th>
      <td>S01008868</td>
      <td>Deans Village - 01</td>
      <td>0.284072</td>
      <td>191</td>
    </tr>
    <tr>
      <th>239</th>
      <td>S01008675</td>
      <td>Old Town, Princes Street and Leith Street - 02</td>
      <td>0.200589</td>
      <td>159</td>
    </tr>
    <tr>
      <th>410</th>
      <td>S01008849</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
    </tr>
    <tr>
      <th>240</th>
      <td>S01008676</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>S01008533</td>
      <td>Comiston and Swanston - 01</td>
      <td>0.137717</td>
      <td>1</td>
    </tr>
    <tr>
      <th>278</th>
      <td>S01008715</td>
      <td>Bingham, Magdalene and The Christians - 04</td>
      <td>0.264941</td>
      <td>1</td>
    </tr>
    <tr>
      <th>533</th>
      <td>S01008978</td>
      <td>Carrick Knowe - 02</td>
      <td>0.072332</td>
      <td>1</td>
    </tr>
    <tr>
      <th>140</th>
      <td>S01008575</td>
      <td>Liberton East - 03</td>
      <td>0.230383</td>
      <td>1</td>
    </tr>
    <tr>
      <th>111</th>
      <td>S01008545</td>
      <td>Fairmilehead - 05</td>
      <td>0.403233</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>566 rows × 4 columns</p>
</div>




```python
number_of_listings = count_listings.Airbnb_Count.sum()
number_of_listings
```




    8044



So our Airbnb listings data covers 566 datazones - sanity check - 566 out of a total of 6976 for the whole of Scotland = 8.1%. Seems reasonable.


```python
# merge the count back into the original GeoDataFrame
airbnb_datazones = airbnb_datazones.merge(count_listings, on='DataZone', how='left')
airbnb_datazones.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>779</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>631</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>0.051285</td>
      <td>27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>1025</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>0.171215</td>
      <td>96</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>727</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>965</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>0.167184</td>
      <td>58</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 33 columns</p>
</div>



### Step 3 - Calculate the property listing density per data zone 


```python
# Step 3: Calculate the density
airbnb_datazones['Density'] = (airbnb_datazones['Airbnb_Count'] / airbnb_datazones['StdAreaKm2_x']).round(1)
```


```python
airbnb_datazones
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
      <td>410.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>0.051285</td>
      <td>27</td>
      <td>526.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>0.171215</td>
      <td>96</td>
      <td>560.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
      <td>656.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>0.167184</td>
      <td>58</td>
      <td>346.9</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Rental unit in Leith · ★New · 1 bedroom · 2 be...</td>
      <td>131677538</td>
      <td>Conor</td>
      <td>NaN</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>Private room</td>
      <td>60</td>
      <td>...</td>
      <td>855</td>
      <td>431</td>
      <td>36.770579</td>
      <td>0.367705</td>
      <td>2978.802850</td>
      <td>367705.759844</td>
      <td>Western Harbour and Leith Docks - 03</td>
      <td>0.367705</td>
      <td>44</td>
      <td>119.7</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Rental unit in Edinburgh · ★New · 4 bedrooms ·...</td>
      <td>146730114</td>
      <td>Adam</td>
      <td>NaN</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>Entire home/apt</td>
      <td>399</td>
      <td>...</td>
      <td>1121</td>
      <td>544</td>
      <td>5.294937</td>
      <td>0.052949</td>
      <td>1822.746662</td>
      <td>52949.359874</td>
      <td>Newington and Dalkeith Road - 05</td>
      <td>0.052949</td>
      <td>23</td>
      <td>434.4</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Rental unit in Edinburgh · ★New · 3 bedrooms ·...</td>
      <td>238856343</td>
      <td>Hany</td>
      <td>NaN</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>Entire home/apt</td>
      <td>268</td>
      <td>...</td>
      <td>1019</td>
      <td>558</td>
      <td>6.646255</td>
      <td>0.066463</td>
      <td>1711.652482</td>
      <td>66462.552191</td>
      <td>Hillside and Calton Hill - 01</td>
      <td>0.066463</td>
      <td>39</td>
      <td>586.8</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Rental unit in Edinburgh · ★New · 1 bedroom · ...</td>
      <td>18970049</td>
      <td>Darcy</td>
      <td>NaN</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>Private room</td>
      <td>56</td>
      <td>...</td>
      <td>817</td>
      <td>346</td>
      <td>20.206043</td>
      <td>0.202062</td>
      <td>2991.947016</td>
      <td>202060.428869</td>
      <td>Moredun and Craigour - 04</td>
      <td>0.202062</td>
      <td>3</td>
      <td>14.8</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Rental unit in Edinburgh · ★New · 2 bedrooms ·...</td>
      <td>284574597</td>
      <td>Luka</td>
      <td>NaN</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>Entire home/apt</td>
      <td>107</td>
      <td>...</td>
      <td>962</td>
      <td>463</td>
      <td>16.393302</td>
      <td>0.163932</td>
      <td>2193.767849</td>
      <td>163933.028716</td>
      <td>Drylaw - 05</td>
      <td>0.163932</td>
      <td>4</td>
      <td>24.4</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 34 columns</p>
</div>



So now we have a `Density` column added to our dataframe, which now gives us of all Airbnb listings in Edinburgh together with the name and geometry of the SIMD datazone that they fall within, and the density of Airbnb properties per datazone.



#### Calculate overall density index for Edinburgh


```python
number_of_listings
```




    8044




```python
edinburgh_area = count_listings.StdAreaKm2.sum().round(1)
edinburgh_area
```




    245.9




```python
edinburgh_density = (number_of_listings / edinburgh_area).round(1)
edinburgh_density
```




    32.7



So the overall density of Airbnb listings as at 11 September 2023 was 32.7 per km2.

#### Rank datazones by Airbnb density

Let's drill down and take a look at the density at data zone level.


```python
density_ranking =airbnb_datazones.groupby('DataZone')[['Name_y', 'Density']].max().round(1)
density_ranking = density_ranking.sort_values(by='Density', ascending=False)
density_ranking
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
      <th>Name_y</th>
      <th>Density</th>
    </tr>
    <tr>
      <th>DataZone</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>S01008671</th>
      <td>Meadows and Southside - 06</td>
      <td>1201.6</td>
    </tr>
    <tr>
      <th>S01008812</th>
      <td>Hillside and Calton Hill - 07</td>
      <td>1201.1</td>
    </tr>
    <tr>
      <th>S01008662</th>
      <td>Tollcross - 04</td>
      <td>1147.1</td>
    </tr>
    <tr>
      <th>S01008664</th>
      <td>Tollcross - 06</td>
      <td>1082.1</td>
    </tr>
    <tr>
      <th>S01008679</th>
      <td>Old Town, Princes Street and Leith Street - 06</td>
      <td>1077.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>S01009002</th>
      <td>Dalmeny, Kirkliston and Newbridge - 06</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>S01008438</th>
      <td>Bonaly and The Pentlands - 01</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>S01009000</th>
      <td>Dalmeny, Kirkliston and Newbridge - 04</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>S01008997</th>
      <td>Dalmeny, Kirkliston and Newbridge - 01</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>S01008418</th>
      <td>Balerno and Bonnington Village - 02</td>
      <td>0.1</td>
    </tr>
  </tbody>
</table>
<p>566 rows × 2 columns</p>
</div>



We can see that the datazone with the highest density index is `Meadows and Southside - 06`.


```python
datazone_density = datazones.merge(airbnb_datazones, on='DataZone', how='inner')
datazone_density
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>ResPop2011_y</th>
      <th>HHCnt2011_y</th>
      <th>StdAreaHa_y</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng_y</th>
      <th>Shape_Area_y</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 43 columns</p>
</div>




```python
datazone_density.columns
```




    Index(['DataZone', 'Name', 'TotPop2011_x', 'ResPop2011_x', 'HHCnt2011_x',
           'StdAreaHa_x', 'StdAreaKm2', 'Shape_Leng_x', 'Shape_Area_x',
           'geometry_x', 'id', 'name', 'host_id', 'host_name',
           'neighbourhood_group', 'neighbourhood', 'latitude', 'longitude',
           'room_type', 'price', 'minimum_nights', 'number_of_reviews',
           'last_review', 'reviews_per_month', 'calculated_host_listings_count',
           'availability_365', 'number_of_reviews_ltm', 'license',
           'point_geometry', 'geometry_y', 'index_right', 'Name_x', 'TotPop2011_y',
           'ResPop2011_y', 'HHCnt2011_y', 'StdAreaHa_y', 'StdAreaKm2_x',
           'Shape_Leng_y', 'Shape_Area_y', 'Name_y', 'StdAreaKm2_y',
           'Airbnb_Count', 'Density'],
          dtype='object')




```python
cols_to_remove = ['point_geometry', 'geometry_y', 'index_right', 'Name_y', 'TotPop2011_y', 'ResPop2011_y', 'HHCnt2011_y', 'StdAreaHa_y', 'StdAreaKm2_y', 'Shape_Leng_y', 'Shape_Area_y']
```


```python
datazone_density_tidy = datazone_density.drop(columns=cols_to_remove)
datazone_density_tidy
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-07-07</td>
      <td>0.33</td>
      <td>1</td>
      <td>237</td>
      <td>1</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-06-20</td>
      <td>0.08</td>
      <td>17</td>
      <td>241</td>
      <td>2</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-08-28</td>
      <td>0.51</td>
      <td>2</td>
      <td>257</td>
      <td>8</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-09-06</td>
      <td>1.13</td>
      <td>2</td>
      <td>272</td>
      <td>11</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2021-10-10</td>
      <td>0.08</td>
      <td>1</td>
      <td>327</td>
      <td>0</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>320</td>
      <td>0</td>
      <td>NaN</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>2023-09-07</td>
      <td>0.49</td>
      <td>2</td>
      <td>0</td>
      <td>14</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>2023-09-02</td>
      <td>1.69</td>
      <td>2</td>
      <td>37</td>
      <td>8</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>2023-04-29</td>
      <td>0.15</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>2023-08-15</td>
      <td>0.36</td>
      <td>1</td>
      <td>73</td>
      <td>6</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 32 columns</p>
</div>




```python
type(datazone_density_tidy)
```




    pandas.core.frame.DataFrame




```python
datazone_airbnb_density = gpd.GeoDataFrame(datazone_density_tidy, geometry=datazone_density_tidy['geometry_x'], crs="WGS84")
```

### Step 4 - Plot a choropleth map

We now have enough information to plot a choropleth density map.

Now let’s see what it looks like without a classification scheme:


```python
plt.figure(figsize=(20,20))
```




    <Figure size 2000x2000 with 0 Axes>




    <Figure size 2000x2000 with 0 Axes>



```python
datazone_airbnb_density.plot(column="Density", cmap="OrRd", edgecolor="k", legend=True)
```




    <AxesSubplot: >




    
![png](output_60_1.png)
    


#### Classification by quantiles

`Quantiles` will create attractive maps that place an equal number of observations in each class: If you have 30 counties and 6 data classes, you’ll have 5 counties in each class. The problem with quantiles is that you can end up with classes that have very different numerical ranges (e.g., 1-4, 4-9, 9-250).


```python
# Splitting the data in four shows some spatial clustering around the center
datazone_airbnb_density.plot(
    column="Density", scheme="quantiles", k=12, cmap="OrRd", edgecolor="k")
```




    <AxesSubplot: >




    
![png](output_63_1.png)
    


#### Classificaton by natural breaks

`Natural Breaks` is a kind of “optimal” classification scheme that finds class breaks that will minimize within-class variance and maximize between-class differences. One drawback of this approach is each dataset generates a unique classification solution, and if you need to make comparison across maps, such as in an atlas or a series (e.g., one map each for 1980, 1990, 2000) you might want to use a single scheme that can be applied across all of the maps.


```python
# Compare this to the previous 3-bin figure with quantiles
datazone_airbnb_density.plot(
    column="Density",
    scheme="natural_breaks",
    k=12,
    cmap="OrRd",
    edgecolor="k",
    legend=False,
)
```




    <AxesSubplot: >




    
![png](output_66_1.png)
    


These maps are OK for obtaining a quick overview, but they are very crude and static. Let's add some functionality by harnessing Folium. 

### Interactive Map using Folium

[Folium](https://python-visualization.github.io/folium/latest/) makes it easy to visualize data that’s been manipulated in Python on an interactive leaflet map. It enables both the binding of data to a map for choropleth visualizations as well as passing rich vector/raster/HTML visualizations as markers on the map.

The library has a number of built-in tilesets from OpenStreetMap, Mapbox, and Stamen, and supports custom tilesets. Folium supports both Image, Video, GeoJSON and TopoJSON overlays and has a number of vector layers built-in.


```python
import folium
```


```python
# Create a Map instance
m = folium.Map(location=[55.9533, -3.1883], zoom_start=10, control_scale=True)
m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.2.0/css/all.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_4de7f276fe76bdef71967df546f50ecf {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
                .leaflet-container { font-size: 1rem; }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_4de7f276fe76bdef71967df546f50ecf&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_4de7f276fe76bdef71967df546f50ecf = L.map(
                &quot;map_4de7f276fe76bdef71967df546f50ecf&quot;,
                {
                    center: [55.9533, -3.1883],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_4de7f276fe76bdef71967df546f50ecf);





            var tile_layer_29e01124caa938ee2aa12f7d7e85cf4b = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca target=\&quot;_blank\&quot; href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca target=\&quot;_blank\&quot; href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_4de7f276fe76bdef71967df546f50ecf);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Let's now create a layer for our Edinburgh Airbnb listings : 


```python
edin_airbnb_listings = pd.read_csv('data/edinburgh_airbnb_listings.csv')
```


```python
# Create a column of empty Point objects
edin_airbnb_listings['geometry'] = [Point() for _ in range(len(edin_listings))]

# Use a lambda function to create Point objects and assign them to the 'point_geometry' column
edin_airbnb_listings['geometry'] = edin_airbnb_listings.apply(lambda row: Point(row['longitude'], row['latitude']), axis=1)
```


```python
edin_airbnb_listings = gpd.GeoDataFrame(edin_airbnb_listings, geometry=edin_airbnb_listings['geometry'], crs="WGS84")
```


```python
edin_airbnb_listings = edin_airbnb_listings[['id', 'neighbourhood', 'latitude', 'longitude', 'geometry']]
```


```python
edin_airbnb_listings
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
      <th>id</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>POINT (-3.18529 55.94498)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>POINT (-3.19641 55.95480)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>POINT (-3.18827 55.98676)</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>POINT (-3.17763 55.93971)</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>POINT (-3.17697 55.95969)</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>POINT (-3.14008 55.91798)</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>POINT (-3.24455 55.96690)</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 5 columns</p>
</div>




```python
type(edin_airbnb_listings)
```




    geopandas.geodataframe.GeoDataFrame



### Convert to GeoJSON

The input data used for creating Folium maps generally needs to be in GeoJson format. We can convert our shapefiles (stored as GeoDataFrames) to GeoJSON using `.features.GeoJson`. 


```python
edin_airbnb_listings_gjson = folium.features.GeoJson(edin_airbnb_listings, name='Airbnb listings')
```


```python
# Check the GeoJSON features
edin_airbnb_listings_gjson.data.get('features')[0]
```




    {'id': '0',
     'type': 'Feature',
     'properties': {'id': 15420,
      'neighbourhood': 'Old Town, Princes Street and Leith Street',
      'latitude': 55.95759,
      'longitude': -3.18805},
     'geometry': {'type': 'Point', 'coordinates': [-3.18805, 55.95759]},
     'bbox': [-3.18805, 55.95759, -3.18805, 55.95759]}



### Heatmap

Now that we have our Airbnb dataset in GeoJson format let's use Folium's mapping capabilitie.

[Folium plugins](https://python-visualization.github.io/folium/plugins.html) allow us to use popular plugins available in leaflet. One of these plugins is [HeatMap](https://python-visualization.github.io/folium/plugins.html#folium.plugins.HeatMap), which creates a heatmap layer from input points.

Let’s visualize a heatmap of the Edinburgh Airbnb listings. HeatMap requires a list of points, or a numpy array as input, so we need to first manipulate the data a bit:


```python
# Get x and y coordinates for each point
edin_airbnb_listings["x"] = edin_airbnb_listings["geometry"].apply(lambda geom: geom.x)
edin_airbnb_listings["y"] = edin_airbnb_listings["geometry"].apply(lambda geom: geom.y)

# Create a list of coordinate pairs
listings = list(zip(edin_airbnb_listings["y"], edin_airbnb_listings["x"]))
```


```python
from folium.plugins import HeatMap

# Create a Map instance
m = folium.Map(location=[55.953251, -3.188267], tiles='stamentoner', zoom_start=10, control_scale=True)

# Add heatmap to map instance
# Available parameters: HeatMap(data, name=None, min_opacity=0.5, max_zoom=18, max_val=1.0, radius=25, blur=15, gradient=None, overlay=True, control=True, show=True)
HeatMap(listings).add_to(m)

# Alternative syntax:
#m.add_child(HeatMap(points_array, radius=15))

# Show map
m
```

The red areas show the areas with a high concentration of Airbnb listings.

### Save our heatmap to html for sharing


```python
edinburgh_airbnb_heatmap = "output/edinburgh_airbnb_heatmap.html"
m.save(edinburgh_airbnb_heatmap)
```

### Choropleth map

Next, let’s check how we can overlay our `datazone` or *tract* map on top of a basemap using Folium’s choropleth method. This method is able to read the geometries and attributes directly from a geodataframe. 


```python
# Create a Geo-id which is needed by the Folium (it needs to have a unique identifier for each row)
datazone_density_tidy['geoid'] = datazone_density_tidy.index.astype(str)
```


```python
datazone_density_tidy
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
      <th>geoid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.33</td>
      <td>1</td>
      <td>237</td>
      <td>1</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.08</td>
      <td>17</td>
      <td>241</td>
      <td>2</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.51</td>
      <td>2</td>
      <td>257</td>
      <td>8</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>1.13</td>
      <td>2</td>
      <td>272</td>
      <td>11</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.08</td>
      <td>1</td>
      <td>327</td>
      <td>0</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>4</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>NaN</td>
      <td>1</td>
      <td>320</td>
      <td>0</td>
      <td>NaN</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
      <td>8039</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>0.49</td>
      <td>2</td>
      <td>0</td>
      <td>14</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
      <td>8040</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>1.69</td>
      <td>2</td>
      <td>37</td>
      <td>8</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
      <td>8041</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>0.15</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
      <td>8042</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>0.36</td>
      <td>1</td>
      <td>73</td>
      <td>6</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
      <td>8043</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 33 columns</p>
</div>




```python
# Select only needed columns
datazone_density_tidy = datazone_density_tidy[['geoid', 'Density', 'geometry_x', 'DataZone', 'Name_x', 'StdAreaKm2_x', 'Airbnb_Count']]

#check data
datazone_density_tidy.head()
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
      <th>geoid</th>
      <th>Density</th>
      <th>geometry_x</th>
      <th>DataZone</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
type(datazone_density_tidy)
```




    pandas.core.frame.DataFrame




```python
datazone_density_tidy = gpd.GeoDataFrame(datazone_density_tidy, geometry=datazone_density_tidy['geometry_x'], crs="WGS84")
```


```python
type(datazone_density_tidy)
```




    geopandas.geodataframe.GeoDataFrame




```python
datazone_density_tidy
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
      <th>geoid</th>
      <th>Density</th>
      <th>geometry_x</th>
      <th>DataZone</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>8039</td>
      <td>14.583566</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>8040</td>
      <td>16.772332</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>8041</td>
      <td>16.772332</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>8042</td>
      <td>2.633644</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>8043</td>
      <td>2.633644</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 8 columns</p>
</div>




```python
# Select only needed columns
datazone_density_tidy.drop(['geometry_x'],axis=1, inplace=True)
```


```python
# From review of the GeoJSON output earlier the key is 'properties.geoid'
key_on = 'properties.geoid'  # Adjust this path based on your GeoJSON structure
```


```python
# Create a Map instance
m = folium.Map(location=[55.953251, -3.188267], zoom_start=14, control_scale=True)

# Plot a choropleth map
# Notice: 'geoid' column that we created earlier needs to be assigned always as the first column
folium.Choropleth(
    geo_data=datazone_density_tidy.to_json(),
    name='Edinburgh Airbnb listings as at 11 September 2023',
    data=datazone_density_tidy,
    columns=['geoid', 'Density'],
    key_on=key_on,  # Use the correct key path here
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    line_color='white', 
    line_weight=0,
    highlight=False, 
    smooth_factor=1.0,
    threshold_scale=[0, 50, 100, 250, 500, 1000, 1202],
    legend_name= 'Edinburgh Airbnb listings density per datazone as at 11 September 2023 (km2)').add_to(m)

#Show map
m
```

### Display metadata using Tooltips

It is possible to add different kinds of pop-up messages and tooltips to the map. Here, it would be nice to see the name of datazone and the value of the density metreic when you hover the mouse over the map. Unfortunately this functionality is not apparently implemented in the Choropleth method we used before.

We can add tooltips to our map when plotting the polygons as GeoJson objects using the GeoJsonTooltip feature. (following examples from here and here)

For a quick workaround, we plot the polygons on top of the coropleth map as a transparent layer, and add the tooltip to these objects. 

>Note: this is not an optimal solution as now the polygon geometry get’s stored twice in the output!


```python
# Convert points to GeoJson
folium.features.GeoJson(datazone_density_tidy,  
                        name='Labels',
                        style_function=lambda x: {'color':'transparent','fillColor':'transparent','weight':0},
                        tooltip=folium.features.GeoJsonTooltip(fields=['Name_x','StdAreaKm2_x','Airbnb_Count','Density'],
                                                                aliases = ['Datazone','Area{km2)', '# Airbnb listings', 'Density'],
                                                                labels=True,
                                                                sticky=False
                                                                            )
                       ).add_to(m)

m
```

### Save our choropleth map to html for sharing


```python
edinburgh_airbnb = "output/edinburgh_airbnb_choropleth_map.html"
m.save(edinburgh_airbnb)
```

### Key Takeaways/Insights

This was a very rewarding project. I was able to enrich the granularity of the Inside Airbnb data (which categorized the property listings by 'Neighbourhood') to `data zone` level, which are a bit like tracts. By carrying out a spatial join I was able to locate which datazones (Polygons) the listings (Points) fell within.

The overall density index for Edinburgh Airbnb listings is 32.7 per km2 (8,044 listings covering an area of 245.9 km2). However, this does not tell the whole story. The top five data zones with the highest density of Airbnb listings are:

- Meadows and Southside - 06	`1201.6`
- Hillside and Calton Hill - 07	`1201.1`
- Tollcross - 04	`1147.1`
- Tollcross - 06	`1082.1`
- Old Town, Princes Street and Leith Street - 06	`1077.0`

It was also a good opportunity to make use of Folium which is a powerful and versatile, interactive mapping option.

    


#### Non-spatial data


```python
### create a non-spatial DataFrame from the listings data
edin_listings = pd.read_csv('data/edinburgh_airbnb_listings.csv')
print(f"Number of rows: {len(edin_listings)}")
```

    Number of rows: 8044



```python
edin_listings.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



So we have 8044 rows which represent Edinburgh Airbnb listings as at 11 September 2023. The data was sourced from [Inside Airbnb](http://insideairbnb.com/get-the-data).


```python
type(edin_listings)
```




    pandas.core.frame.DataFrame



### Convert the `pandas.DataFrame` into a `geopandas.GeoDataFrame`

Although our Airbnb listings dataset has a `longitude` and a `latitude` column this is not enough for it to qualify as a GeoDataFrame. However we can make use of this information to create a `geometry` column.

We want to create a `shapely.geometry.Point` *for each row*, based on the columns `longitude` and `latitude`.
There are different approaches to this task. One such method is the [`apply()` method](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.apply.html) of the data frame, together with a *lambda function*.


```python
from shapely.geometry import Point

# Create a column of empty Point objects
edin_listings['point_geometry'] = [Point() for _ in range(len(edin_listings))]

# Use a lambda function to create Point objects and assign them to the 'point_geometry' column
edin_listings['point_geometry'] = edin_listings.apply(lambda row: Point(row['longitude'], row['latitude']), axis=1)

edin_listings.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>point_geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
      <td>POINT (-3.185293348616568 55.94498320836646)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
      <td>POINT (-3.19641 55.9548)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
  </tbody>
</table>
</div>




```python
# explicitly specify the geometry column and the coordinate system
edin_listings = gpd.GeoDataFrame(edin_listings, geometry=edin_listings['point_geometry'], crs="WGS84")
```


```python
type(edin_listings)
```




    geopandas.geodataframe.GeoDataFrame




```python
edin_listings.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
edin_listings
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>minimum_nights</th>
      <th>number_of_reviews</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>point_geometry</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>3</td>
      <td>511</td>
      <td>2023-09-03</td>
      <td>3.32</td>
      <td>1</td>
      <td>16</td>
      <td>79</td>
      <td>NaN</td>
      <td>POINT (-3.18805 55.95759)</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>3</td>
      <td>286</td>
      <td>2023-07-24</td>
      <td>1.81</td>
      <td>1</td>
      <td>42</td>
      <td>55</td>
      <td>NaN</td>
      <td>POINT (-3.185293348616568 55.94498320836646)</td>
      <td>POINT (-3.18529 55.94498)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>2</td>
      <td>979</td>
      <td>2023-09-08</td>
      <td>6.37</td>
      <td>1</td>
      <td>69</td>
      <td>68</td>
      <td>NaN</td>
      <td>POINT (-3.18305 55.95072)</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>4</td>
      <td>51</td>
      <td>2023-08-23</td>
      <td>0.33</td>
      <td>14</td>
      <td>2</td>
      <td>32</td>
      <td>NaN</td>
      <td>POINT (-3.19641 55.9548)</td>
      <td>POINT (-3.19641 55.95480)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>1</td>
      <td>43</td>
      <td>2022-07-03</td>
      <td>0.32</td>
      <td>3</td>
      <td>175</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.20863 55.94217)</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Rental unit in Leith · ★New · 1 bedroom · 2 be...</td>
      <td>131677538</td>
      <td>Conor</td>
      <td>NaN</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>Private room</td>
      <td>60</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>255</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1882747601883032 55.98675908383569)</td>
      <td>POINT (-3.18827 55.98676)</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Rental unit in Edinburgh · ★New · 4 bedrooms ·...</td>
      <td>146730114</td>
      <td>Adam</td>
      <td>NaN</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>Entire home/apt</td>
      <td>399</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9</td>
      <td>258</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1776279 55.9397141)</td>
      <td>POINT (-3.17763 55.93971)</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Rental unit in Edinburgh · ★New · 3 bedrooms ·...</td>
      <td>238856343</td>
      <td>Hany</td>
      <td>NaN</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>Entire home/apt</td>
      <td>268</td>
      <td>2</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>20</td>
      <td>134</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.176967726675298 55.959687801369945)</td>
      <td>POINT (-3.17697 55.95969)</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Rental unit in Edinburgh · ★New · 1 bedroom · ...</td>
      <td>18970049</td>
      <td>Darcy</td>
      <td>NaN</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>Private room</td>
      <td>56</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>207</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.1400765961073613 55.91798460572752)</td>
      <td>POINT (-3.14008 55.91798)</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Rental unit in Edinburgh · ★New · 2 bedrooms ·...</td>
      <td>284574597</td>
      <td>Luka</td>
      <td>NaN</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>Entire home/apt</td>
      <td>107</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>POINT (-3.244545613737548 55.96690438333705)</td>
      <td>POINT (-3.24455 55.96690)</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 20 columns</p>
</div>



### Step 1 - Performing a spatial join

We have a Point geometry for each of the property listings. We have a Polygon geometry for each of the data zones. We are now ready to join our GeoDataFrames.


```python
airbnb_datazones = edin_listings.sjoin(datazones, how="left", predicate="within")
```


```python
airbnb_datazones.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>index_right</th>
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>2170</td>
      <td>S01008676</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>779</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>2174</td>
      <td>S01008680</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>631</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>2179</td>
      <td>S01008685</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>1025</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>2343</td>
      <td>S01008849</td>
      <td>New Town West - 01</td>
      <td>727</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>2145</td>
      <td>S01008651</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>965</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>



### Step 2: Calculate the number of property listings within each data zone

We can use `.groupby` to group the number of properties in each data zone :


```python
# group by DataZone and count # airbnb listings
count_listings = airbnb_datazones.groupby(['DataZone', 'Name', 'StdAreaKm2']).size().reset_index(name='Airbnb_Count')
count_listings.sort_values(by='Airbnb_Count',ascending=False)
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
      <th>DataZone</th>
      <th>Name</th>
      <th>StdAreaKm2</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>241</th>
      <td>S01008677</td>
      <td>Old Town, Princes Street and Leith Street - 04</td>
      <td>0.438908</td>
      <td>218</td>
    </tr>
    <tr>
      <th>429</th>
      <td>S01008868</td>
      <td>Deans Village - 01</td>
      <td>0.284072</td>
      <td>191</td>
    </tr>
    <tr>
      <th>239</th>
      <td>S01008675</td>
      <td>Old Town, Princes Street and Leith Street - 02</td>
      <td>0.200589</td>
      <td>159</td>
    </tr>
    <tr>
      <th>410</th>
      <td>S01008849</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
    </tr>
    <tr>
      <th>240</th>
      <td>S01008676</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>S01008533</td>
      <td>Comiston and Swanston - 01</td>
      <td>0.137717</td>
      <td>1</td>
    </tr>
    <tr>
      <th>278</th>
      <td>S01008715</td>
      <td>Bingham, Magdalene and The Christians - 04</td>
      <td>0.264941</td>
      <td>1</td>
    </tr>
    <tr>
      <th>533</th>
      <td>S01008978</td>
      <td>Carrick Knowe - 02</td>
      <td>0.072332</td>
      <td>1</td>
    </tr>
    <tr>
      <th>140</th>
      <td>S01008575</td>
      <td>Liberton East - 03</td>
      <td>0.230383</td>
      <td>1</td>
    </tr>
    <tr>
      <th>111</th>
      <td>S01008545</td>
      <td>Fairmilehead - 05</td>
      <td>0.403233</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>566 rows × 4 columns</p>
</div>




```python
number_of_listings = count_listings.Airbnb_Count.sum()
number_of_listings
```




    8044



So our Airbnb listings data covers 566 datazones - sanity check - 566 out of a total of 6976 for the whole of Scotland = 8.1%. Seems reasonable.


```python
# merge the count back into the original GeoDataFrame
airbnb_datazones = airbnb_datazones.merge(count_listings, on='DataZone', how='left')
airbnb_datazones.head()
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>TotPop2011</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>779</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>631</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>0.051285</td>
      <td>27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>1025</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>0.171215</td>
      <td>96</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>727</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>965</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>0.167184</td>
      <td>58</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 33 columns</p>
</div>



### Step 3 - Calculate the property listing density per data zone 


```python
# Step 3: Calculate the density
airbnb_datazones['Density'] = (airbnb_datazones['Airbnb_Count'] / airbnb_datazones['StdAreaKm2_x']).round(1)
```


```python
airbnb_datazones
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
      <th>id</th>
      <th>name</th>
      <th>host_id</th>
      <th>host_name</th>
      <th>neighbourhood_group</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>room_type</th>
      <th>price</th>
      <th>...</th>
      <th>ResPop2011</th>
      <th>HHCnt2011</th>
      <th>StdAreaHa</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Rental unit in Edinburgh · ★4.97 · 1 bedroom ·...</td>
      <td>60423</td>
      <td>Charlotte</td>
      <td>NaN</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>Entire home/apt</td>
      <td>130</td>
      <td>...</td>
      <td>690</td>
      <td>338</td>
      <td>29.476317</td>
      <td>0.294764</td>
      <td>4133.369517</td>
      <td>294763.155287</td>
      <td>Old Town, Princes Street and Leith Street - 03</td>
      <td>0.294764</td>
      <td>121</td>
      <td>410.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Loft in Edinburgh · ★4.62 · 2 bedrooms · 2 bed...</td>
      <td>46498</td>
      <td>Gordon</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>Entire home/apt</td>
      <td>91</td>
      <td>...</td>
      <td>597</td>
      <td>326</td>
      <td>5.128574</td>
      <td>0.051285</td>
      <td>1473.321749</td>
      <td>51285.748615</td>
      <td>Canongate, Southside and Dumbiedykes - 01</td>
      <td>0.051285</td>
      <td>27</td>
      <td>526.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Condo in Edinburgh · ★4.86 · 1 bedroom · 2 bed...</td>
      <td>221474</td>
      <td>Mark</td>
      <td>NaN</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>Private room</td>
      <td>129</td>
      <td>...</td>
      <td>777</td>
      <td>489</td>
      <td>17.121499</td>
      <td>0.171215</td>
      <td>2976.316012</td>
      <td>171214.990188</td>
      <td>Canongate, Southside and Dumbiedykes - 06</td>
      <td>0.171215</td>
      <td>96</td>
      <td>560.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>Rental unit in Edinburgh · ★4.76 · 2 bedrooms ...</td>
      <td>236828</td>
      <td>Francois</td>
      <td>NaN</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>Entire home/apt</td>
      <td>170</td>
      <td>...</td>
      <td>727</td>
      <td>396</td>
      <td>20.245548</td>
      <td>0.202455</td>
      <td>3294.418369</td>
      <td>202455.476170</td>
      <td>New Town West - 01</td>
      <td>0.202455</td>
      <td>133</td>
      <td>656.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Boat in Edinburgh · ★4.61 · 3 bedrooms · 2 bed...</td>
      <td>253850</td>
      <td>Natalie</td>
      <td>NaN</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>Entire home/apt</td>
      <td>225</td>
      <td>...</td>
      <td>686</td>
      <td>497</td>
      <td>16.718419</td>
      <td>0.167184</td>
      <td>3153.946787</td>
      <td>167184.197335</td>
      <td>Dalry and Fountainbridge - 01</td>
      <td>0.167184</td>
      <td>58</td>
      <td>346.9</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Rental unit in Leith · ★New · 1 bedroom · 2 be...</td>
      <td>131677538</td>
      <td>Conor</td>
      <td>NaN</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>Private room</td>
      <td>60</td>
      <td>...</td>
      <td>855</td>
      <td>431</td>
      <td>36.770579</td>
      <td>0.367705</td>
      <td>2978.802850</td>
      <td>367705.759844</td>
      <td>Western Harbour and Leith Docks - 03</td>
      <td>0.367705</td>
      <td>44</td>
      <td>119.7</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Rental unit in Edinburgh · ★New · 4 bedrooms ·...</td>
      <td>146730114</td>
      <td>Adam</td>
      <td>NaN</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>Entire home/apt</td>
      <td>399</td>
      <td>...</td>
      <td>1121</td>
      <td>544</td>
      <td>5.294937</td>
      <td>0.052949</td>
      <td>1822.746662</td>
      <td>52949.359874</td>
      <td>Newington and Dalkeith Road - 05</td>
      <td>0.052949</td>
      <td>23</td>
      <td>434.4</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Rental unit in Edinburgh · ★New · 3 bedrooms ·...</td>
      <td>238856343</td>
      <td>Hany</td>
      <td>NaN</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>Entire home/apt</td>
      <td>268</td>
      <td>...</td>
      <td>1019</td>
      <td>558</td>
      <td>6.646255</td>
      <td>0.066463</td>
      <td>1711.652482</td>
      <td>66462.552191</td>
      <td>Hillside and Calton Hill - 01</td>
      <td>0.066463</td>
      <td>39</td>
      <td>586.8</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Rental unit in Edinburgh · ★New · 1 bedroom · ...</td>
      <td>18970049</td>
      <td>Darcy</td>
      <td>NaN</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>Private room</td>
      <td>56</td>
      <td>...</td>
      <td>817</td>
      <td>346</td>
      <td>20.206043</td>
      <td>0.202062</td>
      <td>2991.947016</td>
      <td>202060.428869</td>
      <td>Moredun and Craigour - 04</td>
      <td>0.202062</td>
      <td>3</td>
      <td>14.8</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Rental unit in Edinburgh · ★New · 2 bedrooms ·...</td>
      <td>284574597</td>
      <td>Luka</td>
      <td>NaN</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>Entire home/apt</td>
      <td>107</td>
      <td>...</td>
      <td>962</td>
      <td>463</td>
      <td>16.393302</td>
      <td>0.163932</td>
      <td>2193.767849</td>
      <td>163933.028716</td>
      <td>Drylaw - 05</td>
      <td>0.163932</td>
      <td>4</td>
      <td>24.4</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 34 columns</p>
</div>



So now we have a `Density` column added to our dataframe, which now gives us of all Airbnb listings in Edinburgh together with the name and geometry of the SIMD datazone that they fall within, and the density of Airbnb properties per datazone.



#### Calculate overall density index for Edinburgh


```python
number_of_listings
```




    8044




```python
edinburgh_area = count_listings.StdAreaKm2.sum().round(1)
edinburgh_area
```




    245.9




```python
edinburgh_density = (number_of_listings / edinburgh_area).round(1)
edinburgh_density
```




    32.7



So the overall density of Airbnb listings as at 11 September 2023 was 32.7 per km2.

#### Rank datazones by Airbnb density

Let's drill down and take a look at the density at data zone level.


```python
density_ranking =airbnb_datazones.groupby('DataZone')[['Name_y', 'Density']].max().round(1)
density_ranking = density_ranking.sort_values(by='Density', ascending=False)
density_ranking
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
      <th>Name_y</th>
      <th>Density</th>
    </tr>
    <tr>
      <th>DataZone</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>S01008671</th>
      <td>Meadows and Southside - 06</td>
      <td>1201.6</td>
    </tr>
    <tr>
      <th>S01008812</th>
      <td>Hillside and Calton Hill - 07</td>
      <td>1201.1</td>
    </tr>
    <tr>
      <th>S01008662</th>
      <td>Tollcross - 04</td>
      <td>1147.1</td>
    </tr>
    <tr>
      <th>S01008664</th>
      <td>Tollcross - 06</td>
      <td>1082.1</td>
    </tr>
    <tr>
      <th>S01008679</th>
      <td>Old Town, Princes Street and Leith Street - 06</td>
      <td>1077.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>S01009002</th>
      <td>Dalmeny, Kirkliston and Newbridge - 06</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>S01008438</th>
      <td>Bonaly and The Pentlands - 01</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>S01009000</th>
      <td>Dalmeny, Kirkliston and Newbridge - 04</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>S01008997</th>
      <td>Dalmeny, Kirkliston and Newbridge - 01</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>S01008418</th>
      <td>Balerno and Bonnington Village - 02</td>
      <td>0.1</td>
    </tr>
  </tbody>
</table>
<p>566 rows × 2 columns</p>
</div>



We can see that the datazone with the highest density index is `Meadows and Southside - 06`.


```python
datazone_density = datazones.merge(airbnb_datazones, on='DataZone', how='inner')
datazone_density
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>ResPop2011_y</th>
      <th>HHCnt2011_y</th>
      <th>StdAreaHa_y</th>
      <th>StdAreaKm2_x</th>
      <th>Shape_Leng_y</th>
      <th>Shape_Area_y</th>
      <th>Name_y</th>
      <th>StdAreaKm2_y</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 43 columns</p>
</div>




```python
datazone_density.columns
```




    Index(['DataZone', 'Name', 'TotPop2011_x', 'ResPop2011_x', 'HHCnt2011_x',
           'StdAreaHa_x', 'StdAreaKm2', 'Shape_Leng_x', 'Shape_Area_x',
           'geometry_x', 'id', 'name', 'host_id', 'host_name',
           'neighbourhood_group', 'neighbourhood', 'latitude', 'longitude',
           'room_type', 'price', 'minimum_nights', 'number_of_reviews',
           'last_review', 'reviews_per_month', 'calculated_host_listings_count',
           'availability_365', 'number_of_reviews_ltm', 'license',
           'point_geometry', 'geometry_y', 'index_right', 'Name_x', 'TotPop2011_y',
           'ResPop2011_y', 'HHCnt2011_y', 'StdAreaHa_y', 'StdAreaKm2_x',
           'Shape_Leng_y', 'Shape_Area_y', 'Name_y', 'StdAreaKm2_y',
           'Airbnb_Count', 'Density'],
          dtype='object')




```python
cols_to_remove = ['point_geometry', 'geometry_y', 'index_right', 'Name_y', 'TotPop2011_y', 'ResPop2011_y', 'HHCnt2011_y', 'StdAreaHa_y', 'StdAreaKm2_y', 'Shape_Leng_y', 'Shape_Area_y']
```


```python
datazone_density_tidy = datazone_density.drop(columns=cols_to_remove)
datazone_density_tidy
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>last_review</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-07-07</td>
      <td>0.33</td>
      <td>1</td>
      <td>237</td>
      <td>1</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-06-20</td>
      <td>0.08</td>
      <td>17</td>
      <td>241</td>
      <td>2</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-08-28</td>
      <td>0.51</td>
      <td>2</td>
      <td>257</td>
      <td>8</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2023-09-06</td>
      <td>1.13</td>
      <td>2</td>
      <td>272</td>
      <td>11</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>2021-10-10</td>
      <td>0.08</td>
      <td>1</td>
      <td>327</td>
      <td>0</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>320</td>
      <td>0</td>
      <td>NaN</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>2023-09-07</td>
      <td>0.49</td>
      <td>2</td>
      <td>0</td>
      <td>14</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>2023-09-02</td>
      <td>1.69</td>
      <td>2</td>
      <td>37</td>
      <td>8</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>2023-04-29</td>
      <td>0.15</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>2023-08-15</td>
      <td>0.36</td>
      <td>1</td>
      <td>73</td>
      <td>6</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 32 columns</p>
</div>




```python
type(datazone_density_tidy)
```




    pandas.core.frame.DataFrame




```python
datazone_airbnb_density = gpd.GeoDataFrame(datazone_density_tidy, geometry=datazone_density_tidy['geometry_x'], crs="WGS84")
```

### Step 4 - Plot a choropleth map

We now have enough information to plot a choropleth density map.

Now let’s see what it looks like without a classification scheme:


```python
plt.figure(figsize=(20,20))
```




    <Figure size 2000x2000 with 0 Axes>




    <Figure size 2000x2000 with 0 Axes>



```python
datazone_airbnb_density.plot(column="Density", cmap="OrRd", edgecolor="k", legend=True)
```




    <AxesSubplot: >




    
![png](images/output_60_1.png)
    


#### Classification by quantiles

`Quantiles` will create attractive maps that place an equal number of observations in each class: If you have 30 counties and 6 data classes, you’ll have 5 counties in each class. The problem with quantiles is that you can end up with classes that have very different numerical ranges (e.g., 1-4, 4-9, 9-250).


```python
# Splitting the data in four shows some spatial clustering around the center
datazone_airbnb_density.plot(
    column="Density", scheme="quantiles", k=12, cmap="OrRd", edgecolor="k")
```




    <AxesSubplot: >




    
![png](images/output_63_1.png)
    


#### Classificaton by natural breaks

`Natural Breaks` is a kind of “optimal” classification scheme that finds class breaks that will minimize within-class variance and maximize between-class differences. One drawback of this approach is each dataset generates a unique classification solution, and if you need to make comparison across maps, such as in an atlas or a series (e.g., one map each for 1980, 1990, 2000) you might want to use a single scheme that can be applied across all of the maps.


```python
# Compare this to the previous 3-bin figure with quantiles
datazone_airbnb_density.plot(
    column="Density",
    scheme="natural_breaks",
    k=12,
    cmap="OrRd",
    edgecolor="k",
    legend=False,
)
```




    <AxesSubplot: >




    
![png](images/output_66_1.png)
    


These maps are OK for obtaining a quick overview, but they are very crude and static. Let's add some functionality by harnessing Folium. 

### Interactive Map using Folium

[Folium](https://python-visualization.github.io/folium/latest/) makes it easy to visualize data that’s been manipulated in Python on an interactive leaflet map. It enables both the binding of data to a map for choropleth visualizations as well as passing rich vector/raster/HTML visualizations as markers on the map.

The library has a number of built-in tilesets from OpenStreetMap, Mapbox, and Stamen, and supports custom tilesets. Folium supports both Image, Video, GeoJSON and TopoJSON overlays and has a number of vector layers built-in.


```python
import folium
```


```python
# Create a Map instance
m = folium.Map(location=[55.9533, -3.1883], zoom_start=10, control_scale=True)
m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.2.0/css/all.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_4de7f276fe76bdef71967df546f50ecf {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
                .leaflet-container { font-size: 1rem; }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_4de7f276fe76bdef71967df546f50ecf&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_4de7f276fe76bdef71967df546f50ecf = L.map(
                &quot;map_4de7f276fe76bdef71967df546f50ecf&quot;,
                {
                    center: [55.9533, -3.1883],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_4de7f276fe76bdef71967df546f50ecf);





            var tile_layer_29e01124caa938ee2aa12f7d7e85cf4b = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca target=\&quot;_blank\&quot; href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca target=\&quot;_blank\&quot; href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_4de7f276fe76bdef71967df546f50ecf);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Let's now create a layer for our Edinburgh Airbnb listings : 


```python
edin_airbnb_listings = pd.read_csv('data/edinburgh_airbnb_listings.csv')
```


```python
# Create a column of empty Point objects
edin_airbnb_listings['geometry'] = [Point() for _ in range(len(edin_listings))]

# Use a lambda function to create Point objects and assign them to the 'point_geometry' column
edin_airbnb_listings['geometry'] = edin_airbnb_listings.apply(lambda row: Point(row['longitude'], row['latitude']), axis=1)
```


```python
edin_airbnb_listings = gpd.GeoDataFrame(edin_airbnb_listings, geometry=edin_airbnb_listings['geometry'], crs="WGS84")
```


```python
edin_airbnb_listings = edin_airbnb_listings[['id', 'neighbourhood', 'latitude', 'longitude', 'geometry']]
```


```python
edin_airbnb_listings
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
      <th>id</th>
      <th>neighbourhood</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15420</td>
      <td>Old Town, Princes Street and Leith Street</td>
      <td>55.957590</td>
      <td>-3.188050</td>
      <td>POINT (-3.18805 55.95759)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24288</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.944983</td>
      <td>-3.185293</td>
      <td>POINT (-3.18529 55.94498)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48645</td>
      <td>Canongate, Southside and Dumbiedykes</td>
      <td>55.950720</td>
      <td>-3.183050</td>
      <td>POINT (-3.18305 55.95072)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51505</td>
      <td>New Town West</td>
      <td>55.954800</td>
      <td>-3.196410</td>
      <td>POINT (-3.19641 55.95480)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>54188</td>
      <td>Dalry and Fountainbridge</td>
      <td>55.942170</td>
      <td>-3.208630</td>
      <td>POINT (-3.20863 55.94217)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>974513574116367354</td>
      <td>Western Harbour and Leith Docks</td>
      <td>55.986759</td>
      <td>-3.188275</td>
      <td>POINT (-3.18827 55.98676)</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>974540296775384595</td>
      <td>Newington and Dalkeith Road</td>
      <td>55.939714</td>
      <td>-3.177628</td>
      <td>POINT (-3.17763 55.93971)</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>975083364733576599</td>
      <td>Hillside and Calton Hill</td>
      <td>55.959688</td>
      <td>-3.176968</td>
      <td>POINT (-3.17697 55.95969)</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>976567080851790966</td>
      <td>Moredun and Craigour</td>
      <td>55.917985</td>
      <td>-3.140077</td>
      <td>POINT (-3.14008 55.91798)</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>977248060255868689</td>
      <td>Drylaw</td>
      <td>55.966904</td>
      <td>-3.244546</td>
      <td>POINT (-3.24455 55.96690)</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 5 columns</p>
</div>




```python
type(edin_airbnb_listings)
```




    geopandas.geodataframe.GeoDataFrame



### Convert to GeoJSON

The input data used for creating Folium maps generally needs to be in GeoJson format. We can convert our shapefiles (stored as GeoDataFrames) to GeoJSON using `.features.GeoJson`. 


```python
edin_airbnb_listings_gjson = folium.features.GeoJson(edin_airbnb_listings, name='Airbnb listings')
```


```python
# Check the GeoJSON features
edin_airbnb_listings_gjson.data.get('features')[0]
```




    {'id': '0',
     'type': 'Feature',
     'properties': {'id': 15420,
      'neighbourhood': 'Old Town, Princes Street and Leith Street',
      'latitude': 55.95759,
      'longitude': -3.18805},
     'geometry': {'type': 'Point', 'coordinates': [-3.18805, 55.95759]},
     'bbox': [-3.18805, 55.95759, -3.18805, 55.95759]}



### Heatmap

Now that we have our Airbnb dataset in GeoJson format let's use Folium's mapping capabilitie.

[Folium plugins](https://python-visualization.github.io/folium/plugins.html) allow us to use popular plugins available in leaflet. One of these plugins is [HeatMap](https://python-visualization.github.io/folium/plugins.html#folium.plugins.HeatMap), which creates a heatmap layer from input points.

Let’s visualize a heatmap of the Edinburgh Airbnb listings. HeatMap requires a list of points, or a numpy array as input, so we need to first manipulate the data a bit:


```python
# Get x and y coordinates for each point
edin_airbnb_listings["x"] = edin_airbnb_listings["geometry"].apply(lambda geom: geom.x)
edin_airbnb_listings["y"] = edin_airbnb_listings["geometry"].apply(lambda geom: geom.y)

# Create a list of coordinate pairs
listings = list(zip(edin_airbnb_listings["y"], edin_airbnb_listings["x"]))
```


```python
from folium.plugins import HeatMap

# Create a Map instance
m = folium.Map(location=[55.953251, -3.188267], tiles='stamentoner', zoom_start=10, control_scale=True)

# Add heatmap to map instance
# Available parameters: HeatMap(data, name=None, min_opacity=0.5, max_zoom=18, max_val=1.0, radius=25, blur=15, gradient=None, overlay=True, control=True, show=True)
HeatMap(listings).add_to(m)

# Alternative syntax:
#m.add_child(HeatMap(points_array, radius=15))

# Show map
m
```

The red areas show the areas with a high concentration of Airbnb listings.

### Save our heatmap to html for sharing


```python
edinburgh_airbnb_heatmap = "output/edinburgh_airbnb_heatmap.html"
m.save(edinburgh_airbnb_heatmap)
```

### Choropleth map

Next, let’s check how we can overlay our `datazone` or *tract* map on top of a basemap using Folium’s choropleth method. This method is able to read the geometries and attributes directly from a geodataframe. 


```python
# Create a Geo-id which is needed by the Folium (it needs to have a unique identifier for each row)
datazone_density_tidy['geoid'] = datazone_density_tidy.index.astype(str)
```


```python
datazone_density_tidy
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
      <th>DataZone</th>
      <th>Name</th>
      <th>TotPop2011_x</th>
      <th>ResPop2011_x</th>
      <th>HHCnt2011_x</th>
      <th>StdAreaHa_x</th>
      <th>StdAreaKm2</th>
      <th>Shape_Leng_x</th>
      <th>Shape_Area_x</th>
      <th>geometry_x</th>
      <th>...</th>
      <th>reviews_per_month</th>
      <th>calculated_host_listings_count</th>
      <th>availability_365</th>
      <th>number_of_reviews_ltm</th>
      <th>license</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>Density</th>
      <th>geoid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.33</td>
      <td>1</td>
      <td>237</td>
      <td>1</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.08</td>
      <td>17</td>
      <td>241</td>
      <td>2</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.51</td>
      <td>2</td>
      <td>257</td>
      <td>8</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>1.13</td>
      <td>2</td>
      <td>272</td>
      <td>11</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>726</td>
      <td>726</td>
      <td>299</td>
      <td>1029.993186</td>
      <td>10.299931</td>
      <td>20191.721420</td>
      <td>1.029993e+07</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>...</td>
      <td>0.08</td>
      <td>1</td>
      <td>327</td>
      <td>0</td>
      <td>NaN</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>0.485440</td>
      <td>4</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>1021</td>
      <td>1021</td>
      <td>404</td>
      <td>20.571212</td>
      <td>0.205711</td>
      <td>2549.606615</td>
      <td>2.057121e+05</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>...</td>
      <td>NaN</td>
      <td>1</td>
      <td>320</td>
      <td>0</td>
      <td>NaN</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>14.583566</td>
      <td>8039</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>0.49</td>
      <td>2</td>
      <td>0</td>
      <td>14</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
      <td>8040</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>752</td>
      <td>752</td>
      <td>296</td>
      <td>11.924532</td>
      <td>0.119244</td>
      <td>1694.365106</td>
      <td>1.192453e+05</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>...</td>
      <td>1.69</td>
      <td>2</td>
      <td>37</td>
      <td>8</td>
      <td>NaN</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>16.772332</td>
      <td>8041</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>0.15</td>
      <td>2</td>
      <td>18</td>
      <td>1</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
      <td>8042</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>893</td>
      <td>893</td>
      <td>363</td>
      <td>75.940344</td>
      <td>0.759404</td>
      <td>5408.285433</td>
      <td>7.594034e+05</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>...</td>
      <td>0.36</td>
      <td>1</td>
      <td>73</td>
      <td>6</td>
      <td>NaN</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>2.633644</td>
      <td>8043</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 33 columns</p>
</div>




```python
# Select only needed columns
datazone_density_tidy = datazone_density_tidy[['geoid', 'Density', 'geometry_x', 'DataZone', 'Name_x', 'StdAreaKm2_x', 'Airbnb_Count']]

#check data
datazone_density_tidy.head()
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
      <th>geoid</th>
      <th>Density</th>
      <th>geometry_x</th>
      <th>DataZone</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>0.48544</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
type(datazone_density_tidy)
```




    pandas.core.frame.DataFrame




```python
datazone_density_tidy = gpd.GeoDataFrame(datazone_density_tidy, geometry=datazone_density_tidy['geometry_x'], crs="WGS84")
```


```python
type(datazone_density_tidy)
```




    geopandas.geodataframe.GeoDataFrame




```python
datazone_density_tidy
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
      <th>geoid</th>
      <th>Density</th>
      <th>geometry_x</th>
      <th>DataZone</th>
      <th>Name_x</th>
      <th>StdAreaKm2_x</th>
      <th>Airbnb_Count</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>0.485440</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
      <td>S01008417</td>
      <td>Balerno and Bonnington Village - 01</td>
      <td>10.299931</td>
      <td>5</td>
      <td>POLYGON ((-3.35779 55.88156, -3.35752 55.88116...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>8039</th>
      <td>8039</td>
      <td>14.583566</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
      <td>S01009010</td>
      <td>Queensferry West - 02</td>
      <td>0.205711</td>
      <td>3</td>
      <td>POLYGON ((-3.40947 55.98996, -3.40948 55.98955...</td>
    </tr>
    <tr>
      <th>8040</th>
      <td>8040</td>
      <td>16.772332</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
    </tr>
    <tr>
      <th>8041</th>
      <td>8041</td>
      <td>16.772332</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
      <td>S01009012</td>
      <td>Queensferry West - 04</td>
      <td>0.119244</td>
      <td>2</td>
      <td>POLYGON ((-3.41584 55.98721, -3.41533 55.98683...</td>
    </tr>
    <tr>
      <th>8042</th>
      <td>8042</td>
      <td>2.633644</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
    </tr>
    <tr>
      <th>8043</th>
      <td>8043</td>
      <td>2.633644</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
      <td>S01009013</td>
      <td>Queensferry West - 05</td>
      <td>0.759404</td>
      <td>2</td>
      <td>POLYGON ((-3.41600 55.99385, -3.41582 55.99347...</td>
    </tr>
  </tbody>
</table>
<p>8044 rows × 8 columns</p>
</div>




```python
# Select only needed columns
datazone_density_tidy.drop(['geometry_x'],axis=1, inplace=True)
```


```python
# From review of the GeoJSON output earlier the key is 'properties.geoid'
key_on = 'properties.geoid'  # Adjust this path based on your GeoJSON structure
```


```python
# Create a Map instance
m = folium.Map(location=[55.953251, -3.188267], zoom_start=14, control_scale=True)

# Plot a choropleth map
# Notice: 'geoid' column that we created earlier needs to be assigned always as the first column
folium.Choropleth(
    geo_data=datazone_density_tidy.to_json(),
    name='Edinburgh Airbnb listings as at 11 September 2023',
    data=datazone_density_tidy,
    columns=['geoid', 'Density'],
    key_on=key_on,  # Use the correct key path here
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    line_color='white', 
    line_weight=0,
    highlight=False, 
    smooth_factor=1.0,
    threshold_scale=[0, 50, 100, 250, 500, 1000, 1202],
    legend_name= 'Edinburgh Airbnb listings density per datazone as at 11 September 2023 (km2)').add_to(m)

#Show map
m
```

### Display metadata using Tooltips

It is possible to add different kinds of pop-up messages and tooltips to the map. Here, it would be nice to see the name of datazone and the value of the density metreic when you hover the mouse over the map. Unfortunately this functionality is not apparently implemented in the Choropleth method we used before.

We can add tooltips to our map when plotting the polygons as GeoJson objects using the GeoJsonTooltip feature. (following examples from here and here)

For a quick workaround, we plot the polygons on top of the coropleth map as a transparent layer, and add the tooltip to these objects. 

>Note: this is not an optimal solution as now the polygon geometry get’s stored twice in the output!


```python
# Convert points to GeoJson
folium.features.GeoJson(datazone_density_tidy,  
                        name='Labels',
                        style_function=lambda x: {'color':'transparent','fillColor':'transparent','weight':0},
                        tooltip=folium.features.GeoJsonTooltip(fields=['Name_x','StdAreaKm2_x','Airbnb_Count','Density'],
                                                                aliases = ['Datazone','Area{km2)', '# Airbnb listings', 'Density'],
                                                                labels=True,
                                                                sticky=False
                                                                            )
                       ).add_to(m)

m
```

### Save our choropleth map to html for sharing


```python
edinburgh_airbnb = "output/edinburgh_airbnb_choropleth_map.html"
m.save(edinburgh_airbnb)
```

### Key Takeaways/Insights

This was a very rewarding project. I was able to enrich the granularity of the Inside Airbnb data (which categorized the property listings by 'Neighbourhood') to `data zone` level, which are a bit like tracts. By carrying out a spatial join I was able to locate which datazones (Polygons) the listings (Points) fell within.

The overall density index for Edinburgh Airbnb listings is 32.7 per km2 (8,044 listings covering an area of 245.9 km2). However, this does not tell the whole story. The top five data zones with the highest density of Airbnb listings are:

- Meadows and Southside - 06	`1201.6`
- Hillside and Calton Hill - 07	`1201.1`
- Tollcross - 04	`1147.1`
- Tollcross - 06	`1082.1`
- Old Town, Princes Street and Leith Street - 06	`1077.0`

It was also a good opportunity to make use of Folium which is a powerful and versatile, interactive mapping option.
