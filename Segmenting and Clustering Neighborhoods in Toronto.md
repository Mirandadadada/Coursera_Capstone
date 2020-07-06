```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np

!conda install -c conda-forge geopy --yes 
from geopy.geocoders import Nominatim # module to convert an address into latitude and longitude values

# libraries for displaying images
from IPython.display import Image 
from IPython.core.display import HTML 
    
# tranforming json file into a pandas dataframe library
from pandas.io.json import json_normalize

!conda install -c conda-forge folium=0.5.0 --yes
import folium # plotting library

print('Folium installed')
print('Libraries imported.')

```

    Collecting package metadata (current_repodata.json): ...working... done
    Solving environment: ...working... done
    
    ## Package Plan ##
    
      environment location: C:\Users\ruxin\anaconda3
    
      added / updated specs:
        - geopy
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        geographiclib-1.50         |             py_0          34 KB  conda-forge
        geopy-2.0.0                |     pyh9f0ad1d_0          63 KB  conda-forge
        ------------------------------------------------------------
                                               Total:          97 KB
    
    The following NEW packages will be INSTALLED:
    
      geographiclib      conda-forge/noarch::geographiclib-1.50-py_0
      geopy              conda-forge/noarch::geopy-2.0.0-pyh9f0ad1d_0
    
    
    
    Downloading and Extracting Packages
    
    geographiclib-1.50   | 34 KB     |            |   0% 
    geographiclib-1.50   | 34 KB     | ####7      |  47% 
    geographiclib-1.50   | 34 KB     | ########## | 100% 
    
    geopy-2.0.0          | 63 KB     |            |   0% 
    geopy-2.0.0          | 63 KB     | ########## | 100% 
    Preparing transaction: ...working... done
    Verifying transaction: ...working... done
    Executing transaction: ...working... done
    Collecting package metadata (current_repodata.json): ...working... done
    Solving environment: ...working... done
    
    # All requested packages already installed.
    
    Folium installed
    Libraries imported.
    

## **1. Scrap Table from Wikipedia**


```python
# Get the html Source
website_url = requests.get('https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M').text
soup = BeautifulSoup(website_url,'lxml')
my_table = soup.find('table',{'class':'wikitable sortable'})
table_rows = my_table.find_all('tr')

# Scrap the tabe from Wikipedia
data = []
for row in table_rows:
    data.append([t.text.strip() for t in row.find_all('td')])

# Transform the data into Pandas DataFrame
df = pd.DataFrame(data, columns=['PostalCode', 'Borough', 'Neighbourhood'])
df = df[~df['PostalCode'].isnull()]
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
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>M1A</td>
      <td>Not assigned</td>
      <td>Not assigned</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M2A</td>
      <td>Not assigned</td>
      <td>Not assigned</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M3A</td>
      <td>North York</td>
      <td>Parkwoods</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M4A</td>
      <td>North York</td>
      <td>Victoria Village</td>
    </tr>
    <tr>
      <th>5</th>
      <td>M5A</td>
      <td>Downtown Toronto</td>
      <td>Regent Park, Harbourfront</td>
    </tr>
  </tbody>
</table>
</div>



#### Delete the rows where Borough is 'Not Assigned'


```python
df = df[df.Borough != 'Not assigned'].reset_index()
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
      <th>index</th>
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>M3A</td>
      <td>North York</td>
      <td>Parkwoods</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>M4A</td>
      <td>North York</td>
      <td>Victoria Village</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>M5A</td>
      <td>Downtown Toronto</td>
      <td>Regent Park, Harbourfront</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>M6A</td>
      <td>North York</td>
      <td>Lawrence Manor, Lawrence Heights</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7</td>
      <td>M7A</td>
      <td>Downtown Toronto</td>
      <td>Queen's Park, Ontario Provincial Government</td>
    </tr>
  </tbody>
</table>
</div>



#### Combine all neighbourhoods where postal code are the same


```python
df = df.groupby('PostalCode').agg(lambda x: ','.join(x))
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
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
    <tr>
      <th>PostalCode</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>M1B</th>
      <td>Scarborough</td>
      <td>Malvern, Rouge</td>
    </tr>
    <tr>
      <th>M1C</th>
      <td>Scarborough</td>
      <td>Rouge Hill, Port Union, Highland Creek</td>
    </tr>
    <tr>
      <th>M1E</th>
      <td>Scarborough</td>
      <td>Guildwood, Morningside, West Hill</td>
    </tr>
    <tr>
      <th>M1G</th>
      <td>Scarborough</td>
      <td>Woburn</td>
    </tr>
    <tr>
      <th>M1H</th>
      <td>Scarborough</td>
      <td>Cedarbrae</td>
    </tr>
  </tbody>
</table>
</div>



#### If neighbourhood is not assigned, replace it by borough name


```python
df.loc[df['Neighbourhood']=="Not assigned",'Neighbourhood']=df.loc[df['Neighbourhood']=="Not assigned",'Borough']
df = df.reset_index()
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
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M1B</td>
      <td>Scarborough</td>
      <td>Malvern, Rouge</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M1C</td>
      <td>Scarborough</td>
      <td>Rouge Hill, Port Union, Highland Creek</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M1E</td>
      <td>Scarborough</td>
      <td>Guildwood, Morningside, West Hill</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M1G</td>
      <td>Scarborough</td>
      <td>Woburn</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M1H</td>
      <td>Scarborough</td>
      <td>Cedarbrae</td>
    </tr>
  </tbody>
</table>
</div>



#### Get the shape of the dataframe


```python
df.shape
```




    (103, 3)



## **2. Get the latitude and the longitude coordinates of each neighborhood**


```python
coor_df = pd.read_csv("./Geospatial_Coordinates.csv")
coor_df.head()
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
      <th>Postal Code</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M1B</td>
      <td>43.806686</td>
      <td>-79.194353</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M1C</td>
      <td>43.784535</td>
      <td>-79.160497</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M1E</td>
      <td>43.763573</td>
      <td>-79.188711</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M1G</td>
      <td>43.770992</td>
      <td>-79.216917</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M1H</td>
      <td>43.773136</td>
      <td>-79.239476</td>
    </tr>
  </tbody>
</table>
</div>



#### Add each neighborhood's geographic coordinate to the DataFrame


```python
# Merge df and coor_df
toronto_df = df.merge(coor_df, left_on='PostalCode', right_on='Postal Code', how='left')

# remove the "Postal Code" column
toronto_df.drop("Postal Code", axis=1, inplace=True)
toronto_df.head()
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
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M1B</td>
      <td>Scarborough</td>
      <td>Malvern, Rouge</td>
      <td>43.806686</td>
      <td>-79.194353</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M1C</td>
      <td>Scarborough</td>
      <td>Rouge Hill, Port Union, Highland Creek</td>
      <td>43.784535</td>
      <td>-79.160497</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M1E</td>
      <td>Scarborough</td>
      <td>Guildwood, Morningside, West Hill</td>
      <td>43.763573</td>
      <td>-79.188711</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M1G</td>
      <td>Scarborough</td>
      <td>Woburn</td>
      <td>43.770992</td>
      <td>-79.216917</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M1H</td>
      <td>Scarborough</td>
      <td>Cedarbrae</td>
      <td>43.773136</td>
      <td>-79.239476</td>
    </tr>
  </tbody>
</table>
</div>



## **3. Explore and cluster the neighborhoods in Toronto**

#### Get the latitude and longitude values of Toronto.


```python
address = 'Toronto, ON'

geolocator = Nominatim(user_agent='toronto_explorer')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geographical coordinate of Toronto are {}, {}.'.format(latitude, longitude))
```

    The geographical coordinate of Toronto are 43.6534817, -79.3839347.
    

#### Create a map of Toronto neighborhoods superimposed on top


```python
# create map of New York using latitude and longitude values
map_toronto = folium.Map(location=[latitude, longitude], zoom_start=10)
map_toronto
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfYzllODExNDM2OTYxNGZlZGEwMDlkYjU5NGIzN2ZlOWQgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2M5ZTgxMTQzNjk2MTRmZWRhMDA5ZGI1OTRiMzdmZTlkIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9jOWU4MTE0MzY5NjE0ZmVkYTAwOWRiNTk0YjM3ZmU5ZCA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9jOWU4MTE0MzY5NjE0ZmVkYTAwOWRiNTk0YjM3ZmU5ZCcsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDMuNjUzNDgxNywtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMCwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMzhiZGFmYzE5NGIyNDUxMDg2NTE5OTU2ODU4YjRkYjcgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYzllODExNDM2OTYxNGZlZGEwMDlkYjU5NGIzN2ZlOWQpOwogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
# Add markers to the map
for lat, lng, borough, neighborhood in zip(toronto_df['Latitude'], toronto_df['Longitude'], toronto_df['Borough'], toronto_df['Neighbourhood']):
    label = '{}, {}'.format(neighborhood, borough)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_toronto)  

map_toronto   
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCcsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDMuNjUzNDgxNywtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMCwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMzIxNTdjNDhjYWM2NDBhM2I0ZjRjZTQ3MmI5ZWQ1NDQgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzY4NjJiY2RmYmVmNzRjNmY4YjM2NzBlY2RjMDZkOTUwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuODA2Njg2Mjk5OTk5OTk2LC03OS4xOTQzNTM0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83Yjc1MGUxZTg5Mzg0NDlhODE1ZGE4NzJjY2ZkNGJlMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83ZDI1NmYwN2RkOTI0ZTg1YTcwYmFjMmIzYWU0YjNjNiA9ICQoJzxkaXYgaWQ9Imh0bWxfN2QyNTZmMDdkZDkyNGU4NWE3MGJhYzJiM2FlNGIzYzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1hbHZlcm4sIFJvdWdlLCBTY2FyYm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfN2I3NTBlMWU4OTM4NDQ5YTgxNWRhODcyY2NmZDRiZTIuc2V0Q29udGVudChodG1sXzdkMjU2ZjA3ZGQ5MjRlODVhNzBiYWMyYjNhZTRiM2M2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzY4NjJiY2RmYmVmNzRjNmY4YjM2NzBlY2RjMDZkOTUwLmJpbmRQb3B1cChwb3B1cF83Yjc1MGUxZTg5Mzg0NDlhODE1ZGE4NzJjY2ZkNGJlMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hZDBjODg0NTM2OGY0ZmY2OWJjZTM0YzZmZDg3NWIxZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc4NDUzNTEsLTc5LjE2MDQ5NzA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzBlZTQzNmMyZDBmNDQ4YmRiMTg1NWUwNmExOTc0NmJiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzNiMjg5MzNlYWE5NTRiM2NhMDkyYmNhM2U4Y2M5YzYyID0gJCgnPGRpdiBpZD0iaHRtbF8zYjI4OTMzZWFhOTU0YjNjYTA5MmJjYTNlOGNjOWM2MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um91Z2UgSGlsbCwgUG9ydCBVbmlvbiwgSGlnaGxhbmQgQ3JlZWssIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wZWU0MzZjMmQwZjQ0OGJkYjE4NTVlMDZhMTk3NDZiYi5zZXRDb250ZW50KGh0bWxfM2IyODkzM2VhYTk1NGIzY2EwOTJiY2EzZThjYzljNjIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYWQwYzg4NDUzNjhmNGZmNjliY2UzNGM2ZmQ4NzViMWQuYmluZFBvcHVwKHBvcHVwXzBlZTQzNmMyZDBmNDQ4YmRiMTg1NWUwNmExOTc0NmJiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2FkMzcxM2ZkMWQ3NzRjNDk4NGJmZGRmZmUxNzY4OGZlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzYzNTcyNiwtNzkuMTg4NzExNV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hOGUyZGVkYWU3NzU0YzE0ODk5YWM1OWEyMWJlOWM1MyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82OGI3OWQwZjM4NjY0MThmOTRmOGFhNDE0NWQ5ODRmNyA9ICQoJzxkaXYgaWQ9Imh0bWxfNjhiNzlkMGYzODY2NDE4Zjk0ZjhhYTQxNDVkOTg0ZjciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkd1aWxkd29vZCwgTW9ybmluZ3NpZGUsIFdlc3QgSGlsbCwgU2NhcmJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2E4ZTJkZWRhZTc3NTRjMTQ4OTlhYzU5YTIxYmU5YzUzLnNldENvbnRlbnQoaHRtbF82OGI3OWQwZjM4NjY0MThmOTRmOGFhNDE0NWQ5ODRmNyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hZDM3MTNmZDFkNzc0YzQ5ODRiZmRkZmZlMTc2ODhmZS5iaW5kUG9wdXAocG9wdXBfYThlMmRlZGFlNzc1NGMxNDg5OWFjNTlhMjFiZTljNTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDMxZTZiNTgzZDlkNDA3NGEyYTE4NzZmNTMyZTgzYjUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43NzA5OTIxLC03OS4yMTY5MTc0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lMWRjNzFjOWQ3MWE0YWFkYTE3MWNhMzZhYmMyN2U2NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iMmM2YzEzMDM3Yzg0ZjBlOTRkYThkMmNmOTQzNDFiMiA9ICQoJzxkaXYgaWQ9Imh0bWxfYjJjNmMxMzAzN2M4NGYwZTk0ZGE4ZDJjZjk0MzQxYjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldvYnVybiwgU2NhcmJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2UxZGM3MWM5ZDcxYTRhYWRhMTcxY2EzNmFiYzI3ZTY3LnNldENvbnRlbnQoaHRtbF9iMmM2YzEzMDM3Yzg0ZjBlOTRkYThkMmNmOTQzNDFiMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80MzFlNmI1ODNkOWQ0MDc0YTJhMTg3NmY1MzJlODNiNS5iaW5kUG9wdXAocG9wdXBfZTFkYzcxYzlkNzFhNGFhZGExNzFjYTM2YWJjMjdlNjcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDA5YmE0NDE5NDM1NDYxNzhhYmY0ZjM4YWI1ZjA3NmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43NzMxMzYsLTc5LjIzOTQ3NjA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Q0MDYyZjRlNDdlMTQyYTJhMjcxNjZlNDIxYmQwNWQ1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdjYzE5YmRkYmZlNjQ4OTM4NGJmMTFiMzFkZmM1YjdkID0gJCgnPGRpdiBpZD0iaHRtbF83Y2MxOWJkZGJmZTY0ODkzODRiZjExYjMxZGZjNWI3ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2VkYXJicmFlLCBTY2FyYm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZDQwNjJmNGU0N2UxNDJhMmEyNzE2NmU0MjFiZDA1ZDUuc2V0Q29udGVudChodG1sXzdjYzE5YmRkYmZlNjQ4OTM4NGJmMTFiMzFkZmM1YjdkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2QwOWJhNDQxOTQzNTQ2MTc4YWJmNGYzOGFiNWYwNzZiLmJpbmRQb3B1cChwb3B1cF9kNDA2MmY0ZTQ3ZTE0MmEyYTI3MTY2ZTQyMWJkMDVkNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mOWMxNzQ0Nzk3ZTk0NGYwYWE5ZTljM2IzMjBjZjUyZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc0NDczNDIsLTc5LjIzOTQ3NjA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U0YjdjNzEwNTdiZjQ2ZjE4YTkyZTdiMTliMzQ3ZjRiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzA5YWIwYTczZTE0MTQ0YjJhMWU2ZGUwMjRkZDU3MmM0ID0gJCgnPGRpdiBpZD0iaHRtbF8wOWFiMGE3M2UxNDE0NGIyYTFlNmRlMDI0ZGQ1NzJjNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2NhcmJvcm91Z2ggVmlsbGFnZSwgU2NhcmJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U0YjdjNzEwNTdiZjQ2ZjE4YTkyZTdiMTliMzQ3ZjRiLnNldENvbnRlbnQoaHRtbF8wOWFiMGE3M2UxNDE0NGIyYTFlNmRlMDI0ZGQ1NzJjNCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mOWMxNzQ0Nzk3ZTk0NGYwYWE5ZTljM2IzMjBjZjUyZS5iaW5kUG9wdXAocG9wdXBfZTRiN2M3MTA1N2JmNDZmMThhOTJlN2IxOWIzNDdmNGIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTlhZjQ0YTg5ZTk2NGRiMTg5MGRkYTE0ODMwMjIzMWQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43Mjc5MjkyLC03OS4yNjIwMjk0MDAwMDAwMl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZDU2ZjcxODkzNDM0ODAxOTgzMWQ0ZDg1N2QzOThlYSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wMzliNGQzNDMxNDY0MTU2ODE3ZTVmYzk1Y2Q4Nzg2MiA9ICQoJzxkaXYgaWQ9Imh0bWxfMDM5YjRkMzQzMTQ2NDE1NjgxN2U1ZmM5NWNkODc4NjIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktlbm5lZHkgUGFyaywgSW9udmlldywgRWFzdCBCaXJjaG1vdW50IFBhcmssIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xZDU2ZjcxODkzNDM0ODAxOTgzMWQ0ZDg1N2QzOThlYS5zZXRDb250ZW50KGh0bWxfMDM5YjRkMzQzMTQ2NDE1NjgxN2U1ZmM5NWNkODc4NjIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTlhZjQ0YTg5ZTk2NGRiMTg5MGRkYTE0ODMwMjIzMWQuYmluZFBvcHVwKHBvcHVwXzFkNTZmNzE4OTM0MzQ4MDE5ODMxZDRkODU3ZDM5OGVhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEwNjg1ODU3ZTdjOTQ1Mzc4ZTM5NzdiYTg3YWZmNzU4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzExMTExNzAwMDAwMDA0LC03OS4yODQ1NzcyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY3MWIwMzJlZmZlMzRlMWJiYTY1MTJkNmIxZWJjNzljID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2JmNmFkZDIwNmQ4NjQ3NTQ5Njk5MzJmYjUxMDEzNWNlID0gJCgnPGRpdiBpZD0iaHRtbF9iZjZhZGQyMDZkODY0NzU0OTY5OTMyZmI1MTAxMzVjZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R29sZGVuIE1pbGUsIENsYWlybGVhLCBPYWtyaWRnZSwgU2NhcmJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY3MWIwMzJlZmZlMzRlMWJiYTY1MTJkNmIxZWJjNzljLnNldENvbnRlbnQoaHRtbF9iZjZhZGQyMDZkODY0NzU0OTY5OTMyZmI1MTAxMzVjZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xMDY4NTg1N2U3Yzk0NTM3OGUzOTc3YmE4N2FmZjc1OC5iaW5kUG9wdXAocG9wdXBfNjcxYjAzMmVmZmUzNGUxYmJhNjUxMmQ2YjFlYmM3OWMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYTk3YzBjMzc5ZDVhNDk2NGIxOTJjZThmMjNjNDVlZWMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTYzMTYsLTc5LjIzOTQ3NjA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA0YzAyNGZhNmY5ZTQ1NmU4NGQzYTVlY2ZmOTc2ZTIxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE3ODg1NzQ4MDgxNDQxMDA5ODZkZmQ2ZTAxYzAyYTVkID0gJCgnPGRpdiBpZD0iaHRtbF8xNzg4NTc0ODA4MTQ0MTAwOTg2ZGZkNmUwMWMwMmE1ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2xpZmZzaWRlLCBDbGlmZmNyZXN0LCBTY2FyYm9yb3VnaCBWaWxsYWdlIFdlc3QsIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wNGMwMjRmYTZmOWU0NTZlODRkM2E1ZWNmZjk3NmUyMS5zZXRDb250ZW50KGh0bWxfMTc4ODU3NDgwODE0NDEwMDk4NmRmZDZlMDFjMDJhNWQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYTk3YzBjMzc5ZDVhNDk2NGIxOTJjZThmMjNjNDVlZWMuYmluZFBvcHVwKHBvcHVwXzA0YzAyNGZhNmY5ZTQ1NmU4NGQzYTVlY2ZmOTc2ZTIxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzY1OTRkY2M4ZTRjNDRhNjM5ZGI4YTU3MmJkMzE1Yjc4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjkyNjU3MDAwMDAwMDA0LC03OS4yNjQ4NDgxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2UzMWQ5ZjBjOTRhYzQwNWE4NzUxMjYwNjRjOTE1ZDZkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdkNjI0M2ZlYjQwMjRiM2RiYjIzNjRjNmUzYmVhZWRhID0gJCgnPGRpdiBpZD0iaHRtbF83ZDYyNDNmZWI0MDI0YjNkYmIyMzY0YzZlM2JlYWVkYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmlyY2ggQ2xpZmYsIENsaWZmc2lkZSBXZXN0LCBTY2FyYm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZTMxZDlmMGM5NGFjNDA1YTg3NTEyNjA2NGM5MTVkNmQuc2V0Q29udGVudChodG1sXzdkNjI0M2ZlYjQwMjRiM2RiYjIzNjRjNmUzYmVhZWRhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzY1OTRkY2M4ZTRjNDRhNjM5ZGI4YTU3MmJkMzE1Yjc4LmJpbmRQb3B1cChwb3B1cF9lMzFkOWYwYzk0YWM0MDVhODc1MTI2MDY0YzkxNWQ2ZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xNjgxMmRlNWJhNGQ0ZWJlOTYyM2QxNjAxNDAxNDM2ZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc1NzQwOTYsLTc5LjI3MzMwNDAwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg3YzNiOTY0Y2I5MzRiZmM5MWFjODk2MTI4MjcwZDc3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzFlYWEzYWI2OGQ1YjRmZjhhZjcyNTZiZTI2OTk1OTdhID0gJCgnPGRpdiBpZD0iaHRtbF8xZWFhM2FiNjhkNWI0ZmY4YWY3MjU2YmUyNjk5NTk3YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9yc2V0IFBhcmssIFdleGZvcmQgSGVpZ2h0cywgU2NhcmJvcm91Z2ggVG93biBDZW50cmUsIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84N2MzYjk2NGNiOTM0YmZjOTFhYzg5NjEyODI3MGQ3Ny5zZXRDb250ZW50KGh0bWxfMWVhYTNhYjY4ZDViNGZmOGFmNzI1NmJlMjY5OTU5N2EpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTY4MTJkZTViYTRkNGViZTk2MjNkMTYwMTQwMTQzNmQuYmluZFBvcHVwKHBvcHVwXzg3YzNiOTY0Y2I5MzRiZmM5MWFjODk2MTI4MjcwZDc3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk5OWFmOTE4N2MyMzQ5NGE5ZmRjNDdkMDA4YzJiNGYxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzUwMDcxNTAwMDAwMDA0LC03OS4yOTU4NDkxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNlZDFiMzcxMDQ1NjQ0MmU5ZDk4ZTA0MTBjYjEyNmYxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzFlYmNkOTA3ZjEyMjQxZDU4ODI3ZTMzZDU5Mzg5MjBkID0gJCgnPGRpdiBpZD0iaHRtbF8xZWJjZDkwN2YxMjI0MWQ1ODgyN2UzM2Q1OTM4OTIwZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+V2V4Zm9yZCwgTWFyeXZhbGUsIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zZWQxYjM3MTA0NTY0NDJlOWQ5OGUwNDEwY2IxMjZmMS5zZXRDb250ZW50KGh0bWxfMWViY2Q5MDdmMTIyNDFkNTg4MjdlMzNkNTkzODkyMGQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTk5YWY5MTg3YzIzNDk0YTlmZGM0N2QwMDhjMmI0ZjEuYmluZFBvcHVwKHBvcHVwXzNlZDFiMzcxMDQ1NjQ0MmU5ZDk4ZTA0MTBjYjEyNmYxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzkxODI3ZWZlYjc1YTQ4NmZiYzkxMWNkNDYzNzZjNzY2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzk0MjAwMywtNzkuMjYyMDI5NDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMGIwZGFiNzU3MGIyNGEzN2FhNDNlNTcxMjRiOWE4ZWQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWQ3MTc3NzIwN2ZkNDYxNGE4YTA0MmNjYjk0YzIwNDEgPSAkKCc8ZGl2IGlkPSJodG1sX2VkNzE3NzcyMDdmZDQ2MTRhOGEwNDJjY2I5NGMyMDQxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BZ2luY291cnQsIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wYjBkYWI3NTcwYjI0YTM3YWE0M2U1NzEyNGI5YThlZC5zZXRDb250ZW50KGh0bWxfZWQ3MTc3NzIwN2ZkNDYxNGE4YTA0MmNjYjk0YzIwNDEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTE4MjdlZmViNzVhNDg2ZmJjOTExY2Q0NjM3NmM3NjYuYmluZFBvcHVwKHBvcHVwXzBiMGRhYjc1NzBiMjRhMzdhYTQzZTU3MTI0YjlhOGVkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzhkMDIxN2RiZDE4ODQ5MDViZmQ5ZWQzYzhjMWU2ZjkyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzgxNjM3NSwtNzkuMzA0MzAyMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83Y2U5YTA1NTFhNTI0NDczYTRiNWE4MmY2ZTAyNTQ0ZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNDQwM2NkYTRiYmU0NjQ4YjUxYmU1OGY5ODUxMjdkOSA9ICQoJzxkaXYgaWQ9Imh0bWxfMzQ0MDNjZGE0YmJlNDY0OGI1MWJlNThmOTg1MTI3ZDkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNsYXJrcyBDb3JuZXJzLCBUYW0gTyYjMzk7U2hhbnRlciwgU3VsbGl2YW4sIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83Y2U5YTA1NTFhNTI0NDczYTRiNWE4MmY2ZTAyNTQ0Zi5zZXRDb250ZW50KGh0bWxfMzQ0MDNjZGE0YmJlNDY0OGI1MWJlNThmOTg1MTI3ZDkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOGQwMjE3ZGJkMTg4NDkwNWJmZDllZDNjOGMxZTZmOTIuYmluZFBvcHVwKHBvcHVwXzdjZTlhMDU1MWE1MjQ0NzNhNGI1YTgyZjZlMDI1NDRmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2FhMmZjNzA5M2ZlNjQ0ZTlhZDlmN2FjZjcwNzhlZTk1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuODE1MjUyMiwtNzkuMjg0NTc3Ml0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82YzI4NWQxMGZiZDY0ZmRhYjg1ZTE1ZDNmN2NmNTI3ZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82NzdkNzA0ZmVjNTQ0NzVjYTIxMDc1MDNmYjJjNmM1YiA9ICQoJzxkaXYgaWQ9Imh0bWxfNjc3ZDcwNGZlYzU0NDc1Y2EyMTA3NTAzZmIyYzZjNWIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1pbGxpa2VuLCBBZ2luY291cnQgTm9ydGgsIFN0ZWVsZXMgRWFzdCwgTCYjMzk7QW1vcmVhdXggRWFzdCwgU2NhcmJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzZjMjg1ZDEwZmJkNjRmZGFiODVlMTVkM2Y3Y2Y1MjdlLnNldENvbnRlbnQoaHRtbF82NzdkNzA0ZmVjNTQ0NzVjYTIxMDc1MDNmYjJjNmM1Yik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hYTJmYzcwOTNmZTY0NGU5YWQ5ZjdhY2Y3MDc4ZWU5NS5iaW5kUG9wdXAocG9wdXBfNmMyODVkMTBmYmQ2NGZkYWI4NWUxNWQzZjdjZjUyN2UpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDdmN2FiMjMzZGNhNDQxNDkwM2FlOWYwMWM5YmZlZTkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43OTk1MjUyMDAwMDAwMDUsLTc5LjMxODM4ODddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2NhYTViMjZlY2QwNGJkYTlkYWY3ODM1NGJlNDZiOTMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2NhN2UyMTcwMDliNGE0YmE3ZGVhMGJjMTU2NDU3OTUgPSAkKCc8ZGl2IGlkPSJodG1sXzNjYTdlMjE3MDA5YjRhNGJhN2RlYTBiYzE1NjQ1Nzk1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdGVlbGVzIFdlc3QsIEwmIzM5O0Ftb3JlYXV4IFdlc3QsIFNjYXJib3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zY2FhNWIyNmVjZDA0YmRhOWRhZjc4MzU0YmU0NmI5My5zZXRDb250ZW50KGh0bWxfM2NhN2UyMTcwMDliNGE0YmE3ZGVhMGJjMTU2NDU3OTUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMDdmN2FiMjMzZGNhNDQxNDkwM2FlOWYwMWM5YmZlZTkuYmluZFBvcHVwKHBvcHVwXzNjYWE1YjI2ZWNkMDRiZGE5ZGFmNzgzNTRiZTQ2YjkzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM1ZDBkYWYzMjFlYjRmZWJhYTk0YzRkN2NiYjJkZWQ5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuODM2MTI0NzAwMDAwMDA2LC03OS4yMDU2MzYwOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80MTVlMTg2ZGVmNDc0YzM0ODI3NmFhNDJiZmMwYWQ5OSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zZTJjOGM0MTc4YTY0ZjdiODk3NjYwZGY3NzY4N2MxNiA9ICQoJzxkaXYgaWQ9Imh0bWxfM2UyYzhjNDE3OGE2NGY3Yjg5NzY2MGRmNzc2ODdjMTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlVwcGVyIFJvdWdlLCBTY2FyYm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDE1ZTE4NmRlZjQ3NGMzNDgyNzZhYTQyYmZjMGFkOTkuc2V0Q29udGVudChodG1sXzNlMmM4YzQxNzhhNjRmN2I4OTc2NjBkZjc3Njg3YzE2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzM1ZDBkYWYzMjFlYjRmZWJhYTk0YzRkN2NiYjJkZWQ5LmJpbmRQb3B1cChwb3B1cF80MTVlMTg2ZGVmNDc0YzM0ODI3NmFhNDJiZmMwYWQ5OSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84MDlhN2I3ZmY0MmM0ZmEyODkxOTg3YTEzMjRhZTAwOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjgwMzc2MjIsLTc5LjM2MzQ1MTddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTRjNzdmZDQ5OTZiNDk1N2I1NzQ5Y2I4YWQ2Nzg3YjQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjEzYmY0ZDJmM2MzNGVjMDg3YjYxYWIwM2JiZTM5N2QgPSAkKCc8ZGl2IGlkPSJodG1sX2YxM2JmNGQyZjNjMzRlYzA4N2I2MWFiMDNiYmUzOTdkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IaWxsY3Jlc3QgVmlsbGFnZSwgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTRjNzdmZDQ5OTZiNDk1N2I1NzQ5Y2I4YWQ2Nzg3YjQuc2V0Q29udGVudChodG1sX2YxM2JmNGQyZjNjMzRlYzA4N2I2MWFiMDNiYmUzOTdkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzgwOWE3YjdmZjQyYzRmYTI4OTE5ODdhMTMyNGFlMDA5LmJpbmRQb3B1cChwb3B1cF81NGM3N2ZkNDk5NmI0OTU3YjU3NDljYjhhZDY3ODdiNCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jOWJiY2E5ZDY0NTg0ZTJhYmUwMzU4ZmI2ZmI1MTQ4YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc3ODUxNzUsLTc5LjM0NjU1NTddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2U2ODc5OTA2ZDc3NDFhY2IxMWNmMjZmZjRjNzNjMmUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzZhY2JkNzcxNzk4NDA0Yjg1ZjRmZDU5MGJkNTQ2YzcgPSAkKCc8ZGl2IGlkPSJodG1sXzM2YWNiZDc3MTc5ODQwNGI4NWY0ZmQ1OTBiZDU0NmM3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GYWlydmlldywgSGVucnkgRmFybSwgT3Jpb2xlLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zZTY4Nzk5MDZkNzc0MWFjYjExY2YyNmZmNGM3M2MyZS5zZXRDb250ZW50KGh0bWxfMzZhY2JkNzcxNzk4NDA0Yjg1ZjRmZDU5MGJkNTQ2YzcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYzliYmNhOWQ2NDU4NGUyYWJlMDM1OGZiNmZiNTE0OGMuYmluZFBvcHVwKHBvcHVwXzNlNjg3OTkwNmQ3NzQxYWNiMTFjZjI2ZmY0YzczYzJlKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZmYTdjMjY5OGEzNjQyN2VhZjRhNjY0Y2MzMDMxYTcwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzg2OTQ3MywtNzkuMzg1OTc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI0ZTM3NmFkN2E0NDQ4ODJhZjkxYjUxYTYzNmVhN2U5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzBhM2Q5OGU3NmU0ZDQ2YjA4MmUzYjcyY2YxNzc0MTkwID0gJCgnPGRpdiBpZD0iaHRtbF8wYTNkOThlNzZlNGQ0NmIwODJlM2I3MmNmMTc3NDE5MCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmF5dmlldyBWaWxsYWdlLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yNGUzNzZhZDdhNDQ0ODgyYWY5MWI1MWE2MzZlYTdlOS5zZXRDb250ZW50KGh0bWxfMGEzZDk4ZTc2ZTRkNDZiMDgyZTNiNzJjZjE3NzQxOTApOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZmZhN2MyNjk4YTM2NDI3ZWFmNGE2NjRjYzMwMzFhNzAuYmluZFBvcHVwKHBvcHVwXzI0ZTM3NmFkN2E0NDQ4ODJhZjkxYjUxYTYzNmVhN2U5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzBmZGU1NzlkYjEwYjQxOTY5OGI0OGZkYWZlNzA0NGM1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzU3NDkwMiwtNzkuMzc0NzE0MDk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMjJmYmM1ZWMzNjI3NGM3NjkyMTllMmIyNDJlYzI1YjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2U2YzZhMTQyY2JlNDk0OTlkM2M5NjkwMjA2MGJlODIgPSAkKCc8ZGl2IGlkPSJodG1sXzdlNmM2YTE0MmNiZTQ5NDk5ZDNjOTY5MDIwNjBiZTgyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Zb3JrIE1pbGxzLCBTaWx2ZXIgSGlsbHMsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzIyZmJjNWVjMzYyNzRjNzY5MjE5ZTJiMjQyZWMyNWIwLnNldENvbnRlbnQoaHRtbF83ZTZjNmExNDJjYmU0OTQ5OWQzYzk2OTAyMDYwYmU4Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wZmRlNTc5ZGIxMGI0MTk2OThiNDhmZGFmZTcwNDRjNS5iaW5kUG9wdXAocG9wdXBfMjJmYmM1ZWMzNjI3NGM3NjkyMTllMmIyNDJlYzI1YjApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjM2MjIyYTE3ODMzNDAwZDhmZjkwMzM4Y2JmZDYxOGUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43ODkwNTMsLTc5LjQwODQ5Mjc5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzcyZDc0OTNkNDM0ZDRkOTVhNWQwNTNhYTM2N2I3ZjQ2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzAwMmExOGYwY2I0ZTQ4ODhhMTE5YWYyNDgwMzg4MjcyID0gJCgnPGRpdiBpZD0iaHRtbF8wMDJhMThmMGNiNGU0ODg4YTExOWFmMjQ4MDM4ODI3MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+V2lsbG93ZGFsZSwgTmV3dG9uYnJvb2ssIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzcyZDc0OTNkNDM0ZDRkOTVhNWQwNTNhYTM2N2I3ZjQ2LnNldENvbnRlbnQoaHRtbF8wMDJhMThmMGNiNGU0ODg4YTExOWFmMjQ4MDM4ODI3Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82MzYyMjJhMTc4MzM0MDBkOGZmOTAzMzhjYmZkNjE4ZS5iaW5kUG9wdXAocG9wdXBfNzJkNzQ5M2Q0MzRkNGQ5NWE1ZDA1M2FhMzY3YjdmNDYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMGYwYzY0OTgyYzBkNDI1NTg1NWJjNzc0NmRlMWE4ZmQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43NzAxMTk5LC03OS40MDg0OTI3OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mYzllYzAyMDAwZGI0ZWFmYWViZDc4ZGY5MGRhNzAxYSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hYjlkM2NmMDgyODA0YjU1YmEyMmZjNDRmNmQxNWZhMiA9ICQoJzxkaXYgaWQ9Imh0bWxfYWI5ZDNjZjA4MjgwNGI1NWJhMjJmYzQ0ZjZkMTVmYTIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldpbGxvd2RhbGUsIFdpbGxvd2RhbGUgRWFzdCwgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZmM5ZWMwMjAwMGRiNGVhZmFlYmQ3OGRmOTBkYTcwMWEuc2V0Q29udGVudChodG1sX2FiOWQzY2YwODI4MDRiNTViYTIyZmM0NGY2ZDE1ZmEyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBmMGM2NDk4MmMwZDQyNTU4NTViYzc3NDZkZTFhOGZkLmJpbmRQb3B1cChwb3B1cF9mYzllYzAyMDAwZGI0ZWFmYWViZDc4ZGY5MGRhNzAxYSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wZDQ3NjNjN2FkMjY0OWFlYTc0YzUyMDdiM2Y3MTMwZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc1Mjc1ODI5OTk5OTk5NiwtNzkuNDAwMDQ5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jZmE0ZjA4ZWJhNmE0NGFlYWJlM2NkMDhlY2NmZWJlNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84YTU5ZmQ5MGQ2ZTQ0ZjY2YjhlZjEyNTg3NWFlMGY0NyA9ICQoJzxkaXYgaWQ9Imh0bWxfOGE1OWZkOTBkNmU0NGY2NmI4ZWYxMjU4NzVhZTBmNDciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPllvcmsgTWlsbHMgV2VzdCwgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfY2ZhNGYwOGViYTZhNDRhZWFiZTNjZDA4ZWNjZmViZTcuc2V0Q29udGVudChodG1sXzhhNTlmZDkwZDZlNDRmNjZiOGVmMTI1ODc1YWUwZjQ3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBkNDc2M2M3YWQyNjQ5YWVhNzRjNTIwN2IzZjcxMzBkLmJpbmRQb3B1cChwb3B1cF9jZmE0ZjA4ZWJhNmE0NGFlYWJlM2NkMDhlY2NmZWJlNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83ZDkyMDkyNGVlOTI0ZTMxOTIzNjU0ZTBhZGQxZjRjNiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc4MjczNjQsLTc5LjQ0MjI1OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDBjY2EyY2VlZjhjNDgxOGFmZjYzYzg1MGFkZmRmZTIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYTdlMjgwMGI0MGQyNDU5ZWFiMWI0NjE5YzllYTE3ZDcgPSAkKCc8ZGl2IGlkPSJodG1sX2E3ZTI4MDBiNDBkMjQ1OWVhYjFiNDYxOWM5ZWExN2Q3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5XaWxsb3dkYWxlLCBXaWxsb3dkYWxlIFdlc3QsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2QwY2NhMmNlZWY4YzQ4MThhZmY2M2M4NTBhZGZkZmUyLnNldENvbnRlbnQoaHRtbF9hN2UyODAwYjQwZDI0NTllYWIxYjQ2MTljOWVhMTdkNyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83ZDkyMDkyNGVlOTI0ZTMxOTIzNjU0ZTBhZGQxZjRjNi5iaW5kUG9wdXAocG9wdXBfZDBjY2EyY2VlZjhjNDgxOGFmZjYzYzg1MGFkZmRmZTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTg5NGM0MzI4NWFkNDkzYjk4ZTk3N2JiMDIxYjE1MTMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43NTMyNTg2LC03OS4zMjk2NTY1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzYyNTI3Y2U2Nzc4YzRkY2JiZmM0ODdlZmRkNmFlMmVhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VjMGQ0MGEwZTgyMjQ4OWRhODExOTAxNjg3MDg4ODA2ID0gJCgnPGRpdiBpZD0iaHRtbF9lYzBkNDBhMGU4MjI0ODlkYTgxMTkwMTY4NzA4ODgwNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFya3dvb2RzLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82MjUyN2NlNjc3OGM0ZGNiYmZjNDg3ZWZkZDZhZTJlYS5zZXRDb250ZW50KGh0bWxfZWMwZDQwYTBlODIyNDg5ZGE4MTE5MDE2ODcwODg4MDYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZTg5NGM0MzI4NWFkNDkzYjk4ZTk3N2JiMDIxYjE1MTMuYmluZFBvcHVwKHBvcHVwXzYyNTI3Y2U2Nzc4YzRkY2JiZmM0ODdlZmRkNmFlMmVhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzUyOGM1ZjE2NWI1YzQ5ODZiNDBhNzk5ODVkYzhmMjYxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzQ1OTA1Nzk5OTk5OTk2LC03OS4zNTIxODhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZWQ1MmI1OWFlODJkNDNhNzg2ZWQ0N2ZjZjIyNTU0YzYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWU1YTNhZjAwZGJmNGNhZDgzMWU5ZjM4MTUwYWMwMDEgPSAkKCc8ZGl2IGlkPSJodG1sX2VlNWEzYWYwMGRiZjRjYWQ4MzFlOWYzODE1MGFjMDAxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb24gTWlsbHMsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2VkNTJiNTlhZTgyZDQzYTc4NmVkNDdmY2YyMjU1NGM2LnNldENvbnRlbnQoaHRtbF9lZTVhM2FmMDBkYmY0Y2FkODMxZTlmMzgxNTBhYzAwMSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81MjhjNWYxNjViNWM0OTg2YjQwYTc5OTg1ZGM4ZjI2MS5iaW5kUG9wdXAocG9wdXBfZWQ1MmI1OWFlODJkNDNhNzg2ZWQ0N2ZjZjIyNTU0YzYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTQ4MjQ2M2VmZmY3NDRjYWEyYTAwNmZjNjM0OWU4MGEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MjU4OTk3MDAwMDAwMSwtNzkuMzQwOTIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2UwYzgyOWM0NDg4ODRiZjE5Y2E2ZTZlNTcxMTdkZmQ4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2NkMDM3M2M0NWU3NDRmOGQ4ZWE4YmVjNDY5MmY4ZTQzID0gJCgnPGRpdiBpZD0iaHRtbF9jZDAzNzNjNDVlNzQ0ZjhkOGVhOGJlYzQ2OTJmOGU0MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9uIE1pbGxzLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lMGM4MjljNDQ4ODg0YmYxOWNhNmU2ZTU3MTE3ZGZkOC5zZXRDb250ZW50KGh0bWxfY2QwMzczYzQ1ZTc0NGY4ZDhlYThiZWM0NjkyZjhlNDMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTQ4MjQ2M2VmZmY3NDRjYWEyYTAwNmZjNjM0OWU4MGEuYmluZFBvcHVwKHBvcHVwX2UwYzgyOWM0NDg4ODRiZjE5Y2E2ZTZlNTcxMTdkZmQ4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzYxZTYyYjBjMTE0MTQ1YmE4YzJlMzczNzRkZTE4NzEwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzU0MzI4MywtNzkuNDQyMjU5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xYjhjZmI1MDFjZWY0MTVkYjE1NTJiMmUzYTdkOTQ0NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lZjA3ZWViZDE2MDY0N2Q2YmZmODI1ZDkwZWQ0YmVlNiA9ICQoJzxkaXYgaWQ9Imh0bWxfZWYwN2VlYmQxNjA2NDdkNmJmZjgyNWQ5MGVkNGJlZTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJhdGh1cnN0IE1hbm9yLCBXaWxzb24gSGVpZ2h0cywgRG93bnN2aWV3IE5vcnRoLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xYjhjZmI1MDFjZWY0MTVkYjE1NTJiMmUzYTdkOTQ0Ny5zZXRDb250ZW50KGh0bWxfZWYwN2VlYmQxNjA2NDdkNmJmZjgyNWQ5MGVkNGJlZTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNjFlNjJiMGMxMTQxNDViYThjMmUzNzM3NGRlMTg3MTAuYmluZFBvcHVwKHBvcHVwXzFiOGNmYjUwMWNlZjQxNWRiMTU1MmIyZTNhN2Q5NDQ3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI4ODRmZGQxYTdiMjQzYjViZTkzN2E0MjUwOWY3MDU3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzY3OTgwMywtNzkuNDg3MjYxOTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWYwOWM0YjdhMWEwNDU3YzllZGU1MTBjODA4YTcyNjMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNDkyZjJmM2MwM2YxNGRmYmE0MzIzMzJmNmI5OTlhNGUgPSAkKCc8ZGl2IGlkPSJodG1sXzQ5MmYyZjNjMDNmMTRkZmJhNDMyMzMyZjZiOTk5YTRlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ob3J0aHdvb2QgUGFyaywgWW9yayBVbml2ZXJzaXR5LCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85ZjA5YzRiN2ExYTA0NTdjOWVkZTUxMGM4MDhhNzI2My5zZXRDb250ZW50KGh0bWxfNDkyZjJmM2MwM2YxNGRmYmE0MzIzMzJmNmI5OTlhNGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMjg4NGZkZDFhN2IyNDNiNWJlOTM3YTQyNTA5ZjcwNTcuYmluZFBvcHVwKHBvcHVwXzlmMDljNGI3YTFhMDQ1N2M5ZWRlNTEwYzgwOGE3MjYzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzY4MTQxZWExYjM1ZjQxYWViOTQ3NjA2MzdmYTlhMjZmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzM3NDczMjAwMDAwMDA0LC03OS40NjQ3NjMyOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMGI2OTE1ZjdjNDk0NWJkYTJmMDg2M2MzMjA2YTY4NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jYjA1NjU2NzFhZTg0Njc0OGYwZDdhMjBjY2I0YzMwMyA9ICQoJzxkaXYgaWQ9Imh0bWxfY2IwNTY1NjcxYWU4NDY3NDhmMGQ3YTIwY2NiNGMzMDMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRvd25zdmlldywgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjBiNjkxNWY3YzQ5NDViZGEyZjA4NjNjMzIwNmE2ODQuc2V0Q29udGVudChodG1sX2NiMDU2NTY3MWFlODQ2NzQ4ZjBkN2EyMGNjYjRjMzAzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzY4MTQxZWExYjM1ZjQxYWViOTQ3NjA2MzdmYTlhMjZmLmJpbmRQb3B1cChwb3B1cF9iMGI2OTE1ZjdjNDk0NWJkYTJmMDg2M2MzMjA2YTY4NCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kZjc1OWVmOGZlZTA0MzIwYmVlNmJhZTVhN2FlMWZmNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjczOTAxNDYsLTc5LjUwNjk0MzZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYWVkMDBhMmMyOGQ0NGUyZGExZGQ1ZTI5YTRhMTZjNjIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjM5YjBkYWQyMTQ1NDE4NWEzMzEzYzIxYjhkZWMyMmUgPSAkKCc8ZGl2IGlkPSJodG1sX2YzOWIwZGFkMjE0NTQxODVhMzMxM2MyMWI4ZGVjMjJlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb3duc3ZpZXcsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2FlZDAwYTJjMjhkNDRlMmRhMWRkNWUyOWE0YTE2YzYyLnNldENvbnRlbnQoaHRtbF9mMzliMGRhZDIxNDU0MTg1YTMzMTNjMjFiOGRlYzIyZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kZjc1OWVmOGZlZTA0MzIwYmVlNmJhZTVhN2FlMWZmNS5iaW5kUG9wdXAocG9wdXBfYWVkMDBhMmMyOGQ0NGUyZGExZGQ1ZTI5YTRhMTZjNjIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfODlhNjZkN2RlODVjNGY5Njg1ODVhYWVhYjRlNzJkM2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43Mjg0OTY0LC03OS40OTU2OTc0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xYWYyOTk1NTQxYWM0ODQ0YjQxMjIyNjljOTgxOWI3MyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jYWVhODQ3NGI1NDQ0NjU1YjE2OTI4NmNhYTM3MTNmNiA9ICQoJzxkaXYgaWQ9Imh0bWxfY2FlYTg0NzRiNTQ0NDY1NWIxNjkyODZjYWEzNzEzZjYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRvd25zdmlldywgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWFmMjk5NTU0MWFjNDg0NGI0MTIyMjY5Yzk4MTliNzMuc2V0Q29udGVudChodG1sX2NhZWE4NDc0YjU0NDQ2NTViMTY5Mjg2Y2FhMzcxM2Y2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzg5YTY2ZDdkZTg1YzRmOTY4NTg1YWFlYWI0ZTcyZDNlLmJpbmRQb3B1cChwb3B1cF8xYWYyOTk1NTQxYWM0ODQ0YjQxMjIyNjljOTgxOWI3Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lOTY5ZDQ5YTQ1MmI0NmJlODUzMTUyMDFhM2YwYjI1MyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjc2MTYzMTMsLTc5LjUyMDk5OTQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2ZhYTBlYjM4M2VlZTQxNDdiNGQ0NmFiMmViMmU5YTlhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQwZmIyZjU4NjNmYzQ4MDE5ZjU5NmRkOTE0ZGQwOTExID0gJCgnPGRpdiBpZD0iaHRtbF80MGZiMmY1ODYzZmM0ODAxOWY1OTZkZDkxNGRkMDkxMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG93bnN2aWV3LCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mYWEwZWIzODNlZWU0MTQ3YjRkNDZhYjJlYjJlOWE5YS5zZXRDb250ZW50KGh0bWxfNDBmYjJmNTg2M2ZjNDgwMTlmNTk2ZGQ5MTRkZDA5MTEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZTk2OWQ0OWE0NTJiNDZiZTg1MzE1MjAxYTNmMGIyNTMuYmluZFBvcHVwKHBvcHVwX2ZhYTBlYjM4M2VlZTQxNDdiNGQ0NmFiMmViMmU5YTlhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM1ZjllYmE3ZGE0MzRhOTdiMTIyNWUxODJkNmYwNmNhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzI1ODgyMjk5OTk5OTk1LC03OS4zMTU1NzE1OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80Mjc2NzQ2OGM5NjM0ZTdhOWQ1OTYxOGZkNTU4NmM5YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85MDNhMzdkZGFiN2M0ODY1OTExMDcyZWFiODViZDE4ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfOTAzYTM3ZGRhYjdjNDg2NTkxMTA3MmVhYjg1YmQxOGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlZpY3RvcmlhIFZpbGxhZ2UsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzQyNzY3NDY4Yzk2MzRlN2E5ZDU5NjE4ZmQ1NTg2YzliLnNldENvbnRlbnQoaHRtbF85MDNhMzdkZGFiN2M0ODY1OTExMDcyZWFiODViZDE4ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zNWY5ZWJhN2RhNDM0YTk3YjEyMjVlMTgyZDZmMDZjYS5iaW5kUG9wdXAocG9wdXBfNDI3Njc0NjhjOTYzNGU3YTlkNTk2MThmZDU1ODZjOWIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZmM3OTAwYjQyY2QzNDEwNzlhZDYyNTg1NzUwZWRhZjggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MDYzOTcyLC03OS4zMDk5MzddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDM0NmZhMDZjYTgxNDY0MGI2ZmIyZjRkZDM5NjgwODkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjE3N2VlMDc1MjZhNGNmZGI1MzJjOGQ1NmZhYWE1MmIgPSAkKCc8ZGl2IGlkPSJodG1sX2IxNzdlZTA3NTI2YTRjZmRiNTMyYzhkNTZmYWFhNTJiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QYXJrdmlldyBIaWxsLCBXb29kYmluZSBHYXJkZW5zLCBFYXN0IFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAzNDZmYTA2Y2E4MTQ2NDBiNmZiMmY0ZGQzOTY4MDg5LnNldENvbnRlbnQoaHRtbF9iMTc3ZWUwNzUyNmE0Y2ZkYjUzMmM4ZDU2ZmFhYTUyYik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mYzc5MDBiNDJjZDM0MTA3OWFkNjI1ODU3NTBlZGFmOC5iaW5kUG9wdXAocG9wdXBfMDM0NmZhMDZjYTgxNDY0MGI2ZmIyZjRkZDM5NjgwODkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTBhOGFmMWUyZmY1NDFmOWExYzcyYzYwMDVkMmRmMTAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42OTUzNDM5MDAwMDAwMDUsLTc5LjMxODM4ODddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDA4MGFjOWJlOGVhNDViMTg5YmU2Y2FjNDAzYTQzN2MgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjlhODM4MDU0NjI2NDUxNTg1NmQ2OTg2MmRjYzI2ZDIgPSAkKCc8ZGl2IGlkPSJodG1sX2Y5YTgzODA1NDYyNjQ1MTU4NTZkNjk4NjJkY2MyNmQyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Xb29kYmluZSBIZWlnaHRzLCBFYXN0IFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAwODBhYzliZThlYTQ1YjE4OWJlNmNhYzQwM2E0MzdjLnNldENvbnRlbnQoaHRtbF9mOWE4MzgwNTQ2MjY0NTE1ODU2ZDY5ODYyZGNjMjZkMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85MGE4YWYxZTJmZjU0MWY5YTFjNzJjNjAwNWQyZGYxMC5iaW5kUG9wdXAocG9wdXBfMDA4MGFjOWJlOGVhNDViMTg5YmU2Y2FjNDAzYTQzN2MpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjg1OWJkNWZhNDY5NDY1Yzk0ZjliMWUxNzczYzA3ZDQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NzYzNTczOTk5OTk5OSwtNzkuMjkzMDMxMl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zMThmY2FmNTI1NzU0ZDUzOTZjNWZmMjJlOWNmMWFiYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hZWQzOTAwZDQxNDg0MDg3YTkxNjc0ZjQwNThmMmY1YyA9ICQoJzxkaXYgaWQ9Imh0bWxfYWVkMzkwMGQ0MTQ4NDA4N2E5MTY3NGY0MDU4ZjJmNWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBCZWFjaGVzLCBFYXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMxOGZjYWY1MjU3NTRkNTM5NmM1ZmYyMmU5Y2YxYWJjLnNldENvbnRlbnQoaHRtbF9hZWQzOTAwZDQxNDg0MDg3YTkxNjc0ZjQwNThmMmY1Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iODU5YmQ1ZmE0Njk0NjVjOTRmOWIxZTE3NzNjMDdkNC5iaW5kUG9wdXAocG9wdXBfMzE4ZmNhZjUyNTc1NGQ1Mzk2YzVmZjIyZTljZjFhYmMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOWNiMzQ3Y2FiODg5NDliZGI1OTU3ZDRjYzk5MTAzNzggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MDkwNjA0LC03OS4zNjM0NTE3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzRhMGU0ZDQ0NTM5NjRkMWFiZDg0NmJjZjY5MjA0YTMyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzcwNDAxZDI3YTliODQ3YTNhZmMwNmZjMmE2MTVhYmM1ID0gJCgnPGRpdiBpZD0iaHRtbF83MDQwMWQyN2E5Yjg0N2EzYWZjMDZmYzJhNjE1YWJjNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGVhc2lkZSwgRWFzdCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80YTBlNGQ0NDUzOTY0ZDFhYmQ4NDZiY2Y2OTIwNGEzMi5zZXRDb250ZW50KGh0bWxfNzA0MDFkMjdhOWI4NDdhM2FmYzA2ZmMyYTYxNWFiYzUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOWNiMzQ3Y2FiODg5NDliZGI1OTU3ZDRjYzk5MTAzNzguYmluZFBvcHVwKHBvcHVwXzRhMGU0ZDQ0NTM5NjRkMWFiZDg0NmJjZjY5MjA0YTMyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzRmZmZiMmM3OTQ0NDQ3ODQ5YzBkOThiYmVlZDljMTdjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzA1MzY4OSwtNzkuMzQ5MzcxOTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTgyYWE3NjgwMzBlNDYzMWI4NjdjNDk2OTFiMmUyMWYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzVhNTNkMzc5MWI3NGE3Mjk2ODRkMTdhMDRiNWQwNDAgPSAkKCc8ZGl2IGlkPSJodG1sXzM1YTUzZDM3OTFiNzRhNzI5Njg0ZDE3YTA0YjVkMDQwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaG9ybmNsaWZmZSBQYXJrLCBFYXN0IFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2E4MmFhNzY4MDMwZTQ2MzFiODY3YzQ5NjkxYjJlMjFmLnNldENvbnRlbnQoaHRtbF8zNWE1M2QzNzkxYjc0YTcyOTY4NGQxN2EwNGI1ZDA0MCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80ZmZmYjJjNzk0NDQ0Nzg0OWMwZDk4YmJlZWQ5YzE3Yy5iaW5kUG9wdXAocG9wdXBfYTgyYWE3NjgwMzBlNDYzMWI4NjdjNDk2OTFiMmUyMWYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNmVmNzAxNGQ1NGE4NGVjZTljZTk1YmQ3ZTMwZTZiZGMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42ODUzNDcsLTc5LjMzODEwNjVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfODA1ZjNiYzE4YjhjNGRiMTk3NzZhMDJjYmE2NmRjYjMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODBmYjJmZjIwZTliNDdlZDhjMzZiNDI0MjU4ZjEyM2IgPSAkKCc8ZGl2IGlkPSJodG1sXzgwZmIyZmYyMGU5YjQ3ZWQ4YzM2YjQyNDI1OGYxMjNiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FYXN0IFRvcm9udG8sIEJyb2FkdmlldyBOb3J0aCAoT2xkIEVhc3QgWW9yayksIEVhc3QgWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODA1ZjNiYzE4YjhjNGRiMTk3NzZhMDJjYmE2NmRjYjMuc2V0Q29udGVudChodG1sXzgwZmIyZmYyMGU5YjQ3ZWQ4YzM2YjQyNDI1OGYxMjNiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZlZjcwMTRkNTRhODRlY2U5Y2U5NWJkN2UzMGU2YmRjLmJpbmRQb3B1cChwb3B1cF84MDVmM2JjMThiOGM0ZGIxOTc3NmEwMmNiYTY2ZGNiMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wMTdhOWEyMzJiNjM0NTE2YjJkNjZiZTc4NzRjMDY0NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY3OTU1NzEsLTc5LjM1MjE4OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81NjhkOTE1YzdiZTU0ZmI1Yjk0N2E0ZmIyZGVkNmFkMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MTE5N2NmOWEyNDY0ODU3OTQyN2NlYzk0MWMwZThjNyA9ICQoJzxkaXYgaWQ9Imh0bWxfNzExOTdjZjlhMjQ2NDg1Nzk0MjdjZWM5NDFjMGU4YzciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBEYW5mb3J0aCBXZXN0LCBSaXZlcmRhbGUsIEVhc3QgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTY4ZDkxNWM3YmU1NGZiNWI5NDdhNGZiMmRlZDZhZDMuc2V0Q29udGVudChodG1sXzcxMTk3Y2Y5YTI0NjQ4NTc5NDI3Y2VjOTQxYzBlOGM3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzAxN2E5YTIzMmI2MzQ1MTZiMmQ2NmJlNzg3NGMwNjQ0LmJpbmRQb3B1cChwb3B1cF81NjhkOTE1YzdiZTU0ZmI1Yjk0N2E0ZmIyZGVkNmFkMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jODdjMGFkODNhMjM0MWZiOTUzYWEwOWNlNjBiZmYzZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2ODk5ODUsLTc5LjMxNTU3MTU5OTk5OTk4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2FlMDk2ODMzNWJkMTQ4OGFiNDJlY2FhYmM5MDZjOGE4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2FmMzY2ZWQ4ODFmYzQ0NTk4MTAyNzc1N2YxNWUzNzhlID0gJCgnPGRpdiBpZD0iaHRtbF9hZjM2NmVkODgxZmM0NDU5ODEwMjc3NTdmMTVlMzc4ZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SW5kaWEgQmF6YWFyLCBUaGUgQmVhY2hlcyBXZXN0LCBFYXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2FlMDk2ODMzNWJkMTQ4OGFiNDJlY2FhYmM5MDZjOGE4LnNldENvbnRlbnQoaHRtbF9hZjM2NmVkODgxZmM0NDU5ODEwMjc3NTdmMTVlMzc4ZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jODdjMGFkODNhMjM0MWZiOTUzYWEwOWNlNjBiZmYzZC5iaW5kUG9wdXAocG9wdXBfYWUwOTY4MzM1YmQxNDg4YWI0MmVjYWFiYzkwNmM4YTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWZlZDdjNGJiMWVmNDZkNjkzNDk3YjljM2E5YmJkMjkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTk1MjU1LC03OS4zNDA5MjNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOTQ3MDNjYTI4M2U2NDY1NWEyN2FmYmNkYmJjNjIyOTUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGMzMWU1NTU5YWJjNDEzN2I3MjZiMzZkYjBjNmY1MjYgPSAkKCc8ZGl2IGlkPSJodG1sXzRjMzFlNTU1OWFiYzQxMzdiNzI2YjM2ZGIwYzZmNTI2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdHVkaW8gRGlzdHJpY3QsIEVhc3QgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTQ3MDNjYTI4M2U2NDY1NWEyN2FmYmNkYmJjNjIyOTUuc2V0Q29udGVudChodG1sXzRjMzFlNTU1OWFiYzQxMzdiNzI2YjM2ZGIwYzZmNTI2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzFmZWQ3YzRiYjFlZjQ2ZDY5MzQ5N2I5YzNhOWJiZDI5LmJpbmRQb3B1cChwb3B1cF85NDcwM2NhMjgzZTY0NjU1YTI3YWZiY2RiYmM2MjI5NSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hMzIzZDE4OGM3ZGQ0M2U0YjQzNzMzZGEzY2MyNmRkZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcyODAyMDUsLTc5LjM4ODc5MDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMmVlZWFkZTM3N2ZmNDkxYjg3Y2YwZmUzNDFjNWRkMGQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGY1YjQ1MTE5Y2M0NGIxZmE1NDI2ZTI0NmRkMTBjYzQgPSAkKCc8ZGl2IGlkPSJodG1sXzRmNWI0NTExOWNjNDRiMWZhNTQyNmUyNDZkZDEwY2M0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYXdyZW5jZSBQYXJrLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzJlZWVhZGUzNzdmZjQ5MWI4N2NmMGZlMzQxYzVkZDBkLnNldENvbnRlbnQoaHRtbF80ZjViNDUxMTljYzQ0YjFmYTU0MjZlMjQ2ZGQxMGNjNCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hMzIzZDE4OGM3ZGQ0M2U0YjQzNzMzZGEzY2MyNmRkZS5iaW5kUG9wdXAocG9wdXBfMmVlZWFkZTM3N2ZmNDkxYjg3Y2YwZmUzNDFjNWRkMGQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2UxZjcxMWU3MzRhNDkyMzliMjVlNzllYjhmNzE1MDcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTI3NTExLC03OS4zOTAxOTc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U3NTM4YTU0YTI3ZTRhYjFhZjFjOTg0Yjg5NDE5NTg3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzVjMzk0ODk5NTRmYjQ4MjlhMzZiYmU2ZjMxOGI2MmI0ID0gJCgnPGRpdiBpZD0iaHRtbF81YzM5NDg5OTU0ZmI0ODI5YTM2YmJlNmYzMThiNjJiNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGF2aXN2aWxsZSBOb3J0aCwgQ2VudHJhbCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lNzUzOGE1NGEyN2U0YWIxYWYxYzk4NGI4OTQxOTU4Ny5zZXRDb250ZW50KGh0bWxfNWMzOTQ4OTk1NGZiNDgyOWEzNmJiZTZmMzE4YjYyYjQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2UxZjcxMWU3MzRhNDkyMzliMjVlNzllYjhmNzE1MDcuYmluZFBvcHVwKHBvcHVwX2U3NTM4YTU0YTI3ZTRhYjFhZjFjOTg0Yjg5NDE5NTg3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZkZTk5M2M5YzdmZjQxYjZiY2RiMDQ5YjRjYWIxNDA4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzE1MzgzNCwtNzkuNDA1Njc4NDAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMWYzM2NjMDgwZTVmNGVlN2FkMTY1MzhjMTZmOWY3ZTQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjU0YTBhMzIxMGNjNGU4Yjg0OTAwZGMxNmFlNjJiZTMgPSAkKCc8ZGl2IGlkPSJodG1sXzY1NGEwYTMyMTBjYzRlOGI4NDkwMGRjMTZhZTYyYmUzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ob3J0aCBUb3JvbnRvIFdlc3QsICBMYXdyZW5jZSBQYXJrLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzFmMzNjYzA4MGU1ZjRlZTdhZDE2NTM4YzE2ZjlmN2U0LnNldENvbnRlbnQoaHRtbF82NTRhMGEzMjEwY2M0ZThiODQ5MDBkYzE2YWU2MmJlMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mZGU5OTNjOWM3ZmY0MWI2YmNkYjA0OWI0Y2FiMTQwOC5iaW5kUG9wdXAocG9wdXBfMWYzM2NjMDgwZTVmNGVlN2FkMTY1MzhjMTZmOWY3ZTQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMGY2ZjhkODYxYzgzNGFmNmFjZmRlZWYzMDgxZWJmYzcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MDQzMjQ0LC03OS4zODg3OTAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U5NmU0ZjA4MDJhMTQ2NDg5NDBhODRhY2QxNWY3MGI5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzNmNTY3Mjc2ZDMzNjQ4MmZhMTc0ODA4ZjgyNDQyZTliID0gJCgnPGRpdiBpZD0iaHRtbF8zZjU2NzI3NmQzMzY0ODJmYTE3NDgwOGY4MjQ0MmU5YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGF2aXN2aWxsZSwgQ2VudHJhbCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lOTZlNGYwODAyYTE0NjQ4OTQwYTg0YWNkMTVmNzBiOS5zZXRDb250ZW50KGh0bWxfM2Y1NjcyNzZkMzM2NDgyZmExNzQ4MDhmODI0NDJlOWIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMGY2ZjhkODYxYzgzNGFmNmFjZmRlZWYzMDgxZWJmYzcuYmluZFBvcHVwKHBvcHVwX2U5NmU0ZjA4MDJhMTQ2NDg5NDBhODRhY2QxNWY3MGI5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I3ZDIxZjk5MjlmNTQ0OGFhMGUzOTc4NjE4NmYzZWEyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjg5NTc0MywtNzkuMzgzMTU5OTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNjZjYWYzOTJhNTRkNDZkMmEzZGIyMGY4MDUxZDZmMTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjU2NTg3MzU1NzFkNDE2OTg5ZDY1MWNiYWNiOWE0OGUgPSAkKCc8ZGl2IGlkPSJodG1sX2I1NjU4NzM1NTcxZDQxNjk4OWQ2NTFjYmFjYjlhNDhlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb29yZSBQYXJrLCBTdW1tZXJoaWxsIEVhc3QsIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNjZjYWYzOTJhNTRkNDZkMmEzZGIyMGY4MDUxZDZmMTYuc2V0Q29udGVudChodG1sX2I1NjU4NzM1NTcxZDQxNjk4OWQ2NTFjYmFjYjlhNDhlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I3ZDIxZjk5MjlmNTQ0OGFhMGUzOTc4NjE4NmYzZWEyLmJpbmRQb3B1cChwb3B1cF82NmNhZjM5MmE1NGQ0NmQyYTNkYjIwZjgwNTFkNmYxNik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80ZmE0YzdhYmRiYjY0YzkxOTIyYmVlYTc3YjExZGQyOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY4NjQxMjI5OTk5OTk5LC03OS40MDAwNDkzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzdiZTI4MzY5OGQyYjQzMzE5ZWRiNTg5NTY5ZDM2MTE2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzRlN2Q2MTc1NmVlMzRlZGI5ZDM2MDFmNDUyNTE1ODRjID0gJCgnPGRpdiBpZD0iaHRtbF80ZTdkNjE3NTZlZTM0ZWRiOWQzNjAxZjQ1MjUxNTg0YyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3VtbWVyaGlsbCBXZXN0LCBSYXRobmVsbHksIFNvdXRoIEhpbGwsIEZvcmVzdCBIaWxsIFNFLCBEZWVyIFBhcmssIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfN2JlMjgzNjk4ZDJiNDMzMTllZGI1ODk1NjlkMzYxMTYuc2V0Q29udGVudChodG1sXzRlN2Q2MTc1NmVlMzRlZGI5ZDM2MDFmNDUyNTE1ODRjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzRmYTRjN2FiZGJiNjRjOTE5MjJiZWVhNzdiMTFkZDI5LmJpbmRQb3B1cChwb3B1cF83YmUyODM2OThkMmI0MzMxOWVkYjU4OTU2OWQzNjExNik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zMmU1NjIxODU1NWI0ZTlhYjVmNGZmZDJkZWY2YjUxOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY3OTU2MjYsLTc5LjM3NzUyOTQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNjYTViMTBiYmIyYzQ0ZWVhMTFjNDI5YWI4MmJmMjk1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdmNzYzMGJkZmZlNDQxZjJiMDlkNWUwMjE3Zjg2MjYzID0gJCgnPGRpdiBpZD0iaHRtbF83Zjc2MzBiZGZmZTQ0MWYyYjA5ZDVlMDIxN2Y4NjI2MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um9zZWRhbGUsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNjYTViMTBiYmIyYzQ0ZWVhMTFjNDI5YWI4MmJmMjk1LnNldENvbnRlbnQoaHRtbF83Zjc2MzBiZGZmZTQ0MWYyYjA5ZDVlMDIxN2Y4NjI2Myk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zMmU1NjIxODU1NWI0ZTlhYjVmNGZmZDJkZWY2YjUxOS5iaW5kUG9wdXAocG9wdXBfM2NhNWIxMGJiYjJjNDRlZWExMWM0MjlhYjgyYmYyOTUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfODJlZWY4Y2M0Zjg5NDg5NjgyYTliNjBiNTQ1OWFkYzAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Njc5NjcsLTc5LjM2NzY3NTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMzI2N2YwNTcwYjliNDZkMGEyOTg1ZmZmZmQ1MzU0MzYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjgwNzBiMjU5NTQzNGFlNjk5MTU2MjUxZTNmMDQ2MDkgPSAkKCc8ZGl2IGlkPSJodG1sX2Y4MDcwYjI1OTU0MzRhZTY5OTE1NjI1MWUzZjA0NjA5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdC4gSmFtZXMgVG93biwgQ2FiYmFnZXRvd24sIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMyNjdmMDU3MGI5YjQ2ZDBhMjk4NWZmZmZkNTM1NDM2LnNldENvbnRlbnQoaHRtbF9mODA3MGIyNTk1NDM0YWU2OTkxNTYyNTFlM2YwNDYwOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84MmVlZjhjYzRmODk0ODk2ODJhOWI2MGI1NDU5YWRjMC5iaW5kUG9wdXAocG9wdXBfMzI2N2YwNTcwYjliNDZkMGEyOTg1ZmZmZmQ1MzU0MzYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYWVhNDZhZWM3MmYwNDE5NmI0ZjUwNTMxNTZmMDM0OTYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjU4NTk5LC03OS4zODMxNTk5MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZmQ5NjgyZmY4ODY0OWQ4OTY4ZDlkOTQ2NWVhMjE1ZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81MmI4ZDdlNjRjMzI0ZmQwYWRlNjQ4ZjUyODQ0ODg3YiA9ICQoJzxkaXYgaWQ9Imh0bWxfNTJiOGQ3ZTY0YzMyNGZkMGFkZTY0OGY1Mjg0NDg4N2IiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNodXJjaCBhbmQgV2VsbGVzbGV5LCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xZmQ5NjgyZmY4ODY0OWQ4OTY4ZDlkOTQ2NWVhMjE1ZC5zZXRDb250ZW50KGh0bWxfNTJiOGQ3ZTY0YzMyNGZkMGFkZTY0OGY1Mjg0NDg4N2IpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYWVhNDZhZWM3MmYwNDE5NmI0ZjUwNTMxNTZmMDM0OTYuYmluZFBvcHVwKHBvcHVwXzFmZDk2ODJmZjg4NjQ5ZDg5NjhkOWQ5NDY1ZWEyMTVkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQ2YmU2Y2JhMTU5NDQxYjdhYWYwNWI1Njg2NWI5NDlkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjU0MjU5OSwtNzkuMzYwNjM1OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mNmVjOGQzMDM2OTU0YjJhYjJkZDRiNWMyNDExZmU0YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kNmY1NjQ2ZDNhNWY0N2MwOTk3OTU5ZGRjMWIxZjE0NyA9ICQoJzxkaXYgaWQ9Imh0bWxfZDZmNTY0NmQzYTVmNDdjMDk5Nzk1OWRkYzFiMWYxNDciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJlZ2VudCBQYXJrLCBIYXJib3VyZnJvbnQsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2Y2ZWM4ZDMwMzY5NTRiMmFiMmRkNGI1YzI0MTFmZTRiLnNldENvbnRlbnQoaHRtbF9kNmY1NjQ2ZDNhNWY0N2MwOTk3OTU5ZGRjMWIxZjE0Nyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80NmJlNmNiYTE1OTQ0MWI3YWFmMDViNTY4NjViOTQ5ZC5iaW5kUG9wdXAocG9wdXBfZjZlYzhkMzAzNjk1NGIyYWIyZGQ0YjVjMjQxMWZlNGIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDFiMWE2ZDRlZDliNDI2NDg2NTUxNTYyZGJiN2EyYmEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTcxNjE4LC03OS4zNzg5MzcwOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mMGE2NTdiNjNjZWY0OGUzYjFlNTBiZDA0YTY1ZTlmYiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82NWU4NGE5YzhhOGE0NWYxOGNkZjM0YzI0NDg4YWYxZiA9ICQoJzxkaXYgaWQ9Imh0bWxfNjVlODRhOWM4YThhNDVmMThjZGYzNGMyNDQ4OGFmMWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkdhcmRlbiBEaXN0cmljdCwgUnllcnNvbiwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjBhNjU3YjYzY2VmNDhlM2IxZTUwYmQwNGE2NWU5ZmIuc2V0Q29udGVudChodG1sXzY1ZTg0YTljOGE4YTQ1ZjE4Y2RmMzRjMjQ0ODhhZjFmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQxYjFhNmQ0ZWQ5YjQyNjQ4NjU1MTU2MmRiYjdhMmJhLmJpbmRQb3B1cChwb3B1cF9mMGE2NTdiNjNjZWY0OGUzYjFlNTBiZDA0YTY1ZTlmYik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85ODNkYmFmYmJkNTY0YjAxOWQwZmM0NzQ2MjA4ZWRmZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1MTQ5MzksLTc5LjM3NTQxNzldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNGJmNThjMmRlNDgwNGUxNmI5NTYwMTY1MjBiNWRhNGIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNTZlMjA5MGMxMzUyNGFhYmI1ZjRiOWUwMzM0ZmJhOTUgPSAkKCc8ZGl2IGlkPSJodG1sXzU2ZTIwOTBjMTM1MjRhYWJiNWY0YjllMDMzNGZiYTk1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdC4gSmFtZXMgVG93biwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNGJmNThjMmRlNDgwNGUxNmI5NTYwMTY1MjBiNWRhNGIuc2V0Q29udGVudChodG1sXzU2ZTIwOTBjMTM1MjRhYWJiNWY0YjllMDMzNGZiYTk1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk4M2RiYWZiYmQ1NjRiMDE5ZDBmYzQ3NDYyMDhlZGZlLmJpbmRQb3B1cChwb3B1cF80YmY1OGMyZGU0ODA0ZTE2Yjk1NjAxNjUyMGI1ZGE0Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80YTk1Y2VjNzZmMjI0ZDc2YjAyOWI2M2RlODI0NTRkYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0NDc3MDc5OTk5OTk5NiwtNzkuMzczMzA2NF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yZDAwNmY5MmRmYjA0YjJiOTQzODQzM2EwNDcwYjM5NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81NjNjNzQ3MTNiOTY0YjBlYjRkY2Y2MWYwYzkwZjk5NCA9ICQoJzxkaXYgaWQ9Imh0bWxfNTYzYzc0NzEzYjk2NGIwZWI0ZGNmNjFmMGM5MGY5OTQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJlcmN6eSBQYXJrLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yZDAwNmY5MmRmYjA0YjJiOTQzODQzM2EwNDcwYjM5NC5zZXRDb250ZW50KGh0bWxfNTYzYzc0NzEzYjk2NGIwZWI0ZGNmNjFmMGM5MGY5OTQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNGE5NWNlYzc2ZjIyNGQ3NmIwMjliNjNkZTgyNDU0ZGMuYmluZFBvcHVwKHBvcHVwXzJkMDA2ZjkyZGZiMDRiMmI5NDM4NDMzYTA0NzBiMzk0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdjNDBjNmI3N2ZiZTRlMTE4ODYyODM1NjAyMTVhMmQ3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjU3OTUyNCwtNzkuMzg3MzgyNl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZDZiMWE1YjEwZGQ0Njk0OTJkNDkzZmUxNGFiYjAwNiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83YzI5Y2U2YzczMWE0ZmEwYWVmZWM5ZWM5M2E1MWExZSA9ICQoJzxkaXYgaWQ9Imh0bWxfN2MyOWNlNmM3MzFhNGZhMGFlZmVjOWVjOTNhNTFhMWUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNlbnRyYWwgQmF5IFN0cmVldCwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWQ2YjFhNWIxMGRkNDY5NDkyZDQ5M2ZlMTRhYmIwMDYuc2V0Q29udGVudChodG1sXzdjMjljZTZjNzMxYTRmYTBhZWZlYzllYzkzYTUxYTFlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzdjNDBjNmI3N2ZiZTRlMTE4ODYyODM1NjAyMTVhMmQ3LmJpbmRQb3B1cChwb3B1cF8xZDZiMWE1YjEwZGQ0Njk0OTJkNDkzZmUxNGFiYjAwNik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zZTE4MmFhYjU2NDY0NDkxOGQxMmQwYjM2Y2FkYWQzNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1MDU3MTIwMDAwMDAxLC03OS4zODQ1Njc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MyYzM1ODdlNTA2ZjQwYzRhM2I0N2Y2MGMwODZhNGU1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2M1YWI0ZGZkODAyYjQ1MzNiNTBiNTE1OWM4ZWZlODk2ID0gJCgnPGRpdiBpZD0iaHRtbF9jNWFiNGRmZDgwMmI0NTMzYjUwYjUxNTljOGVmZTg5NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UmljaG1vbmQsIEFkZWxhaWRlLCBLaW5nLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jMmMzNTg3ZTUwNmY0MGM0YTNiNDdmNjBjMDg2YTRlNS5zZXRDb250ZW50KGh0bWxfYzVhYjRkZmQ4MDJiNDUzM2I1MGI1MTU5YzhlZmU4OTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2UxODJhYWI1NjQ2NDQ5MThkMTJkMGIzNmNhZGFkMzUuYmluZFBvcHVwKHBvcHVwX2MyYzM1ODdlNTA2ZjQwYzRhM2I0N2Y2MGMwODZhNGU1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzRkOGQ4ZjY4OTEwMTQ3MWFhZTcyM2RkM2ZmZGY0OWZmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQwODE1NywtNzkuMzgxNzUyMjk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTgzYTg3YzMzMTVkNDI1NDhlNDVjOGQ4N2Q2MjE3NTkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZmU1Mzg1NzE1OWQwNGU1MjhkY2IyNjI2Y2QxMjA1MzIgPSAkKCc8ZGl2IGlkPSJodG1sX2ZlNTM4NTcxNTlkMDRlNTI4ZGNiMjYyNmNkMTIwNTMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYXJib3VyZnJvbnQgRWFzdCwgVW5pb24gU3RhdGlvbiwgVG9yb250byBJc2xhbmRzLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81ODNhODdjMzMxNWQ0MjU0OGU0NWM4ZDg3ZDYyMTc1OS5zZXRDb250ZW50KGh0bWxfZmU1Mzg1NzE1OWQwNGU1MjhkY2IyNjI2Y2QxMjA1MzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNGQ4ZDhmNjg5MTAxNDcxYWFlNzIzZGQzZmZkZjQ5ZmYuYmluZFBvcHVwKHBvcHVwXzU4M2E4N2MzMzE1ZDQyNTQ4ZTQ1YzhkODdkNjIxNzU5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2EzZWQwMmE2ZWQ2MjQ1MmRhYjM0ZjIxZTk2NzZjNGYwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ3MTc2OCwtNzkuMzgxNTc2NDAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjZjNjg3MDBkZjY1NGRlZGE4ODAwYjg0OTY2NjQxM2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNWIyMDU0ZTk5YmZjNDQwNTg5YWNkYThjZTljZDhhZGYgPSAkKCc8ZGl2IGlkPSJodG1sXzViMjA1NGU5OWJmYzQ0MDU4OWFjZGE4Y2U5Y2Q4YWRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ub3JvbnRvIERvbWluaW9uIENlbnRyZSwgRGVzaWduIEV4Y2hhbmdlLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iNmM2ODcwMGRmNjU0ZGVkYTg4MDBiODQ5NjY2NDEzYi5zZXRDb250ZW50KGh0bWxfNWIyMDU0ZTk5YmZjNDQwNTg5YWNkYThjZTljZDhhZGYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYTNlZDAyYTZlZDYyNDUyZGFiMzRmMjFlOTY3NmM0ZjAuYmluZFBvcHVwKHBvcHVwX2I2YzY4NzAwZGY2NTRkZWRhODgwMGI4NDk2NjY0MTNiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Q0NzUwMDlhNzMyMDRiYTk4MmU1ZTM3YTdmNDZjZWYzID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4MTk4NSwtNzkuMzc5ODE2OTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWU4ZjZjNzU4MmJhNGZiYWEwNmFlZDg4OTU5ZDBiMWIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjAyY2EzNzg3MWY1NGZjMmE4NDhkMjNmODQxN2ZhZjUgPSAkKCc8ZGl2IGlkPSJodG1sX2IwMmNhMzc4NzFmNTRmYzJhODQ4ZDIzZjg0MTdmYWY1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Db21tZXJjZSBDb3VydCwgVmljdG9yaWEgSG90ZWwsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzllOGY2Yzc1ODJiYTRmYmFhMDZhZWQ4ODk1OWQwYjFiLnNldENvbnRlbnQoaHRtbF9iMDJjYTM3ODcxZjU0ZmMyYTg0OGQyM2Y4NDE3ZmFmNSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kNDc1MDA5YTczMjA0YmE5ODJlNWUzN2E3ZjQ2Y2VmMy5iaW5kUG9wdXAocG9wdXBfOWU4ZjZjNzU4MmJhNGZiYWEwNmFlZDg4OTU5ZDBiMWIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzFlOTY3NGIxZDNkNDBiMDlhNjhkMGNiNmE2ZmNhZDQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MzMyODI1LC03OS40MTk3NDk3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA2YmQ1ZGM2YjllYzQwYjg5MmQyYmJhOTcyODY3YzExID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUzNzg4NjU1NmRkNzQwZDg4MzM3YmM2Yzg3ODk1Y2FjID0gJCgnPGRpdiBpZD0iaHRtbF81Mzc4ODY1NTZkZDc0MGQ4ODMzN2JjNmM4Nzg5NWNhYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmVkZm9yZCBQYXJrLCBMYXdyZW5jZSBNYW5vciBFYXN0LCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wNmJkNWRjNmI5ZWM0MGI4OTJkMmJiYTk3Mjg2N2MxMS5zZXRDb250ZW50KGh0bWxfNTM3ODg2NTU2ZGQ3NDBkODgzMzdiYzZjODc4OTVjYWMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMzFlOTY3NGIxZDNkNDBiMDlhNjhkMGNiNmE2ZmNhZDQuYmluZFBvcHVwKHBvcHVwXzA2YmQ1ZGM2YjllYzQwYjg5MmQyYmJhOTcyODY3YzExKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU5MDBhYmY2ODBlNzQxOWU5MzEyMTE5MWExMDQ5M2I2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzExNjk0OCwtNzkuNDE2OTM1NTk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWM3ZThjYjA0ZjFiNDFlNTk3NmFhZDU1MjQ4Nzc2M2YgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZmMzMTY4ZTBiMDM4NGZiM2EyY2I3YjJjM2JlMGVhODIgPSAkKCc8ZGl2IGlkPSJodG1sX2ZjMzE2OGUwYjAzODRmYjNhMmNiN2IyYzNiZTBlYTgyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NlbGF3biwgQ2VudHJhbCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81YzdlOGNiMDRmMWI0MWU1OTc2YWFkNTUyNDg3NzYzZi5zZXRDb250ZW50KGh0bWxfZmMzMTY4ZTBiMDM4NGZiM2EyY2I3YjJjM2JlMGVhODIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTkwMGFiZjY4MGU3NDE5ZTkzMTIxMTkxYTEwNDkzYjYuYmluZFBvcHVwKHBvcHVwXzVjN2U4Y2IwNGYxYjQxZTU5NzZhYWQ1NTI0ODc3NjNmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzU4NzFlMmQxNGYyYTQ3NGRhZDQ2MWRjYjdmMmM3ZjgxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjk2OTQ3NiwtNzkuNDExMzA3MjAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTkxYjA0ZGI0ZmJmNDg0MDkwM2E4MDI1ZDE0ZmZkODMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNmEyZDk5NTU1MGMyNDRmNjg1NzI5ZjEyNDlmYWY2ZjQgPSAkKCc8ZGl2IGlkPSJodG1sXzZhMmQ5OTU1NTBjMjQ0ZjY4NTcyOWYxMjQ5ZmFmNmY0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Gb3Jlc3QgSGlsbCBOb3J0aCAmYW1wOyBXZXN0LCBGb3Jlc3QgSGlsbCBSb2FkIFBhcmssIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTkxYjA0ZGI0ZmJmNDg0MDkwM2E4MDI1ZDE0ZmZkODMuc2V0Q29udGVudChodG1sXzZhMmQ5OTU1NTBjMjQ0ZjY4NTcyOWYxMjQ5ZmFmNmY0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU4NzFlMmQxNGYyYTQ3NGRhZDQ2MWRjYjdmMmM3ZjgxLmJpbmRQb3B1cChwb3B1cF81OTFiMDRkYjRmYmY0ODQwOTAzYTgwMjVkMTRmZmQ4Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84MTM0ZGQ0MWY3NGY0OTAyODQ1ZWNlNDJiYTVkNTZlZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY3MjcwOTcsLTc5LjQwNTY3ODQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2ViMTcxZjdlNWRjZTRmOTRhZDYxNjE2ZjdmMWU0NTEzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzc4MWM5ZjI1ODE5MTRmOTU4YzExZjFlZTZlNTIzYzY0ID0gJCgnPGRpdiBpZD0iaHRtbF83ODFjOWYyNTgxOTE0Zjk1OGMxMWYxZWU2ZTUyM2M2NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIEFubmV4LCBOb3J0aCBNaWR0b3duLCBZb3JrdmlsbGUsIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWIxNzFmN2U1ZGNlNGY5NGFkNjE2MTZmN2YxZTQ1MTMuc2V0Q29udGVudChodG1sXzc4MWM5ZjI1ODE5MTRmOTU4YzExZjFlZTZlNTIzYzY0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzgxMzRkZDQxZjc0ZjQ5MDI4NDVlY2U0MmJhNWQ1NmVmLmJpbmRQb3B1cChwb3B1cF9lYjE3MWY3ZTVkY2U0Zjk0YWQ2MTYxNmY3ZjFlNDUxMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kZTRiZjM4NGU5MTE0Mjg3YTJiNThiNTViZjQyMzM3NiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2MjY5NTYsLTc5LjQwMDA0OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTZmNjE5ZGNjOTRkNDYyNGEzYWE0MmYwODQ5ZDRmYmUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfM2QxZWYwOWNmZmIwNGMxNTk5OTI1Y2I4MjFlNTQyZDIgPSAkKCc8ZGl2IGlkPSJodG1sXzNkMWVmMDljZmZiMDRjMTU5OTkyNWNiODIxZTU0MmQyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Vbml2ZXJzaXR5IG9mIFRvcm9udG8sIEhhcmJvcmQsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2E2ZjYxOWRjYzk0ZDQ2MjRhM2FhNDJmMDg0OWQ0ZmJlLnNldENvbnRlbnQoaHRtbF8zZDFlZjA5Y2ZmYjA0YzE1OTk5MjVjYjgyMWU1NDJkMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kZTRiZjM4NGU5MTE0Mjg3YTJiNThiNTViZjQyMzM3Ni5iaW5kUG9wdXAocG9wdXBfYTZmNjE5ZGNjOTRkNDYyNGEzYWE0MmYwODQ5ZDRmYmUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMjAxNWU1ODYwY2Q2NDA0YjkxZDBiZTUyMDE5NWU5ODQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTMyMDU3LC03OS40MDAwNDkzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Y2YTNiMDEzYTdlZTQ0YzA5NGE5ZDM2NTVjYjNlYWU4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzMyZWMzNzExOWMxMTRhOGJiNDQ0MGRlOGRjODM4MjZmID0gJCgnPGRpdiBpZD0iaHRtbF8zMmVjMzcxMTljMTE0YThiYjQ0NDBkZThkYzgzODI2ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+S2Vuc2luZ3RvbiBNYXJrZXQsIENoaW5hdG93biwgR3JhbmdlIFBhcmssIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2Y2YTNiMDEzYTdlZTQ0YzA5NGE5ZDM2NTVjYjNlYWU4LnNldENvbnRlbnQoaHRtbF8zMmVjMzcxMTljMTE0YThiYjQ0NDBkZThkYzgzODI2Zik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yMDE1ZTU4NjBjZDY0MDRiOTFkMGJlNTIwMTk1ZTk4NC5iaW5kUG9wdXAocG9wdXBfZjZhM2IwMTNhN2VlNDRjMDk0YTlkMzY1NWNiM2VhZTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfODFkNzg2NTU3NDcwNDljZTkxMTNjM2E4MjI3ODlkNzAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Mjg5NDY3LC03OS4zOTQ0MTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzUxY2IzNDg0NGQ4MTQ4MjlhZjRjMjRjYzFjZTExMzMyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2M5MTdiMWUxNmRiNzQxMWNiN2U5YTA3OGJiNmQzZTE3ID0gJCgnPGRpdiBpZD0iaHRtbF9jOTE3YjFlMTZkYjc0MTFjYjdlOWEwNzhiYjZkM2UxNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q04gVG93ZXIsIEtpbmcgYW5kIFNwYWRpbmEsIFJhaWx3YXkgTGFuZHMsIEhhcmJvdXJmcm9udCBXZXN0LCBCYXRodXJzdCBRdWF5LCBTb3V0aCBOaWFnYXJhLCBJc2xhbmQgYWlycG9ydCwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTFjYjM0ODQ0ZDgxNDgyOWFmNGMyNGNjMWNlMTEzMzIuc2V0Q29udGVudChodG1sX2M5MTdiMWUxNmRiNzQxMWNiN2U5YTA3OGJiNmQzZTE3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzgxZDc4NjU1NzQ3MDQ5Y2U5MTEzYzNhODIyNzg5ZDcwLmJpbmRQb3B1cChwb3B1cF81MWNiMzQ4NDRkODE0ODI5YWY0YzI0Y2MxY2UxMTMzMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jN2QzMDNhOGNjZTU0YjcxYWIwZDFhYTcwZjMyMzI0ZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0NjQzNTIsLTc5LjM3NDg0NTk5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzcwODVlOGY3ODk3NzQyYWZiMTRlYTIyY2Q0ZjczMDRiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2NmYzA4YTBiYTIyMzQ1NzdhODk2ZjkxODljOTE5NjdjID0gJCgnPGRpdiBpZD0iaHRtbF9jZmMwOGEwYmEyMjM0NTc3YTg5NmY5MTg5YzkxOTY3YyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3RuIEEgUE8gQm94ZXMsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzcwODVlOGY3ODk3NzQyYWZiMTRlYTIyY2Q0ZjczMDRiLnNldENvbnRlbnQoaHRtbF9jZmMwOGEwYmEyMjM0NTc3YTg5NmY5MTg5YzkxOTY3Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jN2QzMDNhOGNjZTU0YjcxYWIwZDFhYTcwZjMyMzI0ZS5iaW5kUG9wdXAocG9wdXBfNzA4NWU4Zjc4OTc3NDJhZmIxNGVhMjJjZDRmNzMwNGIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMmQ0OGI0YTk0ZmY0NGM4YTlmMGQ4NDQzYjAxMGIwMTcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDg0MjkyLC03OS4zODIyODAyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA2MjNkZmE3NjM0ZjQ0ZDJhOGY3NzI3NGVlZGFmZDM3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzRhNDNlMTljMDAzYjQ5OGRhYTVhMjE2Yzg3YTJmMDdkID0gJCgnPGRpdiBpZD0iaHRtbF80YTQzZTE5YzAwM2I0OThkYWE1YTIxNmM4N2EyZjA3ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rmlyc3QgQ2FuYWRpYW4gUGxhY2UsIFVuZGVyZ3JvdW5kIGNpdHksIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzA2MjNkZmE3NjM0ZjQ0ZDJhOGY3NzI3NGVlZGFmZDM3LnNldENvbnRlbnQoaHRtbF80YTQzZTE5YzAwM2I0OThkYWE1YTIxNmM4N2EyZjA3ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yZDQ4YjRhOTRmZjQ0YzhhOWYwZDg0NDNiMDEwYjAxNy5iaW5kUG9wdXAocG9wdXBfMDYyM2RmYTc2MzRmNDRkMmE4Zjc3Mjc0ZWVkYWZkMzcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZjFmMjFkOWIzMjA5NDFlOGJhNWNlOWFkY2FiMzE4NDkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTg1MTc5OTk5OTk5OTYsLTc5LjQ2NDc2MzI5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzczMTk4ZDhiYThiNjRiOGE4NDE0NjdjZTA1M2ZmM2ZjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ0YmI5YTE5ZDcyZTQ4ZmRiMmI5YzhiNzk2MzkyMThmID0gJCgnPGRpdiBpZD0iaHRtbF80NGJiOWExOWQ3MmU0OGZkYjJiOWM4Yjc5NjM5MjE4ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGF3cmVuY2UgTWFub3IsIExhd3JlbmNlIEhlaWdodHMsIE5vcnRoIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzczMTk4ZDhiYThiNjRiOGE4NDE0NjdjZTA1M2ZmM2ZjLnNldENvbnRlbnQoaHRtbF80NGJiOWExOWQ3MmU0OGZkYjJiOWM4Yjc5NjM5MjE4Zik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mMWYyMWQ5YjMyMDk0MWU4YmE1Y2U5YWRjYWIzMTg0OS5iaW5kUG9wdXAocG9wdXBfNzMxOThkOGJhOGI2NGI4YTg0MTQ2N2NlMDUzZmYzZmMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTk1ODlhZDY3ZjllNGU2ZjgxN2IwYmFlYmZhZTIwYmEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MDk1NzcsLTc5LjQ0NTA3MjU5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY5NDU1ZTkxNWI5ZTRjYTc5NTEyYTkwNzdmN2Y4MDk5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2RjYTBiY2ZmMDQ5YzRkY2I4NWRlOGEwNzc4MjRkZDgzID0gJCgnPGRpdiBpZD0iaHRtbF9kY2EwYmNmZjA0OWM0ZGNiODVkZThhMDc3ODI0ZGQ4MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2xlbmNhaXJuLCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82OTQ1NWU5MTViOWU0Y2E3OTUxMmE5MDc3ZjdmODA5OS5zZXRDb250ZW50KGh0bWxfZGNhMGJjZmYwNDljNGRjYjg1ZGU4YTA3NzgyNGRkODMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTk1ODlhZDY3ZjllNGU2ZjgxN2IwYmFlYmZhZTIwYmEuYmluZFBvcHVwKHBvcHVwXzY5NDU1ZTkxNWI5ZTRjYTc5NTEyYTkwNzdmN2Y4MDk5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2E4ZWEwY2U0Mjg3ZDRjY2FhMWMyNzdhMjI0MGY0OGYxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjkzNzgxMywtNzkuNDI4MTkxNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYWNmNzYyZDJlNjFiNDQ2NWE2OWM2NTI4NGYwNjhhY2IgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjc3YjY3MmViYTRiNGI1OGI5NmU5MTRlZGZkMjM2NzYgPSAkKCc8ZGl2IGlkPSJodG1sXzI3N2I2NzJlYmE0YjRiNThiOTZlOTE0ZWRmZDIzNjc2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IdW1ld29vZC1DZWRhcnZhbGUsIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2FjZjc2MmQyZTYxYjQ0NjVhNjljNjUyODRmMDY4YWNiLnNldENvbnRlbnQoaHRtbF8yNzdiNjcyZWJhNGI0YjU4Yjk2ZTkxNGVkZmQyMzY3Nik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hOGVhMGNlNDI4N2Q0Y2NhYTFjMjc3YTIyNDBmNDhmMS5iaW5kUG9wdXAocG9wdXBfYWNmNzYyZDJlNjFiNDQ2NWE2OWM2NTI4NGYwNjhhY2IpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2I3ZTE0YjAxMmFjNGQzYWJiODVhMDIxZWU5MTVmYzUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42ODkwMjU2LC03OS40NTM1MTJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjIzZDc2ZDk5NGUxNGQ0MTlmNjJjYjI1ZWQ5NzUyNjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjRmNzViNmE2MjE1NGZmNTk4Mzg2Y2M4MTUwYzNhZTYgPSAkKCc8ZGl2IGlkPSJodG1sX2Y0Zjc1YjZhNjIxNTRmZjU5ODM4NmNjODE1MGMzYWU2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYWxlZG9uaWEtRmFpcmJhbmtzLCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iMjNkNzZkOTk0ZTE0ZDQxOWY2MmNiMjVlZDk3NTI2MC5zZXRDb250ZW50KGh0bWxfZjRmNzViNmE2MjE1NGZmNTk4Mzg2Y2M4MTUwYzNhZTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2I3ZTE0YjAxMmFjNGQzYWJiODVhMDIxZWU5MTVmYzUuYmluZFBvcHVwKHBvcHVwX2IyM2Q3NmQ5OTRlMTRkNDE5ZjYyY2IyNWVkOTc1MjYwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM2ZGZjOTYzNzIyZTRmNWY4NTAxMjQ0MWUwN2Y4ZWQwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY5NTQyLC03OS40MjI1NjM3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzk1MmUyYzYzMmE1NTQ3ZTA4MWExZjU1NzYxNmMyNThmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE2NWNjOTA4YzE3NDRhMTBiN2Q4YTNiOTk4MGMxN2FmID0gJCgnPGRpdiBpZD0iaHRtbF8xNjVjYzkwOGMxNzQ0YTEwYjdkOGEzYjk5ODBjMTdhZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hyaXN0aWUsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzk1MmUyYzYzMmE1NTQ3ZTA4MWExZjU1NzYxNmMyNThmLnNldENvbnRlbnQoaHRtbF8xNjVjYzkwOGMxNzQ0YTEwYjdkOGEzYjk5ODBjMTdhZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zNmRmYzk2MzcyMmU0ZjVmODUwMTI0NDFlMDdmOGVkMC5iaW5kUG9wdXAocG9wdXBfOTUyZTJjNjMyYTU1NDdlMDgxYTFmNTU3NjE2YzI1OGYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDlhNzdkNGJhZWIxNGQxNzhlZWI1ZDgxZmYzOTU2NzkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjkwMDUxMDAwMDAwMSwtNzkuNDQyMjU5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zZWM4MDE4Nzk1MDg0MWE0YjJlNDI4YjViYmQxYjIzOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lZThlNjdiNzI0Yjk0OTliYTMzNmEzNGUyNTllOThkMCA9ICQoJzxkaXYgaWQ9Imh0bWxfZWU4ZTY3YjcyNGI5NDk5YmEzMzZhMzRlMjU5ZTk4ZDAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkR1ZmZlcmluLCBEb3ZlcmNvdXJ0IFZpbGxhZ2UsIFdlc3QgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfM2VjODAxODc5NTA4NDFhNGIyZTQyOGI1YmJkMWIyMzguc2V0Q29udGVudChodG1sX2VlOGU2N2I3MjRiOTQ5OWJhMzM2YTM0ZTI1OWU5OGQwKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQ5YTc3ZDRiYWViMTRkMTc4ZWViNWQ4MWZmMzk1Njc5LmJpbmRQb3B1cChwb3B1cF8zZWM4MDE4Nzk1MDg0MWE0YjJlNDI4YjViYmQxYjIzOCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81N2IxMGJlZDExMjI0NDNmOGMwMzhhMGE1NzZjMTY1YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0NzkyNjcwMDAwMDAwNiwtNzkuNDE5NzQ5N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hODcwYTlkMzM1Yjc0NjhlYTg2ZmZmZDgzODQ5ZDMyOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kZWIwMjAwZjIzNTA0Y2QwYWNlY2RhZTA3ZTI4NWE3YSA9ICQoJzxkaXYgaWQ9Imh0bWxfZGViMDIwMGYyMzUwNGNkMGFjZWNkYWUwN2UyODVhN2EiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxpdHRsZSBQb3J0dWdhbCwgVHJpbml0eSwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9hODcwYTlkMzM1Yjc0NjhlYTg2ZmZmZDgzODQ5ZDMyOS5zZXRDb250ZW50KGh0bWxfZGViMDIwMGYyMzUwNGNkMGFjZWNkYWUwN2UyODVhN2EpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTdiMTBiZWQxMTIyNDQzZjhjMDM4YTBhNTc2YzE2NWMuYmluZFBvcHVwKHBvcHVwX2E4NzBhOWQzMzViNzQ2OGVhODZmZmZkODM4NDlkMzI5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzUyMjVlMDMwOWU2MjRmZDBiMDk4N2I4ZTE4YWFhZTkyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjM2ODQ3MiwtNzkuNDI4MTkxNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNzM3YzgzN2FkOWEzNDBkYWJiMjQzY2IzNTU0NWRkYTEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjk1NWExMGI3ZjkyNGNlMDk5MDg3YTFjZjY0MWRlNWEgPSAkKCc8ZGl2IGlkPSJodG1sX2I5NTVhMTBiN2Y5MjRjZTA5OTA4N2ExY2Y2NDFkZTVhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ccm9ja3RvbiwgUGFya2RhbGUgVmlsbGFnZSwgRXhoaWJpdGlvbiBQbGFjZSwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83MzdjODM3YWQ5YTM0MGRhYmIyNDNjYjM1NTQ1ZGRhMS5zZXRDb250ZW50KGh0bWxfYjk1NWExMGI3ZjkyNGNlMDk5MDg3YTFjZjY0MWRlNWEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTIyNWUwMzA5ZTYyNGZkMGIwOTg3YjhlMThhYWFlOTIuYmluZFBvcHVwKHBvcHVwXzczN2M4MzdhZDlhMzQwZGFiYjI0M2NiMzU1NDVkZGExKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzIyMWU2MmM2YjRiYzRmYTE5ZWExNjgyNjI1MTIyOGI1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzEzNzU2MjAwMDAwMDA2LC03OS40OTAwNzM4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzM3ODU0MGM4NmUxNzRhNjk4OGNjOWM4OTcwN2NhNzhkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzg0YmM5NGMwNThhMDRkYWU4ODVlN2NiNzFkMTFjNWU1ID0gJCgnPGRpdiBpZD0iaHRtbF84NGJjOTRjMDU4YTA0ZGFlODg1ZTdjYjcxZDExYzVlNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Tm9ydGggUGFyaywgTWFwbGUgTGVhZiBQYXJrLCBVcHdvb2QgUGFyaywgTm9ydGggWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMzc4NTQwYzg2ZTE3NGE2OTg4Y2M5Yzg5NzA3Y2E3OGQuc2V0Q29udGVudChodG1sXzg0YmM5NGMwNThhMDRkYWU4ODVlN2NiNzFkMTFjNWU1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzIyMWU2MmM2YjRiYzRmYTE5ZWExNjgyNjI1MTIyOGI1LmJpbmRQb3B1cChwb3B1cF8zNzg1NDBjODZlMTc0YTY5ODhjYzljODk3MDdjYTc4ZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kODgzYWJmYWFkZGQ0NDFlOTk4ZTNmODFkMGUzNzc4MCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5MTExNTgsLTc5LjQ3NjAxMzI5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzk5NTIyMTA2ODEwYTRiOTY5ZDJkMGVjZGRkNTQxNjY4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE1YjBhNjEzMGExNTQ4YTM5ZmM4MDYwZWY0YzlmOTZhID0gJCgnPGRpdiBpZD0iaHRtbF8xNWIwYTYxMzBhMTU0OGEzOWZjODA2MGVmNGM5Zjk2YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGVsIFJheSwgTW91bnQgRGVubmlzLCBLZWVsc2RhbGUgYW5kIFNpbHZlcnRob3JuLCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85OTUyMjEwNjgxMGE0Yjk2OWQyZDBlY2RkZDU0MTY2OC5zZXRDb250ZW50KGh0bWxfMTViMGE2MTMwYTE1NDhhMzlmYzgwNjBlZjRjOWY5NmEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZDg4M2FiZmFhZGRkNDQxZTk5OGUzZjgxZDBlMzc3ODAuYmluZFBvcHVwKHBvcHVwXzk5NTIyMTA2ODEwYTRiOTY5ZDJkMGVjZGRkNTQxNjY4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI4MWIzZjNlMjU3NzQ1NmM4N2U4NmQ4ODZhODRjODZhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjczMTg1Mjk5OTk5OTksLTc5LjQ4NzI2MTkwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY5NGU3NTAxYjIwYTRiOTdhMzJhMGRhNjFkYzc4NWM0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzlmNWJmZGMxZmMwMDQ2NTI4ZjU5MTcyZTMyNDVlZjNlID0gJCgnPGRpdiBpZD0iaHRtbF85ZjViZmRjMWZjMDA0NjUyOGY1OTE3MmUzMjQ1ZWYzZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UnVubnltZWRlLCBUaGUgSnVuY3Rpb24gTm9ydGgsIFlvcms8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY5NGU3NTAxYjIwYTRiOTdhMzJhMGRhNjFkYzc4NWM0LnNldENvbnRlbnQoaHRtbF85ZjViZmRjMWZjMDA0NjUyOGY1OTE3MmUzMjQ1ZWYzZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yODFiM2YzZTI1Nzc0NTZjODdlODZkODg2YTg0Yzg2YS5iaW5kUG9wdXAocG9wdXBfNjk0ZTc1MDFiMjBhNGI5N2EzMmEwZGE2MWRjNzg1YzQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDhlZDI4ODBlZDEyNDU2NzhiN2U5N2EyYzFhNzg1ZmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjE2MDgzLC03OS40NjQ3NjMyOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xMDEwYzI2MWJmYzY0ODFlOGFmYmYxMGVmNjc3ZTA4NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kMjQ2OTRiOTRlZDU0NjI5YjRjYTNmZjY3MzYxYzRlMCA9ICQoJzxkaXYgaWQ9Imh0bWxfZDI0Njk0Yjk0ZWQ1NDYyOWI0Y2EzZmY2NzM2MWM0ZTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhpZ2ggUGFyaywgVGhlIEp1bmN0aW9uIFNvdXRoLCBXZXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzEwMTBjMjYxYmZjNjQ4MWU4YWZiZjEwZWY2NzdlMDg0LnNldENvbnRlbnQoaHRtbF9kMjQ2OTRiOTRlZDU0NjI5YjRjYTNmZjY3MzYxYzRlMCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kOGVkMjg4MGVkMTI0NTY3OGI3ZTk3YTJjMWE3ODVmYi5iaW5kUG9wdXAocG9wdXBfMTAxMGMyNjFiZmM2NDgxZThhZmJmMTBlZjY3N2UwODQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYmUzMGZkZWRiYzQ3NDgzM2FkN2RhMThlYzk5NGZiNTQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDg5NTk3LC03OS40NTYzMjVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjY2NmQ4MzM3MmY3NGY1Nzk3N2M5YWRhMzcyYjhiYmMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDhmODAwYjNhYjhiNGNjYjliMTc4ZDVkM2U2Y2U0MTkgPSAkKCc8ZGl2IGlkPSJodG1sX2Q4ZjgwMGIzYWI4YjRjY2I5YjE3OGQ1ZDNlNmNlNDE5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QYXJrZGFsZSwgUm9uY2VzdmFsbGVzLCBXZXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2I2NjZkODMzNzJmNzRmNTc5NzdjOWFkYTM3MmI4YmJjLnNldENvbnRlbnQoaHRtbF9kOGY4MDBiM2FiOGI0Y2NiOWIxNzhkNWQzZTZjZTQxOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iZTMwZmRlZGJjNDc0ODMzYWQ3ZGExOGVjOTk0ZmI1NC5iaW5kUG9wdXAocG9wdXBfYjY2NmQ4MzM3MmY3NGY1Nzk3N2M5YWRhMzcyYjhiYmMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZmZjNWE0ZmJhYTc0NDE5NmI1MDBmNWJmNmUzYjFmM2QgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTE1NzA2LC03OS40ODQ0NDk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzk4ZDM0MTVhM2QxYTRjMjc5NWFkOWI1NmE0NGU4MTlmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzJjMGQzNzA2YWJmMzQ4ODViODNmYmE2OWVkNGUyNGIyID0gJCgnPGRpdiBpZD0iaHRtbF8yYzBkMzcwNmFiZjM0ODg1YjgzZmJhNjllZDRlMjRiMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UnVubnltZWRlLCBTd2Fuc2VhLCBXZXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzk4ZDM0MTVhM2QxYTRjMjc5NWFkOWI1NmE0NGU4MTlmLnNldENvbnRlbnQoaHRtbF8yYzBkMzcwNmFiZjM0ODg1YjgzZmJhNjllZDRlMjRiMik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mZmM1YTRmYmFhNzQ0MTk2YjUwMGY1YmY2ZTNiMWYzZC5iaW5kUG9wdXAocG9wdXBfOThkMzQxNWEzZDFhNGMyNzk1YWQ5YjU2YTQ0ZTgxOWYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNzlhM2RiYTVhNzk0NDBkNzkzOWY4OTlkZmNkNzdhNmYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjIzMDE1LC03OS4zODk0OTM4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA1YmM5MDVjMTJiMTRlYmY4MDIyNzFkMmYwNWI4OTA2ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2U1NTQ1MjkxMTM4ZjRiMzRiZGI4MjZkZWVkZjM2MTE5ID0gJCgnPGRpdiBpZD0iaHRtbF9lNTU0NTI5MTEzOGY0YjM0YmRiODI2ZGVlZGYzNjExOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UXVlZW4mIzM5O3MgUGFyaywgT250YXJpbyBQcm92aW5jaWFsIEdvdmVybm1lbnQsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzA1YmM5MDVjMTJiMTRlYmY4MDIyNzFkMmYwNWI4OTA2LnNldENvbnRlbnQoaHRtbF9lNTU0NTI5MTEzOGY0YjM0YmRiODI2ZGVlZGYzNjExOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83OWEzZGJhNWE3OTQ0MGQ3OTM5Zjg5OWRmY2Q3N2E2Zi5iaW5kUG9wdXAocG9wdXBfMDViYzkwNWMxMmIxNGViZjgwMjI3MWQyZjA1Yjg5MDYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTY2YjI5ODZhNmYyNDQxNmI1M2MyNmZhMWRmMjI0NmYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42MzY5NjU2LC03OS42MTU4MTg5OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80NDkyMmI1YzMzZDQ0OGYyODliZWEzNThiNzRlYTkyMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83ZmIwNDMzNmFmNjI0NTNjYTcxYzI3MDQwNGNmMDY1ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfN2ZiMDQzMzZhZjYyNDUzY2E3MWMyNzA0MDRjZjA2NWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhbmFkYSBQb3N0IEdhdGV3YXkgUHJvY2Vzc2luZyBDZW50cmUsIE1pc3Npc3NhdWdhPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80NDkyMmI1YzMzZDQ0OGYyODliZWEzNThiNzRlYTkyMC5zZXRDb250ZW50KGh0bWxfN2ZiMDQzMzZhZjYyNDUzY2E3MWMyNzA0MDRjZjA2NWQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTY2YjI5ODZhNmYyNDQxNmI1M2MyNmZhMWRmMjI0NmYuYmluZFBvcHVwKHBvcHVwXzQ0OTIyYjVjMzNkNDQ4ZjI4OWJlYTM1OGI3NGVhOTIwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzljNTNhMzE4NTQ3YjRhZmU4MWExNjI1OWVjMTdjMWIyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyNzQzOSwtNzkuMzIxNTU4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzJkNTlhYjEyOGMyNDRlMTBiNGM2MTJiNzEwOTc1NDA5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzhlYTk1NmEwMGNhMzRiZTk5ODIzMjQ3OGNmNTBiZjM3ID0gJCgnPGRpdiBpZD0iaHRtbF84ZWE5NTZhMDBjYTM0YmU5OTgyMzI0NzhjZjUwYmYzNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnVzaW5lc3MgcmVwbHkgbWFpbCBQcm9jZXNzaW5nIENlbnRyZSwgU291dGggQ2VudHJhbCBMZXR0ZXIgUHJvY2Vzc2luZyBQbGFudCBUb3JvbnRvLCBFYXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzJkNTlhYjEyOGMyNDRlMTBiNGM2MTJiNzEwOTc1NDA5LnNldENvbnRlbnQoaHRtbF84ZWE5NTZhMDBjYTM0YmU5OTgyMzI0NzhjZjUwYmYzNyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85YzUzYTMxODU0N2I0YWZlODFhMTYyNTllYzE3YzFiMi5iaW5kUG9wdXAocG9wdXBfMmQ1OWFiMTI4YzI0NGUxMGI0YzYxMmI3MTA5NzU0MDkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjg4YTFlMmIzNGQyNDViZThkMTIyZDNjNjA0MzY5YTEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42MDU2NDY2LC03OS41MDEzMjA3MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMGRlNDYwZjFlYjI0MGRlOTMxNTE5Zjk1YTcwYzQ0NCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNmEwMTU2M2VjZmI0OTE1OGJjZmQ1YzRjMjgyNDIwZCA9ICQoJzxkaXYgaWQ9Imh0bWxfMzZhMDE1NjNlY2ZiNDkxNThiY2ZkNWM0YzI4MjQyMGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5ldyBUb3JvbnRvLCBNaW1pY28gU291dGgsIEh1bWJlciBCYXkgU2hvcmVzLCBFdG9iaWNva2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2IwZGU0NjBmMWViMjQwZGU5MzE1MTlmOTVhNzBjNDQ0LnNldENvbnRlbnQoaHRtbF8zNmEwMTU2M2VjZmI0OTE1OGJjZmQ1YzRjMjgyNDIwZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iODhhMWUyYjM0ZDI0NWJlOGQxMjJkM2M2MDQzNjlhMS5iaW5kUG9wdXAocG9wdXBfYjBkZTQ2MGYxZWIyNDBkZTkzMTUxOWY5NWE3MGM0NDQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNmFmYjFjYjMyZDU3NDg2MGEwYjgyODI4YzY5NDExMDUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42MDI0MTM3MDAwMDAwMSwtNzkuNTQzNDg0MDk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2Y2N2Y0ZDhmMDA0NGJhNWJhODRhOGMxNjdlOTVlNDIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjRjMGFhZDc5ZDM4NGRjY2I4NDFiMjRjOWEzMWQyM2YgPSAkKCc8ZGl2IGlkPSJodG1sX2Y0YzBhYWQ3OWQzODRkY2NiODQxYjI0YzlhMzFkMjNmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbGRlcndvb2QsIExvbmcgQnJhbmNoLCBFdG9iaWNva2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNmNjdmNGQ4ZjAwNDRiYTViYTg0YThjMTY3ZTk1ZTQyLnNldENvbnRlbnQoaHRtbF9mNGMwYWFkNzlkMzg0ZGNjYjg0MWIyNGM5YTMxZDIzZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl82YWZiMWNiMzJkNTc0ODYwYTBiODI4MjhjNjk0MTEwNS5iaW5kUG9wdXAocG9wdXBfM2Y2N2Y0ZDhmMDA0NGJhNWJhODRhOGMxNjdlOTVlNDIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2UwNGQ4ZGE4NzkxNDYzYWJiZjE5MjlkYTI1MDVmYzIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTM2NTM2MDAwMDAwMDUsLTc5LjUwNjk0MzZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZGIxMTQ5NWZiNDU5NDYxMTk2MDIzMTA1OGU1MzAwZjggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGE1M2M5NzgxYjJiNGJmZDllYWU1ZmEzNTRkNmQwZTQgPSAkKCc8ZGl2IGlkPSJodG1sXzRhNTNjOTc4MWIyYjRiZmQ5ZWFlNWZhMzU0ZDZkMGU0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgS2luZ3N3YXksIE1vbnRnb21lcnkgUm9hZCwgT2xkIE1pbGwgTm9ydGgsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZGIxMTQ5NWZiNDU5NDYxMTk2MDIzMTA1OGU1MzAwZjguc2V0Q29udGVudChodG1sXzRhNTNjOTc4MWIyYjRiZmQ5ZWFlNWZhMzU0ZDZkMGU0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNlMDRkOGRhODc5MTQ2M2FiYmYxOTI5ZGEyNTA1ZmMyLmJpbmRQb3B1cChwb3B1cF9kYjExNDk1ZmI0NTk0NjExOTYwMjMxMDU4ZTUzMDBmOCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80MjQ2YWVjYjg3ZWE0YTk4YTA2OWU0YWU2ZDE4NjM5YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjYzNjI1NzksLTc5LjQ5ODUwOTA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzZjOWRhNmY0OTg1YTQ0NzNhMThmYjBmYjZiOTBkNDQwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzAzNGMwYWFjZjgwZTQzODdiNzU3MjEyYjk5NDMwNjliID0gJCgnPGRpdiBpZD0iaHRtbF8wMzRjMGFhY2Y4MGU0Mzg3Yjc1NzIxMmI5OTQzMDY5YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+T2xkIE1pbGwgU291dGgsIEtpbmcmIzM5O3MgTWlsbCBQYXJrLCBTdW5ueWxlYSwgSHVtYmVyIEJheSwgTWltaWNvIE5FLCBUaGUgUXVlZW5zd2F5IEVhc3QsIFJveWFsIFlvcmsgU291dGggRWFzdCwgS2luZ3N3YXkgUGFyayBTb3V0aCBFYXN0LCBFdG9iaWNva2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzZjOWRhNmY0OTg1YTQ0NzNhMThmYjBmYjZiOTBkNDQwLnNldENvbnRlbnQoaHRtbF8wMzRjMGFhY2Y4MGU0Mzg3Yjc1NzIxMmI5OTQzMDY5Yik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80MjQ2YWVjYjg3ZWE0YTk4YTA2OWU0YWU2ZDE4NjM5Yy5iaW5kUG9wdXAocG9wdXBfNmM5ZGE2ZjQ5ODVhNDQ3M2ExOGZiMGZiNmI5MGQ0NDApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjI5YjE3Yzg5OThmNGQyMWFlYWMzM2EwYjhjNTg4MTYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Mjg4NDA4LC03OS41MjA5OTk0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80OTMzYWU0YzFlNzM0YmJmYjNmYTFmYjRiZGVlN2IzMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kNTY1Mzk2MjVmNTY0MzZjYmE5YWE5ZTdhYTI5ZDc1ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfZDU2NTM5NjI1ZjU2NDM2Y2JhOWFhOWU3YWEyOWQ3NWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1pbWljbyBOVywgVGhlIFF1ZWVuc3dheSBXZXN0LCBTb3V0aCBvZiBCbG9vciwgS2luZ3N3YXkgUGFyayBTb3V0aCBXZXN0LCBSb3lhbCBZb3JrIFNvdXRoIFdlc3QsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDkzM2FlNGMxZTczNGJiZmIzZmExZmI0YmRlZTdiMzIuc2V0Q29udGVudChodG1sX2Q1NjUzOTYyNWY1NjQzNmNiYTlhYTllN2FhMjlkNzVkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzYyOWIxN2M4OTk4ZjRkMjFhZWFjMzNhMGI4YzU4ODE2LmJpbmRQb3B1cChwb3B1cF80OTMzYWU0YzFlNzM0YmJmYjNmYTFmYjRiZGVlN2IzMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lMDljNjlhZjEwYzM0ZTY3YjBlMzFmMzQxZjMyYjVhZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2Nzg1NTYsLTc5LjUzMjI0MjQwMDAwMDAyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzVhMDk1ZjhhMzJlYjRmMzFhYTBhYzZhYzllZDg5OTAxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY4YjM1YmIwMTlhYTQ0MzNhOGMxNDVlZjY2MDRmNjMxID0gJCgnPGRpdiBpZD0iaHRtbF82OGIzNWJiMDE5YWE0NDMzYThjMTQ1ZWY2NjA0ZjYzMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SXNsaW5ndG9uIEF2ZW51ZSwgSHVtYmVyIFZhbGxleSBWaWxsYWdlLCBFdG9iaWNva2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzVhMDk1ZjhhMzJlYjRmMzFhYTBhYzZhYzllZDg5OTAxLnNldENvbnRlbnQoaHRtbF82OGIzNWJiMDE5YWE0NDMzYThjMTQ1ZWY2NjA0ZjYzMSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9lMDljNjlhZjEwYzM0ZTY3YjBlMzFmMzQxZjMyYjVhZC5iaW5kUG9wdXAocG9wdXBfNWEwOTVmOGEzMmViNGYzMWFhMGFjNmFjOWVkODk5MDEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDMzYTI0YzJmNzAyNGM1NThiM2UzMjNlNzhhM2IwNjEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTA5NDMyLC03OS41NTQ3MjQ0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82N2I3YjM5ODg1NWE0MmM1OTFjMzdiY2Y5MDFkMzRjZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8xN2NjNzNmNDMxMWE0NjViYjJmMzgzZTY2OGU0YzllNyA9ICQoJzxkaXYgaWQ9Imh0bWxfMTdjYzczZjQzMTFhNDY1YmIyZjM4M2U2NjhlNGM5ZTciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldlc3QgRGVhbmUgUGFyaywgUHJpbmNlc3MgR2FyZGVucywgTWFydGluIEdyb3ZlLCBJc2xpbmd0b24sIENsb3ZlcmRhbGUsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNjdiN2IzOTg4NTVhNDJjNTkxYzM3YmNmOTAxZDM0Y2Yuc2V0Q29udGVudChodG1sXzE3Y2M3M2Y0MzExYTQ2NWJiMmYzODNlNjY4ZTRjOWU3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzAzM2EyNGMyZjcwMjRjNTU4YjNlMzIzZTc4YTNiMDYxLmJpbmRQb3B1cChwb3B1cF82N2I3YjM5ODg1NWE0MmM1OTFjMzdiY2Y5MDFkMzRjZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kY2U0MjYxODcyNDU0YzlkYjIyYTgxYTVlOGRkZGViMSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0MzUxNTIsLTc5LjU3NzIwMDc5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI5Mzc3N2I1ZmVjZTQ3OThiNjMxOTEyMzZkNjA1NDEwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2YzZGZmZWUyNTc4ODQzNjliYzJkZjJmODI2ZmRkZjIxID0gJCgnPGRpdiBpZD0iaHRtbF9mM2RmZmVlMjU3ODg0MzY5YmMyZGYyZjgyNmZkZGYyMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RXJpbmdhdGUsIEJsb29yZGFsZSBHYXJkZW5zLCBPbGQgQnVybmhhbXRob3JwZSwgTWFya2xhbmQgV29vZCwgRXRvYmljb2tlPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yOTM3NzdiNWZlY2U0Nzk4YjYzMTkxMjM2ZDYwNTQxMC5zZXRDb250ZW50KGh0bWxfZjNkZmZlZTI1Nzg4NDM2OWJjMmRmMmY4MjZmZGRmMjEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZGNlNDI2MTg3MjQ1NGM5ZGIyMmE4MWE1ZThkZGRlYjEuYmluZFBvcHVwKHBvcHVwXzI5Mzc3N2I1ZmVjZTQ3OThiNjMxOTEyMzZkNjA1NDEwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2NiYzA2NTQyYmIzODQ1YTRhNmIyZDFmYTkyMTkwMmI0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzU2MzAzMywtNzkuNTY1OTYzMjk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZTM3ODE0MjY5YzNmNDkzMWE2ZmEyZTg3ZmY0OTRiOTIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjUwMzE2ZDk2MjVkNDk5OTg4OTJjOGJlZDdiYzNiNzIgPSAkKCc8ZGl2IGlkPSJodG1sX2Y1MDMxNmQ5NjI1ZDQ5OTk4ODkyYzhiZWQ3YmMzYjcyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IdW1iZXIgU3VtbWl0LCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lMzc4MTQyNjljM2Y0OTMxYTZmYTJlODdmZjQ5NGI5Mi5zZXRDb250ZW50KGh0bWxfZjUwMzE2ZDk2MjVkNDk5OTg4OTJjOGJlZDdiYzNiNzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfY2JjMDY1NDJiYjM4NDVhNGE2YjJkMWZhOTIxOTAyYjQuYmluZFBvcHVwKHBvcHVwX2UzNzgxNDI2OWMzZjQ5MzFhNmZhMmU4N2ZmNDk0YjkyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzgzMzI0YWMyOWRkZTQxNGM5Mzc5ZmU1ZjI2Y2JmODMwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzI0NzY1OSwtNzkuNTMyMjQyNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMjcxZjczNzczNzBjNGNmNjk3MjkxNmIyMDlkZWZmZjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZmFhYjI4NWVlMTFhNGViZTgzOWI1YjRjZmZkZmZlNjggPSAkKCc8ZGl2IGlkPSJodG1sX2ZhYWIyODVlZTExYTRlYmU4MzliNWI0Y2ZmZGZmZTY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IdW1iZXJsZWEsIEVtZXJ5LCBOb3J0aCBZb3JrPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yNzFmNzM3NzM3MGM0Y2Y2OTcyOTE2YjIwOWRlZmZmMC5zZXRDb250ZW50KGh0bWxfZmFhYjI4NWVlMTFhNGViZTgzOWI1YjRjZmZkZmZlNjgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfODMzMjRhYzI5ZGRlNDE0YzkzNzlmZTVmMjZjYmY4MzAuYmluZFBvcHVwKHBvcHVwXzI3MWY3Mzc3MzcwYzRjZjY5NzI5MTZiMjA5ZGVmZmYwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzVmMDVkZmI1MjU5MzQ0YTM4Mjc4OTIxYTQwZDdhODg3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzA2ODc2LC03OS41MTgxODg0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83NmY2Nzk2NjRjODc0MDE0OTAxYjQ2MzdlMWRjMThjMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hMjZkNWVkYTMzMTc0NzNkODdmMTM1NWVmZGM3MzQ0ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfYTI2ZDVlZGEzMzE3NDczZDg3ZjEzNTVlZmRjNzM0NGYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPldlc3RvbiwgWW9yazwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNzZmNjc5NjY0Yzg3NDAxNDkwMWI0NjM3ZTFkYzE4YzMuc2V0Q29udGVudChodG1sX2EyNmQ1ZWRhMzMxNzQ3M2Q4N2YxMzU1ZWZkYzczNDRmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzVmMDVkZmI1MjU5MzQ0YTM4Mjc4OTIxYTQwZDdhODg3LmJpbmRQb3B1cChwb3B1cF83NmY2Nzk2NjRjODc0MDE0OTAxYjQ2MzdlMWRjMThjMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jNjFkNTZjMTgxMmU0YjE0OWJjYzI1MjlkNGE0MDIxZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5NjMxOSwtNzkuNTMyMjQyNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjI4M2Y3NmU0NjNkNGJlYjhkNjRhZmJiZDcyYmViZDQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjQ3ZDU5ZjM5MjIxNDhkMzk2NjM2OTM2ODMyZTVmOGYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZjBkMzcwMjQ0NDAyNDc0ZmIzOGMyNDMwNzFjNjM1ZGQgPSAkKCc8ZGl2IGlkPSJodG1sX2YwZDM3MDI0NDQwMjQ3NGZiMzhjMjQzMDcxYzYzNWRkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5XZXN0bW91bnQsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjQ3ZDU5ZjM5MjIxNDhkMzk2NjM2OTM2ODMyZTVmOGYuc2V0Q29udGVudChodG1sX2YwZDM3MDI0NDQwMjQ3NGZiMzhjMjQzMDcxYzYzNWRkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2M2MWQ1NmMxODEyZTRiMTQ5YmNjMjUyOWQ0YTQwMjFkLmJpbmRQb3B1cChwb3B1cF9iNDdkNTlmMzkyMjE0OGQzOTY2MzY5MzY4MzJlNWY4Zik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80NjgwNjQxN2Q3Nzc0ZmMyOTJlMDY1NzE2MTFkMDg1YSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY4ODkwNTQsLTc5LjU1NDcyNDQwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzYyODNmNzZlNDYzZDRiZWI4ZDY0YWZiYmQ3MmJlYmQ0KTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzBhYTViZTQyYjg2MjQ4YmQ5MmUwMGIzZTNiNTYyMjJiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2YzZTdiZTc5ZWIyYjQ1YjA4MDdjNGEyY2U4N2U0ODI2ID0gJCgnPGRpdiBpZD0iaHRtbF9mM2U3YmU3OWViMmI0NWIwODA3YzRhMmNlODdlNDgyNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+S2luZ3N2aWV3IFZpbGxhZ2UsIFN0LiBQaGlsbGlwcywgTWFydGluIEdyb3ZlIEdhcmRlbnMsIFJpY2h2aWV3IEdhcmRlbnMsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMGFhNWJlNDJiODYyNDhiZDkyZTAwYjNlM2I1NjIyMmIuc2V0Q29udGVudChodG1sX2YzZTdiZTc5ZWIyYjQ1YjA4MDdjNGEyY2U4N2U0ODI2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzQ2ODA2NDE3ZDc3NzRmYzI5MmUwNjU3MTYxMWQwODVhLmJpbmRQb3B1cChwb3B1cF8wYWE1YmU0MmI4NjI0OGJkOTJlMDBiM2UzYjU2MjIyYik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iOTExZTAyNGI4ZTI0MWYyYTc5MTY0MzM4YzcyODIwZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjczOTQxNjM5OTk5OTk5NiwtNzkuNTg4NDM2OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jMzc0YzM4N2JiOWM0MTJjYTZjZGUzMDY5ZGE2YWZmOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81YmYyMjg3YjhmOTU0ODZkYTQ5NjhlYTllZTE0NTk5NCA9ICQoJzxkaXYgaWQ9Imh0bWxfNWJmMjI4N2I4Zjk1NDg2ZGE0OTY4ZWE5ZWUxNDU5OTQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNvdXRoIFN0ZWVsZXMsIFNpbHZlcnN0b25lLCBIdW1iZXJnYXRlLCBKYW1lc3Rvd24sIE1vdW50IE9saXZlLCBCZWF1bW9uZCBIZWlnaHRzLCBUaGlzdGxldG93biwgQWxiaW9uIEdhcmRlbnMsIEV0b2JpY29rZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzM3NGMzODdiYjljNDEyY2E2Y2RlMzA2OWRhNmFmZjkuc2V0Q29udGVudChodG1sXzViZjIyODdiOGY5NTQ4NmRhNDk2OGVhOWVlMTQ1OTk0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I5MTFlMDI0YjhlMjQxZjJhNzkxNjQzMzhjNzI4MjBkLmJpbmRQb3B1cChwb3B1cF9jMzc0YzM4N2JiOWM0MTJjYTZjZGUzMDY5ZGE2YWZmOSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zZGU1ZjcwMTEwYTY0NjViOGJhNGY5YmVkMThkNGUyNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcwNjc0ODI5OTk5OTk5NCwtNzkuNTk0MDU0NF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF82MjgzZjc2ZTQ2M2Q0YmViOGQ2NGFmYmJkNzJiZWJkNCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hODU2MjVkOGZjYmQ0ZGFiYjgxMWEzZTA0NjE2Y2U1ZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yNWI5ZTljNTViZTg0YWEzODVlODQ0N2I2Y2U2ZTFmMyA9ICQoJzxkaXYgaWQ9Imh0bWxfMjViOWU5YzU1YmU4NGFhMzg1ZTg0NDdiNmNlNmUxZjMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRod2VzdCwgV2VzdCBIdW1iZXIgLSBDbGFpcnZpbGxlLCBFdG9iaWNva2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2E4NTYyNWQ4ZmNiZDRkYWJiODExYTNlMDQ2MTZjZTVlLnNldENvbnRlbnQoaHRtbF8yNWI5ZTljNTViZTg0YWEzODVlODQ0N2I2Y2U2ZTFmMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zZGU1ZjcwMTEwYTY0NjViOGJhNGY5YmVkMThkNGUyNy5iaW5kUG9wdXAocG9wdXBfYTg1NjI1ZDhmY2JkNGRhYmI4MTFhM2UwNDYxNmNlNWUpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg== onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



#### Create a map of a part of Toronto City


```python
# Simplify the above map and segment and cluster only the boroughs that contain the word 'Toronto'
borough_toronto = toronto_df[toronto_df['Borough'].str.contains('Toronto')].reset_index(drop=True)
borough_toronto.head()
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
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M4E</td>
      <td>East Toronto</td>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M4K</td>
      <td>East Toronto</td>
      <td>The Danforth West, Riverdale</td>
      <td>43.679557</td>
      <td>-79.352188</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M4L</td>
      <td>East Toronto</td>
      <td>India Bazaar, The Beaches West</td>
      <td>43.668999</td>
      <td>-79.315572</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M4M</td>
      <td>East Toronto</td>
      <td>Studio District</td>
      <td>43.659526</td>
      <td>-79.340923</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M4N</td>
      <td>Central Toronto</td>
      <td>Lawrence Park</td>
      <td>43.728020</td>
      <td>-79.388790</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create a map for this region using latitude and longitude values
map_borough_toronto = folium.Map(location=[latitude, longitude], zoom_start=12)
for lat, lng, borough, neighborhood in zip(borough_toronto['Latitude'], borough_toronto['Longitude'], borough_toronto['Borough'], borough_toronto['Neighbourhood']):
    label = '{}, {}'.format(neighborhood, borough)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_borough_toronto)  

map_borough_toronto
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYScsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDMuNjUzNDgxNywtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMiwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMjhiY2I5ZjBlNmQyNDE4Yjg4NGNkYmM1NWFjYTkyM2YgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2NiMTk3OTAwYTMwNjQxZTViZThmZmZhNDg5YzAzYWVmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjc2MzU3Mzk5OTk5OTksLTc5LjI5MzAzMTJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfY2IyNmY5MmY5OGRlNGNmZTkxN2JjMGUwNjA1MTdlMWIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfOTIwN2MyNmUxYTM1NGEwOWIyNjRlY2Q5MjcwZjA4ZGMgPSAkKCc8ZGl2IGlkPSJodG1sXzkyMDdjMjZlMWEzNTRhMDliMjY0ZWNkOTI3MGYwOGRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgQmVhY2hlcywgRWFzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jYjI2ZjkyZjk4ZGU0Y2ZlOTE3YmMwZTA2MDUxN2UxYi5zZXRDb250ZW50KGh0bWxfOTIwN2MyNmUxYTM1NGEwOWIyNjRlY2Q5MjcwZjA4ZGMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfY2IxOTc5MDBhMzA2NDFlNWJlOGZmZmE0ODljMDNhZWYuYmluZFBvcHVwKHBvcHVwX2NiMjZmOTJmOThkZTRjZmU5MTdiYzBlMDYwNTE3ZTFiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzliNWEyNjI4YzhlOTRjOGNiY2U1Nzc0NDllODg0ZDcyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjc5NTU3MSwtNzkuMzUyMTg4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzhhYTk3NzFkM2Y1YjRjNzI4MDQ3N2M2ZGVkNDc2Y2Y4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY5OGJmNmI2ODI1MTQwZTM4ODc5NDYxYmIxYThlYjE2ID0gJCgnPGRpdiBpZD0iaHRtbF82OThiZjZiNjgyNTE0MGUzODg3OTQ2MWJiMWE4ZWIxNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIERhbmZvcnRoIFdlc3QsIFJpdmVyZGFsZSwgRWFzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84YWE5NzcxZDNmNWI0YzcyODA0NzdjNmRlZDQ3NmNmOC5zZXRDb250ZW50KGh0bWxfNjk4YmY2YjY4MjUxNDBlMzg4Nzk0NjFiYjFhOGViMTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOWI1YTI2MjhjOGU5NGM4Y2JjZTU3NzQ0OWU4ODRkNzIuYmluZFBvcHVwKHBvcHVwXzhhYTk3NzFkM2Y1YjRjNzI4MDQ3N2M2ZGVkNDc2Y2Y4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzc1N2RhMjc5NDNjYzRlZmFiN2YyNzZmNzQ1ZDhmMjljID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY4OTk4NSwtNzkuMzE1NTcxNTk5OTk5OThdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNzZlZGNlNmFkNTliNGViMDlmNzcyNmY1NDQ1YzM1ZTEgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNmM1ZmI0MmVlNjViNDQ3MmFhNDQ1YTY4YTFkMjA2ZmIgPSAkKCc8ZGl2IGlkPSJodG1sXzZjNWZiNDJlZTY1YjQ0NzJhYTQ0NWE2OGExZDIwNmZiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5JbmRpYSBCYXphYXIsIFRoZSBCZWFjaGVzIFdlc3QsIEVhc3QgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNzZlZGNlNmFkNTliNGViMDlmNzcyNmY1NDQ1YzM1ZTEuc2V0Q29udGVudChodG1sXzZjNWZiNDJlZTY1YjQ0NzJhYTQ0NWE2OGExZDIwNmZiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzc1N2RhMjc5NDNjYzRlZmFiN2YyNzZmNzQ1ZDhmMjljLmJpbmRQb3B1cChwb3B1cF83NmVkY2U2YWQ1OWI0ZWIwOWY3NzI2ZjU0NDVjMzVlMSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83YmY2YWQ3YTI4ZDA0ZTQwODgwZjYzNDlhZTBhYTI5OSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1OTUyNTUsLTc5LjM0MDkyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9kZDAyMzI4OTkyYzM0ZmU2ODkxNTBlOTFhYWU4ZWVmMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82MjVjYzhhZDUwYmQ0MGM2YmM5Mjg3ODM4NjY1NGY2YiA9ICQoJzxkaXYgaWQ9Imh0bWxfNjI1Y2M4YWQ1MGJkNDBjNmJjOTI4NzgzODY2NTRmNmIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0dWRpbyBEaXN0cmljdCwgRWFzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kZDAyMzI4OTkyYzM0ZmU2ODkxNTBlOTFhYWU4ZWVmMy5zZXRDb250ZW50KGh0bWxfNjI1Y2M4YWQ1MGJkNDBjNmJjOTI4NzgzODY2NTRmNmIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2JmNmFkN2EyOGQwNGU0MDg4MGY2MzQ5YWUwYWEyOTkuYmluZFBvcHVwKHBvcHVwX2RkMDIzMjg5OTJjMzRmZTY4OTE1MGU5MWFhZThlZWYzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzJkMTFlNDA1MjVmOTRlN2FiYTcyZjZmYjBlMTg5ODU2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzI4MDIwNSwtNzkuMzg4NzkwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81ZWNiNDNiMGE3YWI0YjY4OTljZWMyMTJiNDgzM2ViNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82MjZmZGNiNTc1YmY0NDc1OTdmMWM0YTdmZjM1MzlmMSA9ICQoJzxkaXYgaWQ9Imh0bWxfNjI2ZmRjYjU3NWJmNDQ3NTk3ZjFjNGE3ZmYzNTM5ZjEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhd3JlbmNlIFBhcmssIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWVjYjQzYjBhN2FiNGI2ODk5Y2VjMjEyYjQ4MzNlYjcuc2V0Q29udGVudChodG1sXzYyNmZkY2I1NzViZjQ0NzU5N2YxYzRhN2ZmMzUzOWYxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzJkMTFlNDA1MjVmOTRlN2FiYTcyZjZmYjBlMTg5ODU2LmJpbmRQb3B1cChwb3B1cF81ZWNiNDNiMGE3YWI0YjY4OTljZWMyMTJiNDgzM2ViNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lMDZlNTgzNzk4NjQ0YzM5OGUwNjBhYmVkNWMwNmFiMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcxMjc1MTEsLTc5LjM5MDE5NzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmQyNzE5NjYyOWVlNDNhZjk3NzhjZGFjODM2MmQ4YmQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYTIyMzc0YjRkMzk2NDEyMmFiNjRhYmFiMWYwZGQ0Y2MgPSAkKCc8ZGl2IGlkPSJodG1sX2EyMjM3NGI0ZDM5NjQxMjJhYjY0YWJhYjFmMGRkNGNjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5EYXZpc3ZpbGxlIE5vcnRoLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2JkMjcxOTY2MjllZTQzYWY5Nzc4Y2RhYzgzNjJkOGJkLnNldENvbnRlbnQoaHRtbF9hMjIzNzRiNGQzOTY0MTIyYWI2NGFiYWIxZjBkZDRjYyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9lMDZlNTgzNzk4NjQ0YzM5OGUwNjBhYmVkNWMwNmFiMi5iaW5kUG9wdXAocG9wdXBfYmQyNzE5NjYyOWVlNDNhZjk3NzhjZGFjODM2MmQ4YmQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDRhZDgxZGNhNmMyNGQzZWFmOWMxNmQyMTk3YTFkMTYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTUzODM0LC03OS40MDU2Nzg0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xNGE1Y2RiYTEyMjQ0ZmMxODNhYmIyOTc3ZTNkYjcyMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82YTM1NzFhYTgxNjM0ZjBmYTFjYmRjYTkwOGU4MzU1ZSA9ICQoJzxkaXYgaWQ9Imh0bWxfNmEzNTcxYWE4MTYzNGYwZmExY2JkY2E5MDhlODM1NWUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIFRvcm9udG8gV2VzdCwgIExhd3JlbmNlIFBhcmssIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTRhNWNkYmExMjI0NGZjMTgzYWJiMjk3N2UzZGI3MjAuc2V0Q29udGVudChodG1sXzZhMzU3MWFhODE2MzRmMGZhMWNiZGNhOTA4ZTgzNTVlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzA0YWQ4MWRjYTZjMjRkM2VhZjljMTZkMjE5N2ExZDE2LmJpbmRQb3B1cChwb3B1cF8xNGE1Y2RiYTEyMjQ0ZmMxODNhYmIyOTc3ZTNkYjcyMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83ZmRkYzFmNzlmMmY0Y2U2OTcxMDJkNTFhZWJjNzk2MiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcwNDMyNDQsLTc5LjM4ODc5MDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTIyNDgzZDJmOWNkNGQxNzlkYzg1MGE1ODNlMzdlZTIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNDNmYmVlZjEzZDZkNDQwZGFkY2NkNmQ2NjRmMTU5NzQgPSAkKCc8ZGl2IGlkPSJodG1sXzQzZmJlZWYxM2Q2ZDQ0MGRhZGNjZDZkNjY0ZjE1OTc0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5EYXZpc3ZpbGxlLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzUyMjQ4M2QyZjljZDRkMTc5ZGM4NTBhNTgzZTM3ZWUyLnNldENvbnRlbnQoaHRtbF80M2ZiZWVmMTNkNmQ0NDBkYWRjY2Q2ZDY2NGYxNTk3NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83ZmRkYzFmNzlmMmY0Y2U2OTcxMDJkNTFhZWJjNzk2Mi5iaW5kUG9wdXAocG9wdXBfNTIyNDgzZDJmOWNkNGQxNzlkYzg1MGE1ODNlMzdlZTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjhmZDVmNDllZjJmNGM2ZGI5YTIwMTFmMGRjMDE5YzMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42ODk1NzQzLC03OS4zODMxNTk5MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82OWNiODEwMmNkNTE0NDFhYTJlZjM5ODk2MGFjMTdjOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kZDEwN2IwNjI0MWI0NTdjYjVmZjZmYjk3YzliNjEwNSA9ICQoJzxkaXYgaWQ9Imh0bWxfZGQxMDdiMDYyNDFiNDU3Y2I1ZmY2ZmI5N2M5YjYxMDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1vb3JlIFBhcmssIFN1bW1lcmhpbGwgRWFzdCwgQ2VudHJhbCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82OWNiODEwMmNkNTE0NDFhYTJlZjM5ODk2MGFjMTdjOC5zZXRDb250ZW50KGh0bWxfZGQxMDdiMDYyNDFiNDU3Y2I1ZmY2ZmI5N2M5YjYxMDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjhmZDVmNDllZjJmNGM2ZGI5YTIwMTFmMGRjMDE5YzMuYmluZFBvcHVwKHBvcHVwXzY5Y2I4MTAyY2Q1MTQ0MWFhMmVmMzk4OTYwYWMxN2M4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNkZGNiNWI4NDJlYTRhYTA5OTEzMTE2MGRmNzlhMzIyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjg2NDEyMjk5OTk5OTksLTc5LjQwMDA0OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNmQ5ODYzZjk3ZjU2NGZhYjg2ZTRhOTEyY2I2OTEyNzcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYTBlOWI4NTQ3NDFkNDlkYmE5ODBmODNkYjYxMTU1ZjggPSAkKCc8ZGl2IGlkPSJodG1sX2EwZTliODU0NzQxZDQ5ZGJhOTgwZjgzZGI2MTE1NWY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdW1tZXJoaWxsIFdlc3QsIFJhdGhuZWxseSwgU291dGggSGlsbCwgRm9yZXN0IEhpbGwgU0UsIERlZXIgUGFyaywgQ2VudHJhbCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82ZDk4NjNmOTdmNTY0ZmFiODZlNGE5MTJjYjY5MTI3Ny5zZXRDb250ZW50KGh0bWxfYTBlOWI4NTQ3NDFkNDlkYmE5ODBmODNkYjYxMTU1ZjgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2RkY2I1Yjg0MmVhNGFhMDk5MTMxMTYwZGY3OWEzMjIuYmluZFBvcHVwKHBvcHVwXzZkOTg2M2Y5N2Y1NjRmYWI4NmU0YTkxMmNiNjkxMjc3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI3MGUzYTY1ZWZiNzQ0ZjhiZGE1NjgwMjdjOGU5ODVlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjc5NTYyNiwtNzkuMzc3NTI5NDAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYmRlY2JlMzZjMTk1NGI5NTk4YTVlZTJiNTI0NzNlYzUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNmRmZWU4NmU2MjYzNGIxZmFkZDFlNzE4MjQ4NjgyNjggPSAkKCc8ZGl2IGlkPSJodG1sXzZkZmVlODZlNjI2MzRiMWZhZGQxZTcxODI0ODY4MjY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NlZGFsZSwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYmRlY2JlMzZjMTk1NGI5NTk4YTVlZTJiNTI0NzNlYzUuc2V0Q29udGVudChodG1sXzZkZmVlODZlNjI2MzRiMWZhZGQxZTcxODI0ODY4MjY4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzI3MGUzYTY1ZWZiNzQ0ZjhiZGE1NjgwMjdjOGU5ODVlLmJpbmRQb3B1cChwb3B1cF9iZGVjYmUzNmMxOTU0Yjk1OThhNWVlMmI1MjQ3M2VjNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wNDUzZDE1NzBjYWQ0MWYyODc2ODNjYzI3MjQwNTdiMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2Nzk2NywtNzkuMzY3Njc1M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yY2UyN2MxNTAxY2I0YWU0OGM3M2UyYWNjYjZiMTczOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80ZWE3YWI2YmRiODE0MmYzYmRjMjgwN2E4YjNlM2RlZCA9ICQoJzxkaXYgaWQ9Imh0bWxfNGVhN2FiNmJkYjgxNDJmM2JkYzI4MDdhOGIzZTNkZWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0LiBKYW1lcyBUb3duLCBDYWJiYWdldG93biwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMmNlMjdjMTUwMWNiNGFlNDhjNzNlMmFjY2I2YjE3Mzguc2V0Q29udGVudChodG1sXzRlYTdhYjZiZGI4MTQyZjNiZGMyODA3YThiM2UzZGVkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzA0NTNkMTU3MGNhZDQxZjI4NzY4M2NjMjcyNDA1N2IzLmJpbmRQb3B1cChwb3B1cF8yY2UyN2MxNTAxY2I0YWU0OGM3M2UyYWNjYjZiMTczOCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iMGJmYmI0OGEwMjg0NjRkYWIwZTI3YzQzZWI2NGU0NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2NTg1OTksLTc5LjM4MzE1OTkwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzAxNWVmZWE4OWI3MDQwZjhhNjEwMzJhNDJjYmQyNjExID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzU3MWJhNDg1MGVjYjQ2NGQ4OWI4Y2FmZmU3MWQ4NzkyID0gJCgnPGRpdiBpZD0iaHRtbF81NzFiYTQ4NTBlY2I0NjRkODliOGNhZmZlNzFkODc5MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2h1cmNoIGFuZCBXZWxsZXNsZXksIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAxNWVmZWE4OWI3MDQwZjhhNjEwMzJhNDJjYmQyNjExLnNldENvbnRlbnQoaHRtbF81NzFiYTQ4NTBlY2I0NjRkODliOGNhZmZlNzFkODc5Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iMGJmYmI0OGEwMjg0NjRkYWIwZTI3YzQzZWI2NGU0NC5iaW5kUG9wdXAocG9wdXBfMDE1ZWZlYTg5YjcwNDBmOGE2MTAzMmE0MmNiZDI2MTEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzdlYjYzNTM1NTI2NDJjODg0MWUxNmVmZTRhMzk1OTEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTQyNTk5LC03OS4zNjA2MzU5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzZmZTYyYzZjNTZjNjQxNzU4Y2I5NzE3YTcxMzQxYjljID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzAxMDJlOWUyOWY0NDQ4ODhhNjJiYmYyZDY4YjhlOWMxID0gJCgnPGRpdiBpZD0iaHRtbF8wMTAyZTllMjlmNDQ0ODg4YTYyYmJmMmQ2OGI4ZTljMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UmVnZW50IFBhcmssIEhhcmJvdXJmcm9udCwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNmZlNjJjNmM1NmM2NDE3NThjYjk3MTdhNzEzNDFiOWMuc2V0Q29udGVudChodG1sXzAxMDJlOWUyOWY0NDQ4ODhhNjJiYmYyZDY4YjhlOWMxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzM3ZWI2MzUzNTUyNjQyYzg4NDFlMTZlZmU0YTM5NTkxLmJpbmRQb3B1cChwb3B1cF82ZmU2MmM2YzU2YzY0MTc1OGNiOTcxN2E3MTM0MWI5Yyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84YWRlMzNhYzIwNTQ0MzU5OWRiNDVkZDQ2Y2JhMGZlOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1NzE2MTgsLTc5LjM3ODkzNzA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U1NzY5NmU4NzFiMDQ2MTFiMzhlYWU3ZjcwOWNkNGM1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzFmODRmNjg0OTQ4MTQzNTBhNDBjNDFkOWI5ZDZmYTYzID0gJCgnPGRpdiBpZD0iaHRtbF8xZjg0ZjY4NDk0ODE0MzUwYTQwYzQxZDliOWQ2ZmE2MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2FyZGVuIERpc3RyaWN0LCBSeWVyc29uLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lNTc2OTZlODcxYjA0NjExYjM4ZWFlN2Y3MDljZDRjNS5zZXRDb250ZW50KGh0bWxfMWY4NGY2ODQ5NDgxNDM1MGE0MGM0MWQ5YjlkNmZhNjMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOGFkZTMzYWMyMDU0NDM1OTlkYjQ1ZGQ0NmNiYTBmZTguYmluZFBvcHVwKHBvcHVwX2U1NzY5NmU4NzFiMDQ2MTFiMzhlYWU3ZjcwOWNkNGM1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzljZDcwMWI4ZTM4NDQxOWQ4YTJlNzIyMzFmYTNhZTRlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUxNDkzOSwtNzkuMzc1NDE3OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84ZGQ0Mzk0ZDEyY2U0NGE4OWM3NjU5MTFmODUxMGMwYiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82YjBiMjg0OWNiNjQ0NjFjOTVjMzJiNjQ5NmY0MDA0NSA9ICQoJzxkaXYgaWQ9Imh0bWxfNmIwYjI4NDljYjY0NDYxYzk1YzMyYjY0OTZmNDAwNDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0LiBKYW1lcyBUb3duLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84ZGQ0Mzk0ZDEyY2U0NGE4OWM3NjU5MTFmODUxMGMwYi5zZXRDb250ZW50KGh0bWxfNmIwYjI4NDljYjY0NDYxYzk1YzMyYjY0OTZmNDAwNDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOWNkNzAxYjhlMzg0NDE5ZDhhMmU3MjIzMWZhM2FlNGUuYmluZFBvcHVwKHBvcHVwXzhkZDQzOTRkMTJjZTQ0YTg5Yzc2NTkxMWY4NTEwYzBiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzcxMWNhYWE1ODIwZTRhMDc5NmZjNjBkYzA4N2M4MDkyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ0NzcwNzk5OTk5OTk2LC03OS4zNzMzMDY0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzc4YWIzNDBlYThlNTRiNTk5ZTYzZjc3MWQ1M2Q1NDllID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ3YTVjYzBkMWQyNTQ5YTliMTI2YTEwYTg4MTRkYTE5ID0gJCgnPGRpdiBpZD0iaHRtbF80N2E1Y2MwZDFkMjU0OWE5YjEyNmExMGE4ODE0ZGExOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmVyY3p5IFBhcmssIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzc4YWIzNDBlYThlNTRiNTk5ZTYzZjc3MWQ1M2Q1NDllLnNldENvbnRlbnQoaHRtbF80N2E1Y2MwZDFkMjU0OWE5YjEyNmExMGE4ODE0ZGExOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83MTFjYWFhNTgyMGU0YTA3OTZmYzYwZGMwODdjODA5Mi5iaW5kUG9wdXAocG9wdXBfNzhhYjM0MGVhOGU1NGI1OTllNjNmNzcxZDUzZDU0OWUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDgyYjI0NWE1OTc1NDBlYjk0OGY2ZTRhNGNkYzY3MDkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTc5NTI0LC03OS4zODczODI2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzRiMTk3ZmI1MWZjMjQzZmZhZTQxOWFlMTI0MDFjYzYyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2QwMWI4YTUyMDg5NzQ4NmI4MGE4YmVmYjMzMTk4YWY1ID0gJCgnPGRpdiBpZD0iaHRtbF9kMDFiOGE1MjA4OTc0ODZiODBhOGJlZmIzMzE5OGFmNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2VudHJhbCBCYXkgU3RyZWV0LCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80YjE5N2ZiNTFmYzI0M2ZmYWU0MTlhZTEyNDAxY2M2Mi5zZXRDb250ZW50KGh0bWxfZDAxYjhhNTIwODk3NDg2YjgwYThiZWZiMzMxOThhZjUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDgyYjI0NWE1OTc1NDBlYjk0OGY2ZTRhNGNkYzY3MDkuYmluZFBvcHVwKHBvcHVwXzRiMTk3ZmI1MWZjMjQzZmZhZTQxOWFlMTI0MDFjYzYyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y5NGMwNDM1OGI2ODQ0MzdhY2QwNWExYzk2Y2YwN2FmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUwNTcxMjAwMDAwMDEsLTc5LjM4NDU2NzVdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWQ1NzViMTQ2ZTRiNGY1OTlkYTRhZGY3MzEwODUwODggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGYzODZlZTgzOWJiNDAwOTk0ZmY3MjM2OTM5M2MwMzYgPSAkKCc8ZGl2IGlkPSJodG1sXzRmMzg2ZWU4MzliYjQwMDk5NGZmNzIzNjkzOTNjMDM2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SaWNobW9uZCwgQWRlbGFpZGUsIEtpbmcsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzVkNTc1YjE0NmU0YjRmNTk5ZGE0YWRmNzMxMDg1MDg4LnNldENvbnRlbnQoaHRtbF80ZjM4NmVlODM5YmI0MDA5OTRmZjcyMzY5MzkzYzAzNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mOTRjMDQzNThiNjg0NDM3YWNkMDVhMWM5NmNmMDdhZi5iaW5kUG9wdXAocG9wdXBfNWQ1NzViMTQ2ZTRiNGY1OTlkYTRhZGY3MzEwODUwODgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTI1MTkxNDk3YWM0NDUzZTk4NjE0OWU1MDhiNTI1MzggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDA4MTU3LC03OS4zODE3NTIyOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81OTQ5NDI5MDYxYTQ0NzUxOTc5OWE3YzgwZmE3M2ZjYiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80YmMwZTEzOGVkMTk0OTMwYTczMjExYmMxMTk0NGRlMyA9ICQoJzxkaXYgaWQ9Imh0bWxfNGJjMGUxMzhlZDE5NDkzMGE3MzIxMWJjMTE5NDRkZTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhhcmJvdXJmcm9udCBFYXN0LCBVbmlvbiBTdGF0aW9uLCBUb3JvbnRvIElzbGFuZHMsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzU5NDk0MjkwNjFhNDQ3NTE5Nzk5YTdjODBmYTczZmNiLnNldENvbnRlbnQoaHRtbF80YmMwZTEzOGVkMTk0OTMwYTczMjExYmMxMTk0NGRlMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85MjUxOTE0OTdhYzQ0NTNlOTg2MTQ5ZTUwOGI1MjUzOC5iaW5kUG9wdXAocG9wdXBfNTk0OTQyOTA2MWE0NDc1MTk3OTlhN2M4MGZhNzNmY2IpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZjM5Y2I1ZmIyNjcxNGZiMzk2YzI1MmU5Njg5ZWI1NmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDcxNzY4LC03OS4zODE1NzY0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jNjRhMzMwYjAyMTQ0NTlkYTI0OTUwMjM0NTY3OGRkOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kNzliY2JmM2Y1YTY0ODgwODYwNGM0MjRjMWNjYTMzMyA9ICQoJzxkaXYgaWQ9Imh0bWxfZDc5YmNiZjNmNWE2NDg4MDg2MDRjNDI0YzFjY2EzMzMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRvcm9udG8gRG9taW5pb24gQ2VudHJlLCBEZXNpZ24gRXhjaGFuZ2UsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2M2NGEzMzBiMDIxNDQ1OWRhMjQ5NTAyMzQ1Njc4ZGQ4LnNldENvbnRlbnQoaHRtbF9kNzliY2JmM2Y1YTY0ODgwODYwNGM0MjRjMWNjYTMzMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mMzljYjVmYjI2NzE0ZmIzOTZjMjUyZTk2ODllYjU2Yi5iaW5kUG9wdXAocG9wdXBfYzY0YTMzMGIwMjE0NDU5ZGEyNDk1MDIzNDU2NzhkZDgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWUzY2ZlYmM1MGI0NDM2ZmFjMjA4ZWVjZGVmZjM2ZTggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDgxOTg1LC03OS4zNzk4MTY5MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85M2U0NGExZDZkMzc0NjcyOWFhMmUyMTExNTAwNWExYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MDkwNGQzOGI5Yzg0OTdmYTFiZTBjM2Y5MTM4M2UzMSA9ICQoJzxkaXYgaWQ9Imh0bWxfNzA5MDRkMzhiOWM4NDk3ZmExYmUwYzNmOTEzODNlMzEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNvbW1lcmNlIENvdXJ0LCBWaWN0b3JpYSBIb3RlbCwgRG93bnRvd24gVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTNlNDRhMWQ2ZDM3NDY3MjlhYTJlMjExMTUwMDVhMWMuc2V0Q29udGVudChodG1sXzcwOTA0ZDM4YjljODQ5N2ZhMWJlMGMzZjkxMzgzZTMxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzFlM2NmZWJjNTBiNDQzNmZhYzIwOGVlY2RlZmYzNmU4LmJpbmRQb3B1cChwb3B1cF85M2U0NGExZDZkMzc0NjcyOWFhMmUyMTExNTAwNWExYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kZWY3NTdkOTU1YjE0Nzc1OTcxZDVkMmMxMTRkYWIwNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcxMTY5NDgsLTc5LjQxNjkzNTU5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzU5NThhYjVkNWViNzQxMjZiYzRiMDY5ZGViZjViZTNlID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2RiYjcyZTYzYTg3ODRjNzQ5ZDcxYzEwYTgxZWRjMzgzID0gJCgnPGRpdiBpZD0iaHRtbF9kYmI3MmU2M2E4Nzg0Yzc0OWQ3MWMxMGE4MWVkYzM4MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um9zZWxhd24sIENlbnRyYWwgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTk1OGFiNWQ1ZWI3NDEyNmJjNGIwNjlkZWJmNWJlM2Uuc2V0Q29udGVudChodG1sX2RiYjcyZTYzYTg3ODRjNzQ5ZDcxYzEwYTgxZWRjMzgzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2RlZjc1N2Q5NTViMTQ3NzU5NzFkNWQyYzExNGRhYjA0LmJpbmRQb3B1cChwb3B1cF81OTU4YWI1ZDVlYjc0MTI2YmM0YjA2OWRlYmY1YmUzZSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yNzcyYjhhNzJlYzM0ZGYzYTQ2YzUyYjcwMzQyZGQ4ZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5Njk0NzYsLTc5LjQxMTMwNzIwMDAwMDAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI1NjNmNDA0MjM0NjQ0ZWE4ZWMyYjhjNmE0YjlkMzkwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQwNjIxOTVkNDhiZDRmOGNhMzhkNTAyODI0MWRkZGY2ID0gJCgnPGRpdiBpZD0iaHRtbF80MDYyMTk1ZDQ4YmQ0ZjhjYTM4ZDUwMjgyNDFkZGRmNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rm9yZXN0IEhpbGwgTm9ydGggJmFtcDsgV2VzdCwgRm9yZXN0IEhpbGwgUm9hZCBQYXJrLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzI1NjNmNDA0MjM0NjQ0ZWE4ZWMyYjhjNmE0YjlkMzkwLnNldENvbnRlbnQoaHRtbF80MDYyMTk1ZDQ4YmQ0ZjhjYTM4ZDUwMjgyNDFkZGRmNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8yNzcyYjhhNzJlYzM0ZGYzYTQ2YzUyYjcwMzQyZGQ4ZS5iaW5kUG9wdXAocG9wdXBfMjU2M2Y0MDQyMzQ2NDRlYThlYzJiOGM2YTRiOWQzOTApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzJjY2M5NTFmNTdiNGNhN2FlNGIyOWMxMzZlMTZhYmYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NzI3MDk3LC03OS40MDU2Nzg0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83MjUxOTc3NDk4YjU0YWFiYjU0OWNhOWFkM2QyODE1ZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84MmZkYWMxNWY1NTA0Y2U0ODhmZDA3M2EyMGEwODY0MiA9ICQoJzxkaXYgaWQ9Imh0bWxfODJmZGFjMTVmNTUwNGNlNDg4ZmQwNzNhMjBhMDg2NDIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBBbm5leCwgTm9ydGggTWlkdG93biwgWW9ya3ZpbGxlLCBDZW50cmFsIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzcyNTE5Nzc0OThiNTRhYWJiNTQ5Y2E5YWQzZDI4MTVmLnNldENvbnRlbnQoaHRtbF84MmZkYWMxNWY1NTA0Y2U0ODhmZDA3M2EyMGEwODY0Mik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9jMmNjYzk1MWY1N2I0Y2E3YWU0YjI5YzEzNmUxNmFiZi5iaW5kUG9wdXAocG9wdXBfNzI1MTk3NzQ5OGI1NGFhYmI1NDljYTlhZDNkMjgxNWYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjA5ZmFkMmRlYTM4NDZlNThjNGExNGE3OGUwMWVlODQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjI2OTU2LC03OS40MDAwNDkzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzI1OGFlZTdhY2ViMTQ5MjFhNTBmMDUwOGY5MWExOGQyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2E3MzQyZWI5YjcxMDQ4OWI4MjhiZTY5M2M5Yjg3ZTcxID0gJCgnPGRpdiBpZD0iaHRtbF9hNzM0MmViOWI3MTA0ODliODI4YmU2OTNjOWI4N2U3MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VW5pdmVyc2l0eSBvZiBUb3JvbnRvLCBIYXJib3JkLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yNThhZWU3YWNlYjE0OTIxYTUwZjA1MDhmOTFhMThkMi5zZXRDb250ZW50KGh0bWxfYTczNDJlYjliNzEwNDg5YjgyOGJlNjkzYzliODdlNzEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjA5ZmFkMmRlYTM4NDZlNThjNGExNGE3OGUwMWVlODQuYmluZFBvcHVwKHBvcHVwXzI1OGFlZTdhY2ViMTQ5MjFhNTBmMDUwOGY5MWExOGQyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzUwZGQyNjVlZDlhYTRlYzM5YTYwYTEwMDIyNTQwMjQwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUzMjA1NywtNzkuNDAwMDQ5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83OGUyZGNjNzRjZmE0OTY0YjdiMTRkYjc5NWY3MTgyMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85ZmMzYmE2OTEyOWE0MDVmYWQxZjVkOGQwNzIzODFmYSA9ICQoJzxkaXYgaWQ9Imh0bWxfOWZjM2JhNjkxMjlhNDA1ZmFkMWY1ZDhkMDcyMzgxZmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktlbnNpbmd0b24gTWFya2V0LCBDaGluYXRvd24sIEdyYW5nZSBQYXJrLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83OGUyZGNjNzRjZmE0OTY0YjdiMTRkYjc5NWY3MTgyMi5zZXRDb250ZW50KGh0bWxfOWZjM2JhNjkxMjlhNDA1ZmFkMWY1ZDhkMDcyMzgxZmEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNTBkZDI2NWVkOWFhNGVjMzlhNjBhMTAwMjI1NDAyNDAuYmluZFBvcHVwKHBvcHVwXzc4ZTJkY2M3NGNmYTQ5NjRiN2IxNGRiNzk1ZjcxODIyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzkxMTljY2NiNDczMjRmNWQ5NjYwZDBiODdhODM1NjAxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjI4OTQ2NywtNzkuMzk0NDE5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83NTJiMDNiZDg5OTg0YmJjOWYyYWEzNTM5MDcxZTg2YSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yNWZjMDllMzM1N2M0YzUyYTBhMTAxYTQ4MTZiMzJmMyA9ICQoJzxkaXYgaWQ9Imh0bWxfMjVmYzA5ZTMzNTdjNGM1MmEwYTEwMWE0ODE2YjMyZjMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNOIFRvd2VyLCBLaW5nIGFuZCBTcGFkaW5hLCBSYWlsd2F5IExhbmRzLCBIYXJib3VyZnJvbnQgV2VzdCwgQmF0aHVyc3QgUXVheSwgU291dGggTmlhZ2FyYSwgSXNsYW5kIGFpcnBvcnQsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzc1MmIwM2JkODk5ODRiYmM5ZjJhYTM1MzkwNzFlODZhLnNldENvbnRlbnQoaHRtbF8yNWZjMDllMzM1N2M0YzUyYTBhMTAxYTQ4MTZiMzJmMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85MTE5Y2NjYjQ3MzI0ZjVkOTY2MGQwYjg3YTgzNTYwMS5iaW5kUG9wdXAocG9wdXBfNzUyYjAzYmQ4OTk4NGJiYzlmMmFhMzUzOTA3MWU4NmEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOWEyNjAyZWVlNGM3NDQwYWIyMmE1ZmQzYTBjNzAxMzUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDY0MzUyLC03OS4zNzQ4NDU5OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85YzZkN2ZhYmMzNWQ0OGE4YmFlM2UzMzBkMGYyNWVlNCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lMTZlN2JjMzBjZmU0YzEwYTU4M2I1ZGU2OGI5YTI3MiA9ICQoJzxkaXYgaWQ9Imh0bWxfZTE2ZTdiYzMwY2ZlNGMxMGE1ODNiNWRlNjhiOWEyNzIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0biBBIFBPIEJveGVzLCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85YzZkN2ZhYmMzNWQ0OGE4YmFlM2UzMzBkMGYyNWVlNC5zZXRDb250ZW50KGh0bWxfZTE2ZTdiYzMwY2ZlNGMxMGE1ODNiNWRlNjhiOWEyNzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOWEyNjAyZWVlNGM3NDQwYWIyMmE1ZmQzYTBjNzAxMzUuYmluZFBvcHVwKHBvcHVwXzljNmQ3ZmFiYzM1ZDQ4YThiYWUzZTMzMGQwZjI1ZWU0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQyOGEzYmRkZjU3ODRkNTg4ODRkMmRiYWQ3NmU1NjFhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4NDI5MiwtNzkuMzgyMjgwMl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85N2NkYTUzYmJjNWI0ZmFjYjYxMmU0YWI5MDIwODgwMSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jMmI5ZGY0YzM2Mzc0Njc2YWFhZmUwYTA3ODRlYzY5YyA9ICQoJzxkaXYgaWQ9Imh0bWxfYzJiOWRmNGMzNjM3NDY3NmFhYWZlMGEwNzg0ZWM2OWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZpcnN0IENhbmFkaWFuIFBsYWNlLCBVbmRlcmdyb3VuZCBjaXR5LCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85N2NkYTUzYmJjNWI0ZmFjYjYxMmU0YWI5MDIwODgwMS5zZXRDb250ZW50KGh0bWxfYzJiOWRmNGMzNjM3NDY3NmFhYWZlMGEwNzg0ZWM2OWMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDI4YTNiZGRmNTc4NGQ1ODg4NGQyZGJhZDc2ZTU2MWEuYmluZFBvcHVwKHBvcHVwXzk3Y2RhNTNiYmM1YjRmYWNiNjEyZTRhYjkwMjA4ODAxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2QxNGJmM2NmNzViMjRlNjM5NDdlMzBhOTBiYTljMWVhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY5NTQyLC03OS40MjI1NjM3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzczMDg3NGUyNThjZTQ4MDBiNzk4ODUyZmZkZGYxNDFkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzViOTFhMmJhYjI0MDRkN2RhN2I3NDU5MjljYTllZGI2ID0gJCgnPGRpdiBpZD0iaHRtbF81YjkxYTJiYWIyNDA0ZDdkYTdiNzQ1OTI5Y2E5ZWRiNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hyaXN0aWUsIERvd250b3duIFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzczMDg3NGUyNThjZTQ4MDBiNzk4ODUyZmZkZGYxNDFkLnNldENvbnRlbnQoaHRtbF81YjkxYTJiYWIyNDA0ZDdkYTdiNzQ1OTI5Y2E5ZWRiNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kMTRiZjNjZjc1YjI0ZTYzOTQ3ZTMwYTkwYmE5YzFlYS5iaW5kUG9wdXAocG9wdXBfNzMwODc0ZTI1OGNlNDgwMGI3OTg4NTJmZmRkZjE0MWQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMmUxNTY4ZTc4MWExNDZhYmE3Njg2Y2JjM2VjNDMwYTMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjkwMDUxMDAwMDAwMSwtNzkuNDQyMjU5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hNGY4M2ZhMTFlMWU0NzAzYmZmMzkyY2M1Mjg5YzEwYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yNTU0YTI2OTU4OTc0NTY0OGQxZTJjYjEzODZkN2ZhOCA9ICQoJzxkaXYgaWQ9Imh0bWxfMjU1NGEyNjk1ODk3NDU2NDhkMWUyY2IxMzg2ZDdmYTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkR1ZmZlcmluLCBEb3ZlcmNvdXJ0IFZpbGxhZ2UsIFdlc3QgVG9yb250bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYTRmODNmYTExZTFlNDcwM2JmZjM5MmNjNTI4OWMxMGMuc2V0Q29udGVudChodG1sXzI1NTRhMjY5NTg5NzQ1NjQ4ZDFlMmNiMTM4NmQ3ZmE4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzJlMTU2OGU3ODFhMTQ2YWJhNzY4NmNiYzNlYzQzMGEzLmJpbmRQb3B1cChwb3B1cF9hNGY4M2ZhMTFlMWU0NzAzYmZmMzkyY2M1Mjg5YzEwYyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xODgzYTRlYTE3YzM0ZDAxYWIzMGUzN2FkY2YyZWMxOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0NzkyNjcwMDAwMDAwNiwtNzkuNDE5NzQ5N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85MDY2MWYwNjdjMDU0ZTNlYWJiMTJjMjgwMGYyMzVjZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84OTcxMzVkMTc3MWM0OWM3ODM1ZmZiMmRhZjNkZWQyNCA9ICQoJzxkaXYgaWQ9Imh0bWxfODk3MTM1ZDE3NzFjNDljNzgzNWZmYjJkYWYzZGVkMjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxpdHRsZSBQb3J0dWdhbCwgVHJpbml0eSwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85MDY2MWYwNjdjMDU0ZTNlYWJiMTJjMjgwMGYyMzVjZS5zZXRDb250ZW50KGh0bWxfODk3MTM1ZDE3NzFjNDljNzgzNWZmYjJkYWYzZGVkMjQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTg4M2E0ZWExN2MzNGQwMWFiMzBlMzdhZGNmMmVjMTguYmluZFBvcHVwKHBvcHVwXzkwNjYxZjA2N2MwNTRlM2VhYmIxMmMyODAwZjIzNWNlKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM3NGVlYTlkZmU4ZjQwMmZhNGVkNzI0MmNjYzk5YmFiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjM2ODQ3MiwtNzkuNDI4MTkxNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2YwOTE3NjQ2YjAzNDkzYmExZjUyNmQ3NjdlYjFmZDcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNWRhOTFlYmE0ZDliNGM3MGI5OGRiM2VmOTYzMzk2M2EgPSAkKCc8ZGl2IGlkPSJodG1sXzVkYTkxZWJhNGQ5YjRjNzBiOThkYjNlZjk2MzM5NjNhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ccm9ja3RvbiwgUGFya2RhbGUgVmlsbGFnZSwgRXhoaWJpdGlvbiBQbGFjZSwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zZjA5MTc2NDZiMDM0OTNiYTFmNTI2ZDc2N2ViMWZkNy5zZXRDb250ZW50KGh0bWxfNWRhOTFlYmE0ZDliNGM3MGI5OGRiM2VmOTYzMzk2M2EpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMzc0ZWVhOWRmZThmNDAyZmE0ZWQ3MjQyY2NjOTliYWIuYmluZFBvcHVwKHBvcHVwXzNmMDkxNzY0NmIwMzQ5M2JhMWY1MjZkNzY3ZWIxZmQ3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI1ZWQ5MTRkZWZlNzQ5MmU4YjIzNjA0NjBhMzMwY2E4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYxNjA4MywtNzkuNDY0NzYzMjk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMDVjNmMyMzVlZWVkNDE1ZGFhYjJkMGZhYmU5NWYxMGEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNzI1YmE0ZjRhMThhNGIwZTkxNjk4OTllNzdiMGQzYWYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMGM4OGI0MDI1MTZmNDU4YThkNTVjZTY2ZTQzYzk3MzIgPSAkKCc8ZGl2IGlkPSJodG1sXzBjODhiNDAyNTE2ZjQ1OGE4ZDU1Y2U2NmU0M2M5NzMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IaWdoIFBhcmssIFRoZSBKdW5jdGlvbiBTb3V0aCwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83MjViYTRmNGExOGE0YjBlOTE2OTg5OWU3N2IwZDNhZi5zZXRDb250ZW50KGh0bWxfMGM4OGI0MDI1MTZmNDU4YThkNTVjZTY2ZTQzYzk3MzIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMjVlZDkxNGRlZmU3NDkyZThiMjM2MDQ2MGEzMzBjYTguYmluZFBvcHVwKHBvcHVwXzcyNWJhNGY0YTE4YTRiMGU5MTY5ODk5ZTc3YjBkM2FmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNiMmEwOWQ3MTljNTRiYjE5YWM5OWIwMzI0YzA4N2UyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4OTU5NywtNzkuNDU2MzI1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzJhYjVmMDcxMjIyYzRhMTU5MWFkOGE3MDMwM2RmZGQ5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ2NjYwN2Y0YWQyMDQwYmY4ZGRhN2I2OTVkMjYxOGIyID0gJCgnPGRpdiBpZD0iaHRtbF80NjY2MDdmNGFkMjA0MGJmOGRkYTdiNjk1ZDI2MThiMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFya2RhbGUsIFJvbmNlc3ZhbGxlcywgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yYWI1ZjA3MTIyMmM0YTE1OTFhZDhhNzAzMDNkZmRkOS5zZXRDb250ZW50KGh0bWxfNDY2NjA3ZjRhZDIwNDBiZjhkZGE3YjY5NWQyNjE4YjIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2IyYTA5ZDcxOWM1NGJiMTlhYzk5YjAzMjRjMDg3ZTIuYmluZFBvcHVwKHBvcHVwXzJhYjVmMDcxMjIyYzRhMTU5MWFkOGE3MDMwM2RmZGQ5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZmNjgzOGRkYWQzODQ3ZDA4NDNhMDdhNjI2ZTJhY2M5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUxNTcwNiwtNzkuNDg0NDQ5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xMTAxNmE5MjMxNWM0NmEyOTMyN2ZlODAwM2Q4YTU1MCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82MzIxZmVkYzY3OTc0Nzg3OGM5NTU1ZmFlNzdkYjg2MSA9ICQoJzxkaXYgaWQ9Imh0bWxfNjMyMWZlZGM2Nzk3NDc4NzhjOTU1NWZhZTc3ZGI4NjEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJ1bm55bWVkZSwgU3dhbnNlYSwgV2VzdCBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xMTAxNmE5MjMxNWM0NmEyOTMyN2ZlODAwM2Q4YTU1MC5zZXRDb250ZW50KGh0bWxfNjMyMWZlZGM2Nzk3NDc4NzhjOTU1NWZhZTc3ZGI4NjEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZmY2ODM4ZGRhZDM4NDdkMDg0M2EwN2E2MjZlMmFjYzkuYmluZFBvcHVwKHBvcHVwXzExMDE2YTkyMzE1YzQ2YTI5MzI3ZmU4MDAzZDhhNTUwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzliN2ZmYjI4ZmQxMzRmOWI5OTYwNzZmOTJhNzg5NzI0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyMzAxNSwtNzkuMzg5NDkzOF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8wNWM2YzIzNWVlZWQ0MTVkYWFiMmQwZmFiZTk1ZjEwYSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hYjA3OWQyZWY2MTY0YmZiOTlmNWYzY2MzOGEzOWM3NSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80YzMyNTE1ODdhMTM0YTVjYmY2NWUyY2E5MjczNjY1ZCA9ICQoJzxkaXYgaWQ9Imh0bWxfNGMzMjUxNTg3YTEzNGE1Y2JmNjVlMmNhOTI3MzY2NWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlF1ZWVuJiMzOTtzIFBhcmssIE9udGFyaW8gUHJvdmluY2lhbCBHb3Zlcm5tZW50LCBEb3dudG93biBUb3JvbnRvPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9hYjA3OWQyZWY2MTY0YmZiOTlmNWYzY2MzOGEzOWM3NS5zZXRDb250ZW50KGh0bWxfNGMzMjUxNTg3YTEzNGE1Y2JmNjVlMmNhOTI3MzY2NWQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOWI3ZmZiMjhmZDEzNGY5Yjk5NjA3NmY5MmE3ODk3MjQuYmluZFBvcHVwKHBvcHVwX2FiMDc5ZDJlZjYxNjRiZmI5OWY1ZjNjYzM4YTM5Yzc1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEzNDNiNmUwMzc5YTQ4Mjg5MGUyYjM3YjVlMzUyZDE0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyNzQzOSwtNzkuMzIxNTU4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzA1YzZjMjM1ZWVlZDQxNWRhYWIyZDBmYWJlOTVmMTBhKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U3YTE3ZGFjYWE0NjQwZDQ5MzAyM2FmMGZmOGFlNThlID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzJmOGUyYjA0ZDNlYzRjYzQ5NjNjNDhmMWRjZWI3ZjczID0gJCgnPGRpdiBpZD0iaHRtbF8yZjhlMmIwNGQzZWM0Y2M0OTYzYzQ4ZjFkY2ViN2Y3MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnVzaW5lc3MgcmVwbHkgbWFpbCBQcm9jZXNzaW5nIENlbnRyZSwgU291dGggQ2VudHJhbCBMZXR0ZXIgUHJvY2Vzc2luZyBQbGFudCBUb3JvbnRvLCBFYXN0IFRvcm9udG88L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U3YTE3ZGFjYWE0NjQwZDQ5MzAyM2FmMGZmOGFlNThlLnNldENvbnRlbnQoaHRtbF8yZjhlMmIwNGQzZWM0Y2M0OTYzYzQ4ZjFkY2ViN2Y3Myk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xMzQzYjZlMDM3OWE0ODI4OTBlMmIzN2I1ZTM1MmQxNC5iaW5kUG9wdXAocG9wdXBfZTdhMTdkYWNhYTQ2NDBkNDkzMDIzYWYwZmY4YWU1OGUpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg== onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



## **4. Explore and cluster the neighborhoods in Toronto**

#### Define Foursquare Credentials and Version


```python
CLIENT_ID = '' # your Foursquare ID
CLIENT_SECRET = '' # your Foursquare Secret
VERSION = '' # Foursquare API version
```

### **4.1 Explore the first neighborhood in our dataframe**


```python
# Get the neighborhood name
print("The first neighborhood's name is", borough_toronto.loc[0, 'Neighbourhood'])
```

    The first neighborhood's name is The Beaches
    


```python
# Get the neighborhood's latitude and longitude values.
neighborhood_latitude = borough_toronto.loc[0, 'Latitude'] # neighborhood latitude value
neighborhood_longitude = borough_toronto.loc[0, 'Longitude'] # neighborhood longitude value

neighborhood_name = borough_toronto.loc[0, 'Neighbourhood'] # neighborhood name

print('Latitude and longitude values of {} are {}, {}.'.format(neighborhood_name, 
                                                               neighborhood_latitude, 
                                                               neighborhood_longitude))
```

    Latitude and longitude values of The Beaches are 43.67635739999999, -79.2930312.
    

#### **Now, let's get the top 100 venues that are in The Beaches within a radius of 500 meters.**

#### Create the GET request URL and send the GET request


```python
LIMIT = 100 # limit of number of venues returned by Foursquare API
radius = 500 # define radius
url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
    CLIENT_ID, 
    CLIENT_SECRET, 
    VERSION, 
    neighborhood_latitude, 
    neighborhood_longitude, 
    radius, 
    LIMIT)

# get the result to a json file
results = requests.get(url).json()
```

#### Use the **get_category_type** function from the Foursquare lab to extract the category of the venue


```python
def get_category_type(row):
    try:
        categories_list = row['categories']
    except:
        categories_list = row['venue.categories']
        
    if len(categories_list) == 0:
        return None
    else:
        return categories_list[0]['name']
```

#### Now we are ready to clean the json and structure it into a pandas dataframe.


```python
venues = results['response']['groups'][0]['items']
nearby_venues = json_normalize(venues) # flatten JSON

# filter columns
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.lat', 'venue.location.lng']
nearby_venues =nearby_venues.loc[:, filtered_columns]

# filter the category for each row
nearby_venues['venue.categories'] = nearby_venues.apply(get_category_type, axis=1)

# clean columns
nearby_venues.columns = [col.split(".")[-1] for col in nearby_venues.columns]

nearby_venues
```

    C:\Users\ruxin\anaconda3\lib\site-packages\ipykernel_launcher.py:2: FutureWarning: pandas.io.json.json_normalize is deprecated, use pandas.json_normalize instead
      
    




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
      <th>name</th>
      <th>categories</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Glen Manor Ravine</td>
      <td>Trail</td>
      <td>43.676821</td>
      <td>-79.293942</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Big Carrot Natural Food Market</td>
      <td>Health Food Store</td>
      <td>43.678879</td>
      <td>-79.297734</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Grover Pub and Grub</td>
      <td>Pub</td>
      <td>43.679181</td>
      <td>-79.297215</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Upper Beaches</td>
      <td>Neighborhood</td>
      <td>43.680563</td>
      <td>-79.292869</td>
    </tr>
  </tbody>
</table>
</div>



### **4.2 Explore Neighborhoods in Toronto**
#### Create a function to repeat the same process to all the neighborhoods in Toronto


```python
def getNearbyVenues(names, latitudes, longitudes, radius=500):
    venues_list=[]
    
    for name, lat, lng in zip(names, latitudes, longitudes):
        # print(name)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            name, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['Neighborhood', 
                  'Neighborhood Latitude', 
                  'Neighborhood Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```

#### Run the above function on each neighborhood and create a new dataframe


```python
borough_toronto_venues = getNearbyVenues(names=borough_toronto['Neighbourhood'],
                                   latitudes=borough_toronto['Latitude'],
                                   longitudes=borough_toronto['Longitude']
                                  )
```


```python
borough_toronto_venues.head()
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
      <th>Neighborhood</th>
      <th>Neighborhood Latitude</th>
      <th>Neighborhood Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
      <td>Glen Manor Ravine</td>
      <td>43.676821</td>
      <td>-79.293942</td>
      <td>Trail</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
      <td>The Big Carrot Natural Food Market</td>
      <td>43.678879</td>
      <td>-79.297734</td>
      <td>Health Food Store</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
      <td>Grover Pub and Grub</td>
      <td>43.679181</td>
      <td>-79.297215</td>
      <td>Pub</td>
    </tr>
    <tr>
      <th>3</th>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
      <td>Upper Beaches</td>
      <td>43.680563</td>
      <td>-79.292869</td>
      <td>Neighborhood</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Danforth West, Riverdale</td>
      <td>43.679557</td>
      <td>-79.352188</td>
      <td>MenEssentials</td>
      <td>43.677820</td>
      <td>-79.351265</td>
      <td>Cosmetics Shop</td>
    </tr>
  </tbody>
</table>
</div>



#### Check how many venues were returned for each neighborhood


```python
borough_toronto_venues.groupby('Neighborhood').count()
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
      <th>Neighborhood Latitude</th>
      <th>Neighborhood Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
    <tr>
      <th>Neighborhood</th>
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
      <th>Berczy Park</th>
      <td>57</td>
      <td>57</td>
      <td>57</td>
      <td>57</td>
      <td>57</td>
      <td>57</td>
    </tr>
    <tr>
      <th>Brockton, Parkdale Village, Exhibition Place</th>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
    </tr>
    <tr>
      <th>Business reply mail Processing Centre, South Central Letter Processing Plant Toronto</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>CN Tower, King and Spadina, Railway Lands, Harbourfront West, Bathurst Quay, South Niagara, Island airport</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>Central Bay Street</th>
      <td>68</td>
      <td>68</td>
      <td>68</td>
      <td>68</td>
      <td>68</td>
      <td>68</td>
    </tr>
    <tr>
      <th>Christie</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>Church and Wellesley</th>
      <td>77</td>
      <td>77</td>
      <td>77</td>
      <td>77</td>
      <td>77</td>
      <td>77</td>
    </tr>
    <tr>
      <th>Commerce Court, Victoria Hotel</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Davisville</th>
      <td>35</td>
      <td>35</td>
      <td>35</td>
      <td>35</td>
      <td>35</td>
      <td>35</td>
    </tr>
    <tr>
      <th>Davisville North</th>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
    </tr>
    <tr>
      <th>Dufferin, Dovercourt Village</th>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
      <td>14</td>
    </tr>
    <tr>
      <th>First Canadian Place, Underground city</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Forest Hill North &amp; West, Forest Hill Road Park</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Garden District, Ryerson</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Harbourfront East, Union Station, Toronto Islands</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>High Park, The Junction South</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
    </tr>
    <tr>
      <th>India Bazaar, The Beaches West</th>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
    </tr>
    <tr>
      <th>Kensington Market, Chinatown, Grange Park</th>
      <td>60</td>
      <td>60</td>
      <td>60</td>
      <td>60</td>
      <td>60</td>
      <td>60</td>
    </tr>
    <tr>
      <th>Lawrence Park</th>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>Little Portugal, Trinity</th>
      <td>44</td>
      <td>44</td>
      <td>44</td>
      <td>44</td>
      <td>44</td>
      <td>44</td>
    </tr>
    <tr>
      <th>Moore Park, Summerhill East</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>North Toronto West,  Lawrence Park</th>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
    </tr>
    <tr>
      <th>Parkdale, Roncesvalles</th>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
    </tr>
    <tr>
      <th>Queen's Park, Ontario Provincial Government</th>
      <td>33</td>
      <td>33</td>
      <td>33</td>
      <td>33</td>
      <td>33</td>
      <td>33</td>
    </tr>
    <tr>
      <th>Regent Park, Harbourfront</th>
      <td>45</td>
      <td>45</td>
      <td>45</td>
      <td>45</td>
      <td>45</td>
      <td>45</td>
    </tr>
    <tr>
      <th>Richmond, Adelaide, King</th>
      <td>94</td>
      <td>94</td>
      <td>94</td>
      <td>94</td>
      <td>94</td>
      <td>94</td>
    </tr>
    <tr>
      <th>Rosedale</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Roselawn</th>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>Runnymede, Swansea</th>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
    </tr>
    <tr>
      <th>St. James Town</th>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
      <td>79</td>
    </tr>
    <tr>
      <th>St. James Town, Cabbagetown</th>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
    </tr>
    <tr>
      <th>Stn A PO Boxes</th>
      <td>97</td>
      <td>97</td>
      <td>97</td>
      <td>97</td>
      <td>97</td>
      <td>97</td>
    </tr>
    <tr>
      <th>Studio District</th>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
    </tr>
    <tr>
      <th>Summerhill West, Rathnelly, South Hill, Forest Hill SE, Deer Park</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
    <tr>
      <th>The Annex, North Midtown, Yorkville</th>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
      <td>20</td>
    </tr>
    <tr>
      <th>The Beaches</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>The Danforth West, Riverdale</th>
      <td>43</td>
      <td>43</td>
      <td>43</td>
      <td>43</td>
      <td>43</td>
      <td>43</td>
    </tr>
    <tr>
      <th>Toronto Dominion Centre, Design Exchange</th>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
    </tr>
    <tr>
      <th>University of Toronto, Harbord</th>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
      <td>37</td>
    </tr>
  </tbody>
</table>
</div>



#### Find out how many unique categories can be curated from all the returned venues


```python
print('There are {} uniques categories.'.format(len(borough_toronto_venues['Venue Category'].unique())))
```

    There are 236 uniques categories.
    

### **4.3 Analyze each neighborhood**


```python
# one hot encoding
borough_toronto_onehot = pd.get_dummies(borough_toronto_venues[['Venue Category']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
borough_toronto_onehot['Neighborhood'] = borough_toronto_venues['Neighborhood'] 

# move neighborhood column to the first column
fixed_columns = [borough_toronto_onehot.columns[-1]] + list(borough_toronto_onehot.columns[:-1])
borough_toronto_onehot = borough_toronto_onehot[fixed_columns]

borough_toronto_onehot.head()
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
      <th>Yoga Studio</th>
      <th>Afghan Restaurant</th>
      <th>Airport</th>
      <th>Airport Food Court</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>Airport Terminal</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>Aquarium</th>
      <th>...</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Train Station</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Video Game Store</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Women's Store</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 236 columns</p>
</div>



#### Group rows by neighborhood and by taking the mean of the frequency of occurrence of each category


```python
borough_toronto_grouped = borough_toronto_onehot.groupby('Neighborhood').mean().reset_index()
borough_toronto_grouped.head()
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
      <th>Neighborhood</th>
      <th>Yoga Studio</th>
      <th>Afghan Restaurant</th>
      <th>Airport</th>
      <th>Airport Food Court</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>Airport Terminal</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>...</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Train Station</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Video Game Store</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Women's Store</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Berczy Park</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.017544</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brockton, Parkdale Village, Exhibition Place</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Business reply mail Processing Centre, South C...</td>
      <td>0.062500</td>
      <td>0.0</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CN Tower, King and Spadina, Railway Lands, Har...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0625</td>
      <td>0.0625</td>
      <td>0.125</td>
      <td>0.125</td>
      <td>0.125</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Central Bay Street</td>
      <td>0.014706</td>
      <td>0.0</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.014706</td>
      <td>0.0</td>
      <td>0.014706</td>
      <td>0.0</td>
      <td>0.014706</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 236 columns</p>
</div>



#### Print each neighborhood along with the top 5 most common venues


```python
num_top_venues = 5

for hood in borough_toronto_grouped['Neighborhood']:
    print("----"+hood+"----")
    temp = borough_toronto_grouped[borough_toronto_grouped['Neighborhood'] == hood].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```

    ----Berczy Park----
              venue  freq
    0   Coffee Shop  0.09
    1  Cocktail Bar  0.05
    2   Cheese Shop  0.04
    3        Bakery  0.04
    4    Restaurant  0.04
    
    
    ----Brockton, Parkdale Village, Exhibition Place----
                       venue  freq
    0                   Café  0.14
    1         Breakfast Spot  0.09
    2            Coffee Shop  0.09
    3  Performing Arts Venue  0.05
    4           Intersection  0.05
    
    
    ----Business reply mail Processing Centre, South Central Letter Processing Plant Toronto----
             venue  freq
    0  Yoga Studio  0.06
    1      Brewery  0.06
    2   Smoke Shop  0.06
    3   Skate Park  0.06
    4   Restaurant  0.06
    
    
    ----CN Tower, King and Spadina, Railway Lands, Harbourfront West, Bathurst Quay, South Niagara, Island airport----
                  venue  freq
    0    Airport Lounge  0.12
    1   Airport Service  0.12
    2  Airport Terminal  0.12
    3  Sculpture Garden  0.06
    4   Harbor / Marina  0.06
    
    
    ----Central Bay Street----
                     venue  freq
    0          Coffee Shop  0.16
    1  Japanese Restaurant  0.06
    2       Sandwich Place  0.06
    3   Italian Restaurant  0.06
    4                 Café  0.04
    
    
    ----Christie----
               venue  freq
    0  Grocery Store  0.25
    1           Café  0.19
    2           Park  0.12
    3     Baby Store  0.06
    4      Nightclub  0.06
    
    
    ----Church and Wellesley----
                     venue  freq
    0          Coffee Shop  0.08
    1     Sushi Restaurant  0.06
    2  Japanese Restaurant  0.05
    3           Restaurant  0.04
    4              Gay Bar  0.04
    
    
    ----Commerce Court, Victoria Hotel----
                     venue  freq
    0          Coffee Shop  0.12
    1           Restaurant  0.07
    2                 Café  0.07
    3                Hotel  0.06
    4  American Restaurant  0.04
    
    
    ----Davisville----
                    venue  freq
    0         Pizza Place  0.11
    1      Sandwich Place  0.09
    2        Dessert Shop  0.09
    3  Italian Restaurant  0.06
    4                Café  0.06
    
    
    ----Davisville North----
                      venue  freq
    0  Gym / Fitness Center  0.12
    1                 Hotel  0.12
    2           Pizza Place  0.12
    3      Department Store  0.12
    4        Sandwich Place  0.12
    
    
    ----Dufferin, Dovercourt Village----
                           venue  freq
    0                   Pharmacy  0.14
    1                     Bakery  0.14
    2  Middle Eastern Restaurant  0.07
    3                       Bank  0.07
    4                    Brewery  0.07
    
    
    ----First Canadian Place, Underground city----
             venue  freq
    0  Coffee Shop  0.10
    1         Café  0.08
    2        Hotel  0.05
    3   Restaurant  0.05
    4          Gym  0.04
    
    
    ----Forest Hill North & West, Forest Hill Road Park----
                     venue  freq
    0                 Park  0.25
    1        Jewelry Store  0.25
    2                Trail  0.25
    3     Sushi Restaurant  0.25
    4  Monument / Landmark  0.00
    
    
    ----Garden District, Ryerson----
                           venue  freq
    0             Clothing Store  0.09
    1                Coffee Shop  0.07
    2             Cosmetics Shop  0.03
    3  Middle Eastern Restaurant  0.03
    4                       Café  0.03
    
    
    ----Harbourfront East, Union Station, Toronto Islands----
             venue  freq
    0  Coffee Shop  0.13
    1     Aquarium  0.05
    2        Hotel  0.04
    3         Café  0.04
    4   Restaurant  0.03
    
    
    ----High Park, The Junction South----
                     venue  freq
    0                 Café  0.08
    1   Mexican Restaurant  0.08
    2      Thai Restaurant  0.08
    3  Fried Chicken Joint  0.04
    4               Bakery  0.04
    
    
    ----India Bazaar, The Beaches West----
                      venue  freq
    0                  Park  0.09
    1           Pizza Place  0.09
    2  Fast Food Restaurant  0.09
    3                   Pub  0.05
    4               Brewery  0.05
    
    
    ----Kensington Market, Chinatown, Grange Park----
                               venue  freq
    0                           Café  0.08
    1                    Coffee Shop  0.05
    2          Vietnamese Restaurant  0.05
    3                         Bakery  0.05
    4  Vegetarian / Vegan Restaurant  0.05
    
    
    ----Lawrence Park----
                     venue  freq
    0                 Park  0.33
    1             Bus Line  0.33
    2          Swim School  0.33
    3          Yoga Studio  0.00
    4  Monument / Landmark  0.00
    
    
    ----Little Portugal, Trinity----
                               venue  freq
    0                            Bar  0.11
    1                     Restaurant  0.05
    2               Asian Restaurant  0.05
    3          Vietnamese Restaurant  0.05
    4  Vegetarian / Vegan Restaurant  0.05
    
    
    ----Moore Park, Summerhill East----
                     venue  freq
    0               Lawyer   1.0
    1          Yoga Studio   0.0
    2  Moroccan Restaurant   0.0
    3     Malay Restaurant   0.0
    4               Market   0.0
    
    
    ----North Toronto West,  Lawrence Park----
                  venue  freq
    0    Clothing Store  0.15
    1       Coffee Shop  0.10
    2       Yoga Studio  0.05
    3              Café  0.05
    4  Toy / Game Store  0.05
    
    
    ----Parkdale, Roncesvalles----
                    venue  freq
    0      Breakfast Spot  0.13
    1           Gift Shop  0.13
    2        Dessert Shop  0.07
    3  Italian Restaurant  0.07
    4                 Bar  0.07
    
    
    ----Queen's Park, Ontario Provincial Government----
                  venue  freq
    0       Coffee Shop  0.24
    1             Diner  0.06
    2  Sushi Restaurant  0.06
    3       Yoga Studio  0.03
    4              Bank  0.03
    
    
    ----Regent Park, Harbourfront----
                venue  freq
    0     Coffee Shop  0.18
    1             Pub  0.07
    2            Park  0.07
    3          Bakery  0.07
    4  Breakfast Spot  0.04
    
    
    ----Richmond, Adelaide, King----
               venue  freq
    0    Coffee Shop  0.11
    1           Café  0.05
    2     Restaurant  0.05
    3            Gym  0.04
    4  Deli / Bodega  0.03
    
    
    ----Rosedale----
                     venue  freq
    0                 Park  0.50
    1           Playground  0.25
    2                Trail  0.25
    3  Moroccan Restaurant  0.00
    4   Mac & Cheese Joint  0.00
    
    
    ----Roselawn----
                  venue  freq
    0            Garden  0.33
    1      Home Service  0.33
    2       Music Venue  0.33
    3     Movie Theater  0.00
    4  Malay Restaurant  0.00
    
    
    ----Runnymede, Swansea----
                  venue  freq
    0       Coffee Shop  0.11
    1              Café  0.08
    2  Sushi Restaurant  0.05
    3             Diner  0.05
    4       Pizza Place  0.05
    
    
    ----St. James Town----
              venue  freq
    0   Coffee Shop  0.06
    1          Café  0.06
    2     Gastropub  0.04
    3  Cocktail Bar  0.04
    4    Restaurant  0.04
    
    
    ----St. James Town, Cabbagetown----
                    venue  freq
    0                Park  0.07
    1              Bakery  0.07
    2         Coffee Shop  0.07
    3                Café  0.07
    4  Italian Restaurant  0.04
    
    
    ----Stn A PO Boxes----
                    venue  freq
    0         Coffee Shop  0.10
    1  Seafood Restaurant  0.04
    2                Café  0.04
    3          Restaurant  0.03
    4            Beer Bar  0.03
    
    
    ----Studio District----
                     venue  freq
    0                 Café  0.10
    1          Coffee Shop  0.07
    2              Brewery  0.05
    3            Gastropub  0.05
    4  American Restaurant  0.05
    
    
    ----Summerhill West, Rathnelly, South Hill, Forest Hill SE, Deer Park----
                  venue  freq
    0       Coffee Shop  0.12
    1               Pub  0.12
    2        Restaurant  0.06
    3      Liquor Store  0.06
    4  Sushi Restaurant  0.06
    
    
    ----The Annex, North Midtown, Yorkville----
                           venue  freq
    0             Sandwich Place  0.15
    1                       Café  0.15
    2                Coffee Shop  0.10
    3  Middle Eastern Restaurant  0.05
    4                 Donut Shop  0.05
    
    
    ----The Beaches----
                     venue  freq
    0    Health Food Store  0.25
    1                Trail  0.25
    2                  Pub  0.25
    3          Yoga Studio  0.00
    4  Moroccan Restaurant  0.00
    
    
    ----The Danforth West, Riverdale----
                    venue  freq
    0    Greek Restaurant  0.19
    1         Coffee Shop  0.07
    2  Italian Restaurant  0.07
    3          Restaurant  0.05
    4      Ice Cream Shop  0.05
    
    
    ----Toronto Dominion Centre, Design Exchange----
                     venue  freq
    0          Coffee Shop  0.10
    1                Hotel  0.08
    2                 Café  0.07
    3           Restaurant  0.04
    4  Japanese Restaurant  0.03
    
    
    ----University of Toronto, Harbord----
                    venue  freq
    0                Café  0.14
    1           Bookstore  0.08
    2  Italian Restaurant  0.05
    3          Restaurant  0.05
    4              Bakery  0.05
    
    
    

#### **Put that into a pandas dataframe**
#### A function that sorts the venues in descending order.


```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
```

#### Create a new dataframe and displace the top 10 venues for each neighborhood


```python
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['Neighborhood']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
neighborhoods_venues_sorted = pd.DataFrame(columns=columns)
neighborhoods_venues_sorted['Neighborhood'] = borough_toronto_grouped['Neighborhood']

for ind in np.arange(borough_toronto_grouped.shape[0]):
    neighborhoods_venues_sorted.iloc[ind, 1:] = return_most_common_venues(borough_toronto_grouped.iloc[ind, :], num_top_venues)

neighborhoods_venues_sorted.head()
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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Berczy Park</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Restaurant</td>
      <td>Bakery</td>
      <td>Beer Bar</td>
      <td>Seafood Restaurant</td>
      <td>Cheese Shop</td>
      <td>Café</td>
      <td>Irish Pub</td>
      <td>Pharmacy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brockton, Parkdale Village, Exhibition Place</td>
      <td>Café</td>
      <td>Breakfast Spot</td>
      <td>Coffee Shop</td>
      <td>Bakery</td>
      <td>Intersection</td>
      <td>Bar</td>
      <td>Italian Restaurant</td>
      <td>Restaurant</td>
      <td>Climbing Gym</td>
      <td>Furniture / Home Store</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Business reply mail Processing Centre, South C...</td>
      <td>Gym / Fitness Center</td>
      <td>Smoke Shop</td>
      <td>Auto Workshop</td>
      <td>Brewery</td>
      <td>Burrito Place</td>
      <td>Comic Shop</td>
      <td>Farmers Market</td>
      <td>Fast Food Restaurant</td>
      <td>Garden</td>
      <td>Garden Center</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CN Tower, King and Spadina, Railway Lands, Har...</td>
      <td>Airport Lounge</td>
      <td>Airport Service</td>
      <td>Airport Terminal</td>
      <td>Coffee Shop</td>
      <td>Boutique</td>
      <td>Rental Car Location</td>
      <td>Plane</td>
      <td>Boat or Ferry</td>
      <td>Bar</td>
      <td>Harbor / Marina</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Central Bay Street</td>
      <td>Coffee Shop</td>
      <td>Italian Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Sandwich Place</td>
      <td>Café</td>
      <td>Salad Place</td>
      <td>Bar</td>
      <td>Bubble Tea Shop</td>
      <td>Burger Joint</td>
      <td>Department Store</td>
    </tr>
  </tbody>
</table>
</div>



### **4.4 Cluster Neighborhoods**
#### Run k-means to cluster the neighborhood into 5 clusters.


```python
from sklearn.cluster import KMeans
# set number of clusters
kclusters = 5

toronto_grouped_clustering = borough_toronto_grouped.drop('Neighborhood', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(toronto_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10] 
```




    array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0])



#### Create a new dataframe that includes the cluster as well as the top 10 venues for each neighborhood.


```python
# add clustering labels
neighborhoods_venues_sorted.insert(0, 'Cluster Labels', kmeans.labels_)

toronto_merged = borough_toronto

# merge toronto_grouped with toronto_data to add latitude/longitude for each neighborhood
toronto_merged = toronto_merged.join(neighborhoods_venues_sorted.set_index('Neighborhood'), on='Neighbourhood')

toronto_merged.head() # check the last columns!
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
      <th>PostalCode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M4E</td>
      <td>East Toronto</td>
      <td>The Beaches</td>
      <td>43.676357</td>
      <td>-79.293031</td>
      <td>0</td>
      <td>Trail</td>
      <td>Health Food Store</td>
      <td>Pub</td>
      <td>Doner Restaurant</td>
      <td>Dessert Shop</td>
      <td>Diner</td>
      <td>Discount Store</td>
      <td>Distribution Center</td>
      <td>Dog Run</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M4K</td>
      <td>East Toronto</td>
      <td>The Danforth West, Riverdale</td>
      <td>43.679557</td>
      <td>-79.352188</td>
      <td>0</td>
      <td>Greek Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Bookstore</td>
      <td>Ice Cream Shop</td>
      <td>Furniture / Home Store</td>
      <td>Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Pub</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M4L</td>
      <td>East Toronto</td>
      <td>India Bazaar, The Beaches West</td>
      <td>43.668999</td>
      <td>-79.315572</td>
      <td>0</td>
      <td>Fast Food Restaurant</td>
      <td>Park</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Pet Store</td>
      <td>Fish &amp; Chips Shop</td>
      <td>Ice Cream Shop</td>
      <td>Sushi Restaurant</td>
      <td>Brewery</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M4M</td>
      <td>East Toronto</td>
      <td>Studio District</td>
      <td>43.659526</td>
      <td>-79.340923</td>
      <td>0</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>American Restaurant</td>
      <td>Bakery</td>
      <td>Brewery</td>
      <td>Gastropub</td>
      <td>Gym / Fitness Center</td>
      <td>Fish Market</td>
      <td>Pet Store</td>
      <td>Park</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M4N</td>
      <td>Central Toronto</td>
      <td>Lawrence Park</td>
      <td>43.728020</td>
      <td>-79.388790</td>
      <td>2</td>
      <td>Park</td>
      <td>Swim School</td>
      <td>Bus Line</td>
      <td>Department Store</td>
      <td>Event Space</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



#### Visualize the resulting clusters


```python
# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=11)

# set color scheme for the clusters
x = np.arange(kclusters)
ys = [i + x + (i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(toronto_merged['Latitude'], toronto_merged['Longitude'], toronto_merged['Neighbourhood'], toronto_merged['Cluster Labels']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMycsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDMuNjUzNDgxNywtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfOGVjZGIxMTJjYWQ1NGQ5MThmYTkxYmQ4MzQ5Zjg3YzggPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RlY2Y5ZTg0ZWRiMDRiNmJiZDdmMWViMTFhMWZkZDM4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjc2MzU3Mzk5OTk5OTksLTc5LjI5MzAzMTJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfN2RmMDU0NTMwN2MzNGQxZmE5OTZiNzk4MTAyZGMyNzAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYmYwNWJkODQyZDRhNDBmYmI2Zjk3NWNhN2U2MjU0ZjggPSAkKCc8ZGl2IGlkPSJodG1sX2JmMDViZDg0MmQ0YTQwZmJiNmY5NzVjYTdlNjI1NGY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgQmVhY2hlcyBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzdkZjA1NDUzMDdjMzRkMWZhOTk2Yjc5ODEwMmRjMjcwLnNldENvbnRlbnQoaHRtbF9iZjA1YmQ4NDJkNGE0MGZiYjZmOTc1Y2E3ZTYyNTRmOCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kZWNmOWU4NGVkYjA0YjZiYmQ3ZjFlYjExYTFmZGQzOC5iaW5kUG9wdXAocG9wdXBfN2RmMDU0NTMwN2MzNGQxZmE5OTZiNzk4MTAyZGMyNzApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDY0Nzk4M2E0ZjU2NGM1ZmI1NDBjMjdmYmZkOTUwZjkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Nzk1NTcxLC03OS4zNTIxODhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNDUwODk2NWI1MTU2NDVmN2FkMDhlNjEwMTUyOTAzZGYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzdjZWU1MDhlMjE3NGQ0ZDg2ZDhjZGZlMGE4OTg2YjkgPSAkKCc8ZGl2IGlkPSJodG1sX2M3Y2VlNTA4ZTIxNzRkNGQ4NmQ4Y2RmZTBhODk4NmI5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgRGFuZm9ydGggV2VzdCwgUml2ZXJkYWxlIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDUwODk2NWI1MTU2NDVmN2FkMDhlNjEwMTUyOTAzZGYuc2V0Q29udGVudChodG1sX2M3Y2VlNTA4ZTIxNzRkNGQ4NmQ4Y2RmZTBhODk4NmI5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q2NDc5ODNhNGY1NjRjNWZiNTQwYzI3ZmJmZDk1MGY5LmJpbmRQb3B1cChwb3B1cF80NTA4OTY1YjUxNTY0NWY3YWQwOGU2MTAxNTI5MDNkZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jYjI4MGZkY2RmYmQ0NzRhODU1NjY3YzMzZmExOGFkMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2ODk5ODUsLTc5LjMxNTU3MTU5OTk5OTk4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzk5ZWQ4MDA5YjQ3MDRhNTU5M2E2NjgxYjEyNTQyMjg5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzNjNDNmNWFmYmI4YzRiZWU4Zjc5ZmRlOWQzZjNmMDcxID0gJCgnPGRpdiBpZD0iaHRtbF8zYzQzZjVhZmJiOGM0YmVlOGY3OWZkZTlkM2YzZjA3MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SW5kaWEgQmF6YWFyLCBUaGUgQmVhY2hlcyBXZXN0IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTllZDgwMDliNDcwNGE1NTkzYTY2ODFiMTI1NDIyODkuc2V0Q29udGVudChodG1sXzNjNDNmNWFmYmI4YzRiZWU4Zjc5ZmRlOWQzZjNmMDcxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2NiMjgwZmRjZGZiZDQ3NGE4NTU2NjdjMzNmYTE4YWQyLmJpbmRQb3B1cChwb3B1cF85OWVkODAwOWI0NzA0YTU1OTNhNjY4MWIxMjU0MjI4OSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84NWVlNTFkYmNiMWY0NGZkYmE0YmJiNjQ5MDE0MTE3YiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1OTUyNTUsLTc5LjM0MDkyM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85ZGI4ZjU5ZDM4MGM0ZmExOTZhOTY3YTQ3YjgyNjVlNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mYjJkZDlhMmM2ZDM0MzdjYmE1NzcxYjE2YTYxZTg0YyA9ICQoJzxkaXYgaWQ9Imh0bWxfZmIyZGQ5YTJjNmQzNDM3Y2JhNTc3MWIxNmE2MWU4NGMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0dWRpbyBEaXN0cmljdCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzlkYjhmNTlkMzgwYzRmYTE5NmE5NjdhNDdiODI2NWU3LnNldENvbnRlbnQoaHRtbF9mYjJkZDlhMmM2ZDM0MzdjYmE1NzcxYjE2YTYxZTg0Yyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84NWVlNTFkYmNiMWY0NGZkYmE0YmJiNjQ5MDE0MTE3Yi5iaW5kUG9wdXAocG9wdXBfOWRiOGY1OWQzODBjNGZhMTk2YTk2N2E0N2I4MjY1ZTcpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjY4NWU4MTIxZTJlNDU0OWFjZjRhODhmZTUxODBmYWUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MjgwMjA1LC03OS4zODg3OTAxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMwMGI1ZWIiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMDBiNWViIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNlNzE4ZjM1NTMzODQ2MDE5N2I5OGZmNzdlZWRiYTEyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzZiZjMwNGI5NTJjNDRmMzZhNjA4OWM0MTRjMDJmMmYzID0gJCgnPGRpdiBpZD0iaHRtbF82YmYzMDRiOTUyYzQ0ZjM2YTYwODljNDE0YzAyZjJmMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGF3cmVuY2UgUGFyayBDbHVzdGVyIDI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzNlNzE4ZjM1NTMzODQ2MDE5N2I5OGZmNzdlZWRiYTEyLnNldENvbnRlbnQoaHRtbF82YmYzMDRiOTUyYzQ0ZjM2YTYwODljNDE0YzAyZjJmMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iNjg1ZTgxMjFlMmU0NTQ5YWNmNGE4OGZlNTE4MGZhZS5iaW5kUG9wdXAocG9wdXBfM2U3MThmMzU1MzM4NDYwMTk3Yjk4ZmY3N2VlZGJhMTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNWJmNmQ2MzExZGQ4NGFlODkxZWM5Nzg3NGY2NjhjMDMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTI3NTExLC03OS4zOTAxOTc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2I5NGIwMTc0MDk5MzQ0NTdiMjM4ZDNkYTU3NmZmYWEyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzMwODRjNmRhMzAyMzQzNTU5NmNmNjk2ZmNkYzE5NTY3ID0gJCgnPGRpdiBpZD0iaHRtbF8zMDg0YzZkYTMwMjM0MzU1OTZjZjY5NmZjZGMxOTU2NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGF2aXN2aWxsZSBOb3J0aCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2I5NGIwMTc0MDk5MzQ0NTdiMjM4ZDNkYTU3NmZmYWEyLnNldENvbnRlbnQoaHRtbF8zMDg0YzZkYTMwMjM0MzU1OTZjZjY5NmZjZGMxOTU2Nyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81YmY2ZDYzMTFkZDg0YWU4OTFlYzk3ODc0ZjY2OGMwMy5iaW5kUG9wdXAocG9wdXBfYjk0YjAxNzQwOTkzNDQ1N2IyMzhkM2RhNTc2ZmZhYTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjQ3OTA5YWZmODQwNGVmMDg5Y2VjMDUxOTczMmFlOGEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTUzODM0LC03OS40MDU2Nzg0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jYjRmMTNkMTVmNmY0MjA0YTdmOTQxMDdiN2MzYjg4OCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zNzQxNTMwYmZmMDU0MTUwODMxNzYzNTE4ZmQzMGRlMSA9ICQoJzxkaXYgaWQ9Imh0bWxfMzc0MTUzMGJmZjA1NDE1MDgzMTc2MzUxOGZkMzBkZTEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIFRvcm9udG8gV2VzdCwgIExhd3JlbmNlIFBhcmsgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jYjRmMTNkMTVmNmY0MjA0YTdmOTQxMDdiN2MzYjg4OC5zZXRDb250ZW50KGh0bWxfMzc0MTUzMGJmZjA1NDE1MDgzMTc2MzUxOGZkMzBkZTEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNjQ3OTA5YWZmODQwNGVmMDg5Y2VjMDUxOTczMmFlOGEuYmluZFBvcHVwKHBvcHVwX2NiNGYxM2QxNWY2ZjQyMDRhN2Y5NDEwN2I3YzNiODg4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzg2NTBiYzA5NDk3ZjQ1Yzk5NThlMDVlZWFiNjY2OGY5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzA0MzI0NCwtNzkuMzg4NzkwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iZDE5MTA2MTJjZjk0MDFlODM0OWNkY2ZkZjY2MmMyMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hMWE0OWU1ZGYzZTE0NjlmOWM4MjJmYjQ0NGM0ODg5MCA9ICQoJzxkaXYgaWQ9Imh0bWxfYTFhNDllNWRmM2UxNDY5ZjljODIyZmI0NDRjNDg4OTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRhdmlzdmlsbGUgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iZDE5MTA2MTJjZjk0MDFlODM0OWNkY2ZkZjY2MmMyMi5zZXRDb250ZW50KGh0bWxfYTFhNDllNWRmM2UxNDY5ZjljODIyZmI0NDRjNDg4OTApOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfODY1MGJjMDk0OTdmNDVjOTk1OGUwNWVlYWI2NjY4ZjkuYmluZFBvcHVwKHBvcHVwX2JkMTkxMDYxMmNmOTQwMWU4MzQ5Y2RjZmRmNjYyYzIyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdlYmZiNWUwYzQ5YzRlZTM5ZjNhYTk0YTgyMzRmY2RhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjg5NTc0MywtNzkuMzgzMTU5OTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfN2IzNzI1YjYwNDg1NDVkNmFmNWU1ZTk2ZDY1YjEwYTUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYjVlZmQyMzljZWNiNDc3NWEyMDRmMGNlN2I0NDQwZGMgPSAkKCc8ZGl2IGlkPSJodG1sX2I1ZWZkMjM5Y2VjYjQ3NzVhMjA0ZjBjZTdiNDQ0MGRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb29yZSBQYXJrLCBTdW1tZXJoaWxsIEVhc3QgQ2x1c3RlciAxPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83YjM3MjViNjA0ODU0NWQ2YWY1ZTVlOTZkNjViMTBhNS5zZXRDb250ZW50KGh0bWxfYjVlZmQyMzljZWNiNDc3NWEyMDRmMGNlN2I0NDQwZGMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2ViZmI1ZTBjNDljNGVlMzlmM2FhOTRhODIzNGZjZGEuYmluZFBvcHVwKHBvcHVwXzdiMzcyNWI2MDQ4NTQ1ZDZhZjVlNWU5NmQ2NWIxMGE1KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzVhNTliNmZlOWQ3YzRlODI4MTY3YjI2NjJkNDEyN2VkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjg2NDEyMjk5OTk5OTksLTc5LjQwMDA0OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNjVjNzJmOTAzMjE5NDg2Y2I4OGJiODFiZGViODM4YWMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYmM1MjhlMTk5Nzk4NDI2Yjk3NTVmNDQ3NjQyMGVlODkgPSAkKCc8ZGl2IGlkPSJodG1sX2JjNTI4ZTE5OTc5ODQyNmI5NzU1ZjQ0NzY0MjBlZTg5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdW1tZXJoaWxsIFdlc3QsIFJhdGhuZWxseSwgU291dGggSGlsbCwgRm9yZXN0IEhpbGwgU0UsIERlZXIgUGFyayBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY1YzcyZjkwMzIxOTQ4NmNiODhiYjgxYmRlYjgzOGFjLnNldENvbnRlbnQoaHRtbF9iYzUyOGUxOTk3OTg0MjZiOTc1NWY0NDc2NDIwZWU4OSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81YTU5YjZmZTlkN2M0ZTgyODE2N2IyNjYyZDQxMjdlZC5iaW5kUG9wdXAocG9wdXBfNjVjNzJmOTAzMjE5NDg2Y2I4OGJiODFiZGViODM4YWMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNmM0OTlkNTJiN2ZiNGI3M2JkM2FjOTUzMzVhODEzZDIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Nzk1NjI2LC03OS4zNzc1Mjk0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODBmZmI0IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwZmZiNCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wM2RhODEyOTcxMzE0MDc5YmYzYjU0NGJlYThlMGFhZCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8xOGM3YjVmMjk4YWQ0MTczOTllNTVhOWY2M2EzMDhhZSA9ICQoJzxkaXYgaWQ9Imh0bWxfMThjN2I1ZjI5OGFkNDE3Mzk5ZTU1YTlmNjNhMzA4YWUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJvc2VkYWxlIENsdXN0ZXIgMzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMDNkYTgxMjk3MTMxNDA3OWJmM2I1NDRiZWE4ZTBhYWQuc2V0Q29udGVudChodG1sXzE4YzdiNWYyOThhZDQxNzM5OWU1NWE5ZjYzYTMwOGFlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZjNDk5ZDUyYjdmYjRiNzNiZDNhYzk1MzM1YTgxM2QyLmJpbmRQb3B1cChwb3B1cF8wM2RhODEyOTcxMzE0MDc5YmYzYjU0NGJlYThlMGFhZCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81ZTcwZjQyZDhjYWE0MzY5ODZkZDE4YzQ2NmNmZDQ2MiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2Nzk2NywtNzkuMzY3Njc1M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zMTI5MjI4ZGUyY2U0OTMzYTM3YjZjODgxZGEyZTkzZSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8zMDJlYzVjNjMyZjQ0YWMzODc2MzI1NWM0NmU3NThhMyA9ICQoJzxkaXYgaWQ9Imh0bWxfMzAyZWM1YzYzMmY0NGFjMzg3NjMyNTVjNDZlNzU4YTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0LiBKYW1lcyBUb3duLCBDYWJiYWdldG93biBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzMxMjkyMjhkZTJjZTQ5MzNhMzdiNmM4ODFkYTJlOTNlLnNldENvbnRlbnQoaHRtbF8zMDJlYzVjNjMyZjQ0YWMzODc2MzI1NWM0NmU3NThhMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81ZTcwZjQyZDhjYWE0MzY5ODZkZDE4YzQ2NmNmZDQ2Mi5iaW5kUG9wdXAocG9wdXBfMzEyOTIyOGRlMmNlNDkzM2EzN2I2Yzg4MWRhMmU5M2UpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDVkOTVlMTUyNTU1NDEzZTlhMjk1ZWQ4YmZiNjM4MTkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjU4NTk5LC03OS4zODMxNTk5MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yNzdiNTBiY2MyODQ0OTBiOWM4ZDdmZDhmYjYxNGUxNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80NWY1ZWJlNWQ2MGU0YmFhYTE3MjliYjk5YzU4NmM1OCA9ICQoJzxkaXYgaWQ9Imh0bWxfNDVmNWViZTVkNjBlNGJhYWExNzI5YmI5OWM1ODZjNTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNodXJjaCBhbmQgV2VsbGVzbGV5IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMjc3YjUwYmNjMjg0NDkwYjljOGQ3ZmQ4ZmI2MTRlMTUuc2V0Q29udGVudChodG1sXzQ1ZjVlYmU1ZDYwZTRiYWFhMTcyOWJiOTljNTg2YzU4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q1ZDk1ZTE1MjU1NTQxM2U5YTI5NWVkOGJmYjYzODE5LmJpbmRQb3B1cChwb3B1cF8yNzdiNTBiY2MyODQ0OTBiOWM4ZDdmZDhmYjYxNGUxNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xZjI1NDY0YTYyN2U0YjhlYjMwMGNhNWVjMTA3NGQyMyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1NDI1OTksLTc5LjM2MDYzNTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYzVhZTRiMWY2OTU4NDA0ZmEzZjQ1MzA1MmY1M2E1MDUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjJlY2M1ODI2MmE4NGIyYmExNDI2MzUzMzI0MWMzYjIgPSAkKCc8ZGl2IGlkPSJodG1sXzYyZWNjNTgyNjJhODRiMmJhMTQyNjM1MzMyNDFjM2IyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SZWdlbnQgUGFyaywgSGFyYm91cmZyb250IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzVhZTRiMWY2OTU4NDA0ZmEzZjQ1MzA1MmY1M2E1MDUuc2V0Q29udGVudChodG1sXzYyZWNjNTgyNjJhODRiMmJhMTQyNjM1MzMyNDFjM2IyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzFmMjU0NjRhNjI3ZTRiOGViMzAwY2E1ZWMxMDc0ZDIzLmJpbmRQb3B1cChwb3B1cF9jNWFlNGIxZjY5NTg0MDRmYTNmNDUzMDUyZjUzYTUwNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zOTVhMWRmODdkODI0NDgwYTRiYTlkMGQ4Nzc5MjIwYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1NzE2MTgsLTc5LjM3ODkzNzA5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ0OGNkNTQwNzdkZTQxYTY5NDE0NjRkNDNiZWY3OGMwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzYzZmU4NmZiNjE5ZTQ3NTM5NWZmNTcwMzkyMzJlNzQzID0gJCgnPGRpdiBpZD0iaHRtbF82M2ZlODZmYjYxOWU0NzUzOTVmZjU3MDM5MjMyZTc0MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2FyZGVuIERpc3RyaWN0LCBSeWVyc29uIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDQ4Y2Q1NDA3N2RlNDFhNjk0MTQ2NGQ0M2JlZjc4YzAuc2V0Q29udGVudChodG1sXzYzZmU4NmZiNjE5ZTQ3NTM5NWZmNTcwMzkyMzJlNzQzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzM5NWExZGY4N2Q4MjQ0ODBhNGJhOWQwZDg3NzkyMjBhLmJpbmRQb3B1cChwb3B1cF80NDhjZDU0MDc3ZGU0MWE2OTQxNDY0ZDQzYmVmNzhjMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wZTMyMGFlZGM3ZDA0MGRjOTMzMTc2NGNjMjc3NzcyNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1MTQ5MzksLTc5LjM3NTQxNzldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWEzNjVmMjk0YzUxNDM1OTgxNDJhZDk5NDY2NjhjNTQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNzRiYTAxOWM2ODhlNGU2OGI1YmY1NzNlNDE4ODEyZGYgPSAkKCc8ZGl2IGlkPSJodG1sXzc0YmEwMTljNjg4ZTRlNjhiNWJmNTczZTQxODgxMmRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdC4gSmFtZXMgVG93biBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzVhMzY1ZjI5NGM1MTQzNTk4MTQyYWQ5OTQ2NjY4YzU0LnNldENvbnRlbnQoaHRtbF83NGJhMDE5YzY4OGU0ZTY4YjViZjU3M2U0MTg4MTJkZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wZTMyMGFlZGM3ZDA0MGRjOTMzMTc2NGNjMjc3NzcyNy5iaW5kUG9wdXAocG9wdXBfNWEzNjVmMjk0YzUxNDM1OTgxNDJhZDk5NDY2NjhjNTQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfN2ExN2RmZjY4OWE2NGJhN2JiYjE0ZTQ3MDM1ZGMxMjAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDQ3NzA3OTk5OTk5OTYsLTc5LjM3MzMwNjRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDJlZWZmYjhiMzY1NDdmYjkxM2YxM2JhYjU4YzI0MTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMDM2MThiNTFhNmIwNDk3YmIxMTI5MTQzNjJjZmVkMWYgPSAkKCc8ZGl2IGlkPSJodG1sXzAzNjE4YjUxYTZiMDQ5N2JiMTEyOTE0MzYyY2ZlZDFmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CZXJjenkgUGFyayBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAyZWVmZmI4YjM2NTQ3ZmI5MTNmMTNiYWI1OGMyNDE2LnNldENvbnRlbnQoaHRtbF8wMzYxOGI1MWE2YjA0OTdiYjExMjkxNDM2MmNmZWQxZik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl83YTE3ZGZmNjg5YTY0YmE3YmJiMTRlNDcwMzVkYzEyMC5iaW5kUG9wdXAocG9wdXBfMDJlZWZmYjhiMzY1NDdmYjkxM2YxM2JhYjU4YzI0MTYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTc5ZjAwMzhiODhhNDVmMThkMmU3MTkwNDJlZTVkODYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTc5NTI0LC03OS4zODczODI2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzUwMDY1MWI2NzhkNjRmZmQ4ZTgzZTQ2ZGFhNmRhM2EyID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2UyY2ZlMmI2N2ExMzQxZDVhOWY1Nzg5YjQyN2NkMTlkID0gJCgnPGRpdiBpZD0iaHRtbF9lMmNmZTJiNjdhMTM0MWQ1YTlmNTc4OWI0MjdjZDE5ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2VudHJhbCBCYXkgU3RyZWV0IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTAwNjUxYjY3OGQ2NGZmZDhlODNlNDZkYWE2ZGEzYTIuc2V0Q29udGVudChodG1sX2UyY2ZlMmI2N2ExMzQxZDVhOWY1Nzg5YjQyN2NkMTlkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU3OWYwMDM4Yjg4YTQ1ZjE4ZDJlNzE5MDQyZWU1ZDg2LmJpbmRQb3B1cChwb3B1cF81MDA2NTFiNjc4ZDY0ZmZkOGU4M2U0NmRhYTZkYTNhMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85NGEyOWI0ODZmNDE0NjYyYWE3YWVmZTlkMjMzZDUxYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1MDU3MTIwMDAwMDAxLC03OS4zODQ1Njc1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzNmOWNkMmEyMDg3MzQ2ZDZiMTBkMWZjNzllNjRjYzFmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2MyZGQ1MTFmNDBhNDQ1NWFhNzk0N2Y4ZTRiZjU3YzJlID0gJCgnPGRpdiBpZD0iaHRtbF9jMmRkNTExZjQwYTQ0NTVhYTc5NDdmOGU0YmY1N2MyZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UmljaG1vbmQsIEFkZWxhaWRlLCBLaW5nIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfM2Y5Y2QyYTIwODczNDZkNmIxMGQxZmM3OWU2NGNjMWYuc2V0Q29udGVudChodG1sX2MyZGQ1MTFmNDBhNDQ1NWFhNzk0N2Y4ZTRiZjU3YzJlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk0YTI5YjQ4NmY0MTQ2NjJhYTdhZWZlOWQyMzNkNTFhLmJpbmRQb3B1cChwb3B1cF8zZjljZDJhMjA4NzM0NmQ2YjEwZDFmYzc5ZTY0Y2MxZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hZmJjMzlmYjVkNjQ0YmI1OGNkOWUzMjQ4OWRiMmYyNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0MDgxNTcsLTc5LjM4MTc1MjI5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzZiYWMwMGI3MGE2YTQ0NzI5N2QyZjQ4YjllY2FmMDdhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2Q5YmY1NGQ3YTRiNzQwZDI5MmY2MmVjYmU4MWM2YTlkID0gJCgnPGRpdiBpZD0iaHRtbF9kOWJmNTRkN2E0Yjc0MGQyOTJmNjJlY2JlODFjNmE5ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SGFyYm91cmZyb250IEVhc3QsIFVuaW9uIFN0YXRpb24sIFRvcm9udG8gSXNsYW5kcyBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzZiYWMwMGI3MGE2YTQ0NzI5N2QyZjQ4YjllY2FmMDdhLnNldENvbnRlbnQoaHRtbF9kOWJmNTRkN2E0Yjc0MGQyOTJmNjJlY2JlODFjNmE5ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hZmJjMzlmYjVkNjQ0YmI1OGNkOWUzMjQ4OWRiMmYyNS5iaW5kUG9wdXAocG9wdXBfNmJhYzAwYjcwYTZhNDQ3Mjk3ZDJmNDhiOWVjYWYwN2EpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzk1NzA0ODA3YmFkNDUzOWI5NTI0NDE5YmVmODNlZDQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDcxNzY4LC03OS4zODE1NzY0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9mYTkxYmE3ZGMwMzE0NWMzOTFjNGJmNmRkNzg5ZDBiMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mZjQ5Njk5ZTg4YmU0MWMxODU1MDdiMGEzMDQxYzVjNyA9ICQoJzxkaXYgaWQ9Imh0bWxfZmY0OTY5OWU4OGJlNDFjMTg1NTA3YjBhMzA0MWM1YzciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRvcm9udG8gRG9taW5pb24gQ2VudHJlLCBEZXNpZ24gRXhjaGFuZ2UgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mYTkxYmE3ZGMwMzE0NWMzOTFjNGJmNmRkNzg5ZDBiMC5zZXRDb250ZW50KGh0bWxfZmY0OTY5OWU4OGJlNDFjMTg1NTA3YjBhMzA0MWM1YzcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMzk1NzA0ODA3YmFkNDUzOWI5NTI0NDE5YmVmODNlZDQuYmluZFBvcHVwKHBvcHVwX2ZhOTFiYTdkYzAzMTQ1YzM5MWM0YmY2ZGQ3ODlkMGIwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2JkNDU3NmViZTg0ODQ5MGI5ZDIwYjQ3N2MxYjU2MzY1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4MTk4NSwtNzkuMzc5ODE2OTAwMDAwMDFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZThhNjAzMGY5ZjNmNGE2ZmI4NDI4NjE0ZTU0MTRiZGQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZmQ1YjlmM2UxY2NkNGU0ZDgwYTczYTMxYWE0NmU0ZGUgPSAkKCc8ZGl2IGlkPSJodG1sX2ZkNWI5ZjNlMWNjZDRlNGQ4MGE3M2EzMWFhNDZlNGRlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Db21tZXJjZSBDb3VydCwgVmljdG9yaWEgSG90ZWwgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9lOGE2MDMwZjlmM2Y0YTZmYjg0Mjg2MTRlNTQxNGJkZC5zZXRDb250ZW50KGh0bWxfZmQ1YjlmM2UxY2NkNGU0ZDgwYTczYTMxYWE0NmU0ZGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYmQ0NTc2ZWJlODQ4NDkwYjlkMjBiNDc3YzFiNTYzNjUuYmluZFBvcHVwKHBvcHVwX2U4YTYwMzBmOWYzZjRhNmZiODQyODYxNGU1NDE0YmRkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2FkN2E2NDY2YzBlYzRmMWY5OGE0YjRhYzA1NThjMDllID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzExNjk0OCwtNzkuNDE2OTM1NTk5OTk5OTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmYjM2MCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZmIzNjAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNjYzYTAwNjZiNmQ0NGMxMWI5MGQ2Zjc1M2Y3MDY2MTIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYmFjOTQ3MjM3NWNlNDM0MmI2MTdmYzAzYTNmMGMwMDggPSAkKCc8ZGl2IGlkPSJodG1sX2JhYzk0NzIzNzVjZTQzNDJiNjE3ZmMwM2EzZjBjMDA4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NlbGF3biBDbHVzdGVyIDQ8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzY2M2EwMDY2YjZkNDRjMTFiOTBkNmY3NTNmNzA2NjEyLnNldENvbnRlbnQoaHRtbF9iYWM5NDcyMzc1Y2U0MzQyYjYxN2ZjMDNhM2YwYzAwOCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hZDdhNjQ2NmMwZWM0ZjFmOThhNGI0YWMwNTU4YzA5ZS5iaW5kUG9wdXAocG9wdXBfNjYzYTAwNjZiNmQ0NGMxMWI5MGQ2Zjc1M2Y3MDY2MTIpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNGNiOGUyNjhiMmIyNDBjOWI5ODlhYTFjZWU5ZjRkNGQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42OTY5NDc2LC03OS40MTEzMDcyMDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjODBmZmI0IiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzgwZmZiNCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iMzYzZGU1ZGVhODA0YTM1OGI5MDlmZDZlMTYzNDlkNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8wNDJjZmFmNTA1ODE0YWY3ODllODhhNGIyMjAyOTFhNCA9ICQoJzxkaXYgaWQ9Imh0bWxfMDQyY2ZhZjUwNTgxNGFmNzg5ZTg4YTRiMjIwMjkxYTQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZvcmVzdCBIaWxsIE5vcnRoICZhbXA7IFdlc3QsIEZvcmVzdCBIaWxsIFJvYWQgUGFyayBDbHVzdGVyIDM8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2IzNjNkZTVkZWE4MDRhMzU4YjkwOWZkNmUxNjM0OWQ1LnNldENvbnRlbnQoaHRtbF8wNDJjZmFmNTA1ODE0YWY3ODllODhhNGIyMjAyOTFhNCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl80Y2I4ZTI2OGIyYjI0MGM5Yjk4OWFhMWNlZTlmNGQ0ZC5iaW5kUG9wdXAocG9wdXBfYjM2M2RlNWRlYTgwNGEzNThiOTA5ZmQ2ZTE2MzQ5ZDUpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMjIwN2U1ZTVkMmYwNGI5NDgwYzQxYzU0OGM4NGRhN2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NzI3MDk3LC03OS40MDU2Nzg0MDAwMDAwMV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xODhiMGJjNzBlYTA0NGVmYTJkZDBmOWFiYTFhZWM2NiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8xN2NlMGZkYWY2OGY0ODVlYjE3ZjQwZGZkZDA0ZDM5NyA9ICQoJzxkaXYgaWQ9Imh0bWxfMTdjZTBmZGFmNjhmNDg1ZWIxN2Y0MGRmZGQwNGQzOTciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBBbm5leCwgTm9ydGggTWlkdG93biwgWW9ya3ZpbGxlIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTg4YjBiYzcwZWEwNDRlZmEyZGQwZjlhYmExYWVjNjYuc2V0Q29udGVudChodG1sXzE3Y2UwZmRhZjY4ZjQ4NWViMTdmNDBkZmRkMDRkMzk3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzIyMDdlNWU1ZDJmMDRiOTQ4MGM0MWM1NDhjODRkYTdlLmJpbmRQb3B1cChwb3B1cF8xODhiMGJjNzBlYTA0NGVmYTJkZDBmOWFiYTFhZWM2Nik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lMjg4YjFlNmMyY2M0ZWY5YmE3YWQyOTM0OTc0MWJiOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2MjY5NTYsLTc5LjQwMDA0OTNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZDYyY2U3MzBiMTg0NDhmNWIyMDgwMmVkM2VmYWIxMmIgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjFkYTQ1MjMxOGNkNDM2ZDliZTRiZTI4YmFjNGI1YmIgPSAkKCc8ZGl2IGlkPSJodG1sXzYxZGE0NTIzMThjZDQzNmQ5YmU0YmUyOGJhYzRiNWJiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Vbml2ZXJzaXR5IG9mIFRvcm9udG8sIEhhcmJvcmQgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kNjJjZTczMGIxODQ0OGY1YjIwODAyZWQzZWZhYjEyYi5zZXRDb250ZW50KGh0bWxfNjFkYTQ1MjMxOGNkNDM2ZDliZTRiZTI4YmFjNGI1YmIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZTI4OGIxZTZjMmNjNGVmOWJhN2FkMjkzNDk3NDFiYjguYmluZFBvcHVwKHBvcHVwX2Q2MmNlNzMwYjE4NDQ4ZjViMjA4MDJlZDNlZmFiMTJiKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2E2NmVlNjM5NDZkYTRjYzY5NmEzYWY0MmY0MWM4ZjI1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUzMjA1NywtNzkuNDAwMDQ5M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85YjkzNGNmZDdjOGQ0ZDJhOWM3ZTI4NzRhNDZmMzFjNSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81MmQ2YzcyMjQyOGY0NDIxYTA2OGY5M2VhY2E2MjExYyA9ICQoJzxkaXYgaWQ9Imh0bWxfNTJkNmM3MjI0MjhmNDQyMWEwNjhmOTNlYWNhNjIxMWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktlbnNpbmd0b24gTWFya2V0LCBDaGluYXRvd24sIEdyYW5nZSBQYXJrIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOWI5MzRjZmQ3YzhkNGQyYTljN2UyODc0YTQ2ZjMxYzUuc2V0Q29udGVudChodG1sXzUyZDZjNzIyNDI4ZjQ0MjFhMDY4ZjkzZWFjYTYyMTFjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2E2NmVlNjM5NDZkYTRjYzY5NmEzYWY0MmY0MWM4ZjI1LmJpbmRQb3B1cChwb3B1cF85YjkzNGNmZDdjOGQ0ZDJhOWM3ZTI4NzRhNDZmMzFjNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82YTQ2NWU2MmZiODk0MTJlYTEyNTE5NWJmNGNiNWYxYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjYyODk0NjcsLTc5LjM5NDQxOTldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTZlZGZkZTk4NWI1NDU0YmFkNTA2YmViN2Q3ZjM1Y2EgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMDFjM2MzN2E4YzQ5NDRmNGIxY2I4ZmJhNzcxMWI2YzcgPSAkKCc8ZGl2IGlkPSJodG1sXzAxYzNjMzdhOGM0OTQ0ZjRiMWNiOGZiYTc3MTFiNmM3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DTiBUb3dlciwgS2luZyBhbmQgU3BhZGluYSwgUmFpbHdheSBMYW5kcywgSGFyYm91cmZyb250IFdlc3QsIEJhdGh1cnN0IFF1YXksIFNvdXRoIE5pYWdhcmEsIElzbGFuZCBhaXJwb3J0IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTZlZGZkZTk4NWI1NDU0YmFkNTA2YmViN2Q3ZjM1Y2Euc2V0Q29udGVudChodG1sXzAxYzNjMzdhOGM0OTQ0ZjRiMWNiOGZiYTc3MTFiNmM3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZhNDY1ZTYyZmI4OTQxMmVhMTI1MTk1YmY0Y2I1ZjFjLmJpbmRQb3B1cChwb3B1cF81NmVkZmRlOTg1YjU0NTRiYWQ1MDZiZWI3ZDdmMzVjYSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82ZTVhZGI0NWIyMjY0YWQ5OWUyNDA3OGFlOTQyYTg5ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0NjQzNTIsLTc5LjM3NDg0NTk5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzkzNTNhY2MxM2RiYjQ5NDg5MjgyY2YzNTM1NjNhM2IzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzljYThhYWZmMzZiMTQyM2FhYWFlNzkyYzYxZTdmMThhID0gJCgnPGRpdiBpZD0iaHRtbF85Y2E4YWFmZjM2YjE0MjNhYWFhZTc5MmM2MWU3ZjE4YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3RuIEEgUE8gQm94ZXMgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85MzUzYWNjMTNkYmI0OTQ4OTI4MmNmMzUzNTYzYTNiMy5zZXRDb250ZW50KGh0bWxfOWNhOGFhZmYzNmIxNDIzYWFhYWU3OTJjNjFlN2YxOGEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNmU1YWRiNDViMjI2NGFkOTllMjQwNzhhZTk0MmE4OWYuYmluZFBvcHVwKHBvcHVwXzkzNTNhY2MxM2RiYjQ5NDg5MjgyY2YzNTM1NjNhM2IzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y4ZTg4YjZkNjk1MjQ5MjQ5YjdkYmRlMjI5YmM0NWE0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4NDI5MiwtNzkuMzgyMjgwMl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84NDUxMzI2NTcwMmQ0NzIyOWEyZmEyOTlhMjJhMDg1YSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83MjViYTA5NjAyOTY0MDhlYjNjYzY2Yzc1NWFiMGQzMyA9ICQoJzxkaXYgaWQ9Imh0bWxfNzI1YmEwOTYwMjk2NDA4ZWIzY2M2NmM3NTVhYjBkMzMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZpcnN0IENhbmFkaWFuIFBsYWNlLCBVbmRlcmdyb3VuZCBjaXR5IENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODQ1MTMyNjU3MDJkNDcyMjlhMmZhMjk5YTIyYTA4NWEuc2V0Q29udGVudChodG1sXzcyNWJhMDk2MDI5NjQwOGViM2NjNjZjNzU1YWIwZDMzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Y4ZTg4YjZkNjk1MjQ5MjQ5YjdkYmRlMjI5YmM0NWE0LmJpbmRQb3B1cChwb3B1cF84NDUxMzI2NTcwMmQ0NzIyOWEyZmEyOTlhMjJhMDg1YSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84NDg2ZDRjMGM4ZWQ0ODY5OWEzYTVkODAwY2Q1MTk5OSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2OTU0MiwtNzkuNDIyNTYzN10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82MzFjYzZlZjAxMzk0NDdiOTJlZDRjNTFlOWVkMWRlMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yMmFhYjg2ODQ5MGY0YmQ3OWYyYjA2MTQxMmU5NzRhNSA9ICQoJzxkaXYgaWQ9Imh0bWxfMjJhYWI4Njg0OTBmNGJkNzlmMmIwNjE0MTJlOTc0YTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNocmlzdGllIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNjMxY2M2ZWYwMTM5NDQ3YjkyZWQ0YzUxZTllZDFkZTMuc2V0Q29udGVudChodG1sXzIyYWFiODY4NDkwZjRiZDc5ZjJiMDYxNDEyZTk3NGE1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzg0ODZkNGMwYzhlZDQ4Njk5YTNhNWQ4MDBjZDUxOTk5LmJpbmRQb3B1cChwb3B1cF82MzFjYzZlZjAxMzk0NDdiOTJlZDRjNTFlOWVkMWRlMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lNmI3ODc1OTkwMTU0ZWFkYTAzNWYyYmNiZjY4Nzk1NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2OTAwNTEwMDAwMDAxLC03OS40NDIyNTkzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzhkNjVkZGQ5MWI1NDQ0YzQ4YjE1ODgwYmNkNGM3MzkzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzFiMzg0MTI4Yzc1ZTQ3ODliOGJhNWYwY2Y4ZWVlNjBlID0gJCgnPGRpdiBpZD0iaHRtbF8xYjM4NDEyOGM3NWU0Nzg5YjhiYTVmMGNmOGVlZTYwZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RHVmZmVyaW4sIERvdmVyY291cnQgVmlsbGFnZSBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzhkNjVkZGQ5MWI1NDQ0YzQ4YjE1ODgwYmNkNGM3MzkzLnNldENvbnRlbnQoaHRtbF8xYjM4NDEyOGM3NWU0Nzg5YjhiYTVmMGNmOGVlZTYwZSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9lNmI3ODc1OTkwMTU0ZWFkYTAzNWYyYmNiZjY4Nzk1NC5iaW5kUG9wdXAocG9wdXBfOGQ2NWRkZDkxYjU0NDRjNDhiMTU4ODBiY2Q0YzczOTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMmVlYjI0MTYxN2E1NDkyZjllZTBhN2RmYzNiZDQyZTQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDc5MjY3MDAwMDAwMDYsLTc5LjQxOTc0OTddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDI1ZTRhYjRjYzRhNDM1MzllZWU5MTIwOTFlNTAxMDQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNTE1ZDQ3MGU5MTliNDYwMTgwNmU0YmQ0ZGVlZmY2YWYgPSAkKCc8ZGl2IGlkPSJodG1sXzUxNWQ0NzBlOTE5YjQ2MDE4MDZlNGJkNGRlZWZmNmFmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MaXR0bGUgUG9ydHVnYWwsIFRyaW5pdHkgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wMjVlNGFiNGNjNGE0MzUzOWVlZTkxMjA5MWU1MDEwNC5zZXRDb250ZW50KGh0bWxfNTE1ZDQ3MGU5MTliNDYwMTgwNmU0YmQ0ZGVlZmY2YWYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmVlYjI0MTYxN2E1NDkyZjllZTBhN2RmYzNiZDQyZTQuYmluZFBvcHVwKHBvcHVwXzAyNWU0YWI0Y2M0YTQzNTM5ZWVlOTEyMDkxZTUwMTA0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk4ZTc0OTIyYmM4MTRiMTA5YjEzNzkyYzcyOGViYjVjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjM2ODQ3MiwtNzkuNDI4MTkxNDAwMDAwMDJdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjg0MTA3NzJhODAyNGQ1ODhhODc1MjdiZTg2NGZlYzMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMDNlMjYxYmQ2YWRhNDAzNmE5NGMxN2I3YWVjMmIzYjAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjZjNTI1MTk3YTk5NGQ0Yzg3MmQxMjcyM2YyMDYxNjMgPSAkKCc8ZGl2IGlkPSJodG1sXzI2YzUyNTE5N2E5OTRkNGM4NzJkMTI3MjNmMjA2MTYzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ccm9ja3RvbiwgUGFya2RhbGUgVmlsbGFnZSwgRXhoaWJpdGlvbiBQbGFjZSBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAzZTI2MWJkNmFkYTQwMzZhOTRjMTdiN2FlYzJiM2IwLnNldENvbnRlbnQoaHRtbF8yNmM1MjUxOTdhOTk0ZDRjODcyZDEyNzIzZjIwNjE2Myk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl85OGU3NDkyMmJjODE0YjEwOWIxMzc5MmM3MjhlYmI1Yy5iaW5kUG9wdXAocG9wdXBfMDNlMjYxYmQ2YWRhNDAzNmE5NGMxN2I3YWVjMmIzYjApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDU0MGUzOGU3ZjU3NDcyYzhlMGUzY2QzNTEwN2VlMzUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjE2MDgzLC03OS40NjQ3NjMyOTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83ZDcyYjYzNTIyM2U0OWQwOTEwM2U3ZmEwMDkzYzRiOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iYmU5MWVkMDc4YTc0ZDhjYmM4ZjM5NGE4ZGMxOGFmZCA9ICQoJzxkaXYgaWQ9Imh0bWxfYmJlOTFlZDA3OGE3NGQ4Y2JjOGYzOTRhOGRjMThhZmQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhpZ2ggUGFyaywgVGhlIEp1bmN0aW9uIFNvdXRoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfN2Q3MmI2MzUyMjNlNDlkMDkxMDNlN2ZhMDA5M2M0Yjguc2V0Q29udGVudChodG1sX2JiZTkxZWQwNzhhNzRkOGNiYzhmMzk0YThkYzE4YWZkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2Q1NDBlMzhlN2Y1NzQ3MmM4ZTBlM2NkMzUxMDdlZTM1LmJpbmRQb3B1cChwb3B1cF83ZDcyYjYzNTIyM2U0OWQwOTEwM2U3ZmEwMDkzYzRiOCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kZjBlMWY4ZDc1MTA0MDYxOTk0YjMyODRlOTk4ZDJkOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODk1OTcsLTc5LjQ1NjMyNV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wMjkxZDU4MWY3OGU0MjU0YmRjZGYwYzcwYmExM2IxYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84ZmFkZjdlNjk2NmQ0YTk4OWY5YjZmNGNlYzNmNGJkYSA9ICQoJzxkaXYgaWQ9Imh0bWxfOGZhZGY3ZTY5NjZkNGE5ODlmOWI2ZjRjZWMzZjRiZGEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBhcmtkYWxlLCBSb25jZXN2YWxsZXMgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wMjkxZDU4MWY3OGU0MjU0YmRjZGYwYzcwYmExM2IxYy5zZXRDb250ZW50KGh0bWxfOGZhZGY3ZTY5NjZkNGE5ODlmOWI2ZjRjZWMzZjRiZGEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZGYwZTFmOGQ3NTEwNDA2MTk5NGIzMjg0ZTk5OGQyZDguYmluZFBvcHVwKHBvcHVwXzAyOTFkNTgxZjc4ZTQyNTRiZGNkZjBjNzBiYTEzYjFjKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzA0MmMzYmViZmQ2MjRmYzQ4Mjg1YmVhMGEwNzI0YTNkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUxNTcwNiwtNzkuNDg0NDQ5OV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9iODQxMDc3MmE4MDI0ZDU4OGE4NzUyN2JlODY0ZmVjMyk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85MDVjZGEwZWMzZGQ0NWJlOWEwMjAyYzJlYmVlZDZiZiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83ZmRjODczNjcwYWY0NWRkYWZlNTBlODY4N2RiZDlmZCA9ICQoJzxkaXYgaWQ9Imh0bWxfN2ZkYzg3MzY3MGFmNDVkZGFmZTUwZTg2ODdkYmQ5ZmQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJ1bm55bWVkZSwgU3dhbnNlYSBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzkwNWNkYTBlYzNkZDQ1YmU5YTAyMDJjMmViZWVkNmJmLnNldENvbnRlbnQoaHRtbF83ZmRjODczNjcwYWY0NWRkYWZlNTBlODY4N2RiZDlmZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8wNDJjM2JlYmZkNjI0ZmM0ODI4NWJlYTBhMDcyNGEzZC5iaW5kUG9wdXAocG9wdXBfOTA1Y2RhMGVjM2RkNDViZTlhMDIwMmMyZWJlZWQ2YmYpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzRmMzI5Y2FlZWI3NGRiOThhZDczYzM3NmZjZGJlOGQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjIzMDE1LC03OS4zODk0OTM4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzJkMGU5YjU3YmFhMzQwYjBhZjAyODJlOGM0NzUzNWYxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzk2YmM3ZDhmZTRkMzQzOTJhY2U1ZTMyNjEyNjg4NWZjID0gJCgnPGRpdiBpZD0iaHRtbF85NmJjN2Q4ZmU0ZDM0MzkyYWNlNWUzMjYxMjY4ODVmYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UXVlZW4mIzM5O3MgUGFyaywgT250YXJpbyBQcm92aW5jaWFsIEdvdmVybm1lbnQgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8yZDBlOWI1N2JhYTM0MGIwYWYwMjgyZThjNDc1MzVmMS5zZXRDb250ZW50KGh0bWxfOTZiYzdkOGZlNGQzNDM5MmFjZTVlMzI2MTI2ODg1ZmMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYzRmMzI5Y2FlZWI3NGRiOThhZDczYzM3NmZjZGJlOGQuYmluZFBvcHVwKHBvcHVwXzJkMGU5YjU3YmFhMzQwYjBhZjAyODJlOGM0NzUzNWYxKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzBkM2Y5MmJhNjk0OTRiOTM4Yzg5MzI4NDAwMjRmNDk4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyNzQzOSwtNzkuMzIxNTU4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2I4NDEwNzcyYTgwMjRkNTg4YTg3NTI3YmU4NjRmZWMzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzViZjZlMTZkZWRhZTRkNGM4MmQ3YzViYzNiNWYxZmE3ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzg1N2M3MGE0YjI1NTQwMGI5ZDBlMzIzMjI3YzAwMmRiID0gJCgnPGRpdiBpZD0iaHRtbF84NTdjNzBhNGIyNTU0MDBiOWQwZTMyMzIyN2MwMDJkYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnVzaW5lc3MgcmVwbHkgbWFpbCBQcm9jZXNzaW5nIENlbnRyZSwgU291dGggQ2VudHJhbCBMZXR0ZXIgUHJvY2Vzc2luZyBQbGFudCBUb3JvbnRvIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWJmNmUxNmRlZGFlNGQ0YzgyZDdjNWJjM2I1ZjFmYTcuc2V0Q29udGVudChodG1sXzg1N2M3MGE0YjI1NTQwMGI5ZDBlMzIzMjI3YzAwMmRiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBkM2Y5MmJhNjk0OTRiOTM4Yzg5MzI4NDAwMjRmNDk4LmJpbmRQb3B1cChwb3B1cF81YmY2ZTE2ZGVkYWU0ZDRjODJkN2M1YmMzYjVmMWZhNyk7CgogICAgICAgICAgICAKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



### **4.5 Exam Clusters**
#### Cluster 1


```python
toronto_merged.loc[toronto_merged['Cluster Labels'] == 0, toronto_merged.columns[[1] + list(range(5, toronto_merged.shape[1]))]]
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
      <th>Borough</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East Toronto</td>
      <td>0</td>
      <td>Trail</td>
      <td>Health Food Store</td>
      <td>Pub</td>
      <td>Doner Restaurant</td>
      <td>Dessert Shop</td>
      <td>Diner</td>
      <td>Discount Store</td>
      <td>Distribution Center</td>
      <td>Dog Run</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East Toronto</td>
      <td>0</td>
      <td>Greek Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Bookstore</td>
      <td>Ice Cream Shop</td>
      <td>Furniture / Home Store</td>
      <td>Restaurant</td>
      <td>Caribbean Restaurant</td>
      <td>Pub</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>2</th>
      <td>East Toronto</td>
      <td>0</td>
      <td>Fast Food Restaurant</td>
      <td>Park</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Pet Store</td>
      <td>Fish &amp; Chips Shop</td>
      <td>Ice Cream Shop</td>
      <td>Sushi Restaurant</td>
      <td>Brewery</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>East Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>American Restaurant</td>
      <td>Bakery</td>
      <td>Brewery</td>
      <td>Gastropub</td>
      <td>Gym / Fitness Center</td>
      <td>Fish Market</td>
      <td>Pet Store</td>
      <td>Park</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Central Toronto</td>
      <td>0</td>
      <td>Gym / Fitness Center</td>
      <td>Hotel</td>
      <td>Pizza Place</td>
      <td>Department Store</td>
      <td>Sandwich Place</td>
      <td>Breakfast Spot</td>
      <td>Food &amp; Drink Shop</td>
      <td>Park</td>
      <td>Gastropub</td>
      <td>General Travel</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Central Toronto</td>
      <td>0</td>
      <td>Clothing Store</td>
      <td>Coffee Shop</td>
      <td>Gym / Fitness Center</td>
      <td>Sporting Goods Shop</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Diner</td>
      <td>Fast Food Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Park</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Central Toronto</td>
      <td>0</td>
      <td>Pizza Place</td>
      <td>Dessert Shop</td>
      <td>Sandwich Place</td>
      <td>Sushi Restaurant</td>
      <td>Coffee Shop</td>
      <td>Gym</td>
      <td>Italian Restaurant</td>
      <td>Café</td>
      <td>Park</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Central Toronto</td>
      <td>0</td>
      <td>Pub</td>
      <td>Coffee Shop</td>
      <td>Fried Chicken Joint</td>
      <td>Supermarket</td>
      <td>Vietnamese Restaurant</td>
      <td>Light Rail Station</td>
      <td>Sushi Restaurant</td>
      <td>Liquor Store</td>
      <td>Bagel Shop</td>
      <td>American Restaurant</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Park</td>
      <td>Bakery</td>
      <td>Italian Restaurant</td>
      <td>Chinese Restaurant</td>
      <td>Pizza Place</td>
      <td>Restaurant</td>
      <td>Pub</td>
      <td>General Entertainment</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Sushi Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Restaurant</td>
      <td>Gay Bar</td>
      <td>Yoga Studio</td>
      <td>Smoke Shop</td>
      <td>Bubble Tea Shop</td>
      <td>Pub</td>
      <td>Hotel</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Pub</td>
      <td>Bakery</td>
      <td>Park</td>
      <td>Breakfast Spot</td>
      <td>Café</td>
      <td>Theater</td>
      <td>Gym / Fitness Center</td>
      <td>Event Space</td>
      <td>Restaurant</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Clothing Store</td>
      <td>Coffee Shop</td>
      <td>Bubble Tea Shop</td>
      <td>Café</td>
      <td>Middle Eastern Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Lingerie Store</td>
      <td>Tea Room</td>
      <td>Pizza Place</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>American Restaurant</td>
      <td>Gastropub</td>
      <td>Cocktail Bar</td>
      <td>Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Department Store</td>
      <td>Clothing Store</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Restaurant</td>
      <td>Bakery</td>
      <td>Beer Bar</td>
      <td>Seafood Restaurant</td>
      <td>Cheese Shop</td>
      <td>Café</td>
      <td>Irish Pub</td>
      <td>Pharmacy</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Italian Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Sandwich Place</td>
      <td>Café</td>
      <td>Salad Place</td>
      <td>Bar</td>
      <td>Bubble Tea Shop</td>
      <td>Burger Joint</td>
      <td>Department Store</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Restaurant</td>
      <td>Gym</td>
      <td>Thai Restaurant</td>
      <td>Hotel</td>
      <td>Deli / Bodega</td>
      <td>Bar</td>
      <td>Salad Place</td>
      <td>Pizza Place</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Aquarium</td>
      <td>Café</td>
      <td>Hotel</td>
      <td>Scenic Lookout</td>
      <td>Fried Chicken Joint</td>
      <td>Sporting Goods Shop</td>
      <td>Restaurant</td>
      <td>Brewery</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Hotel</td>
      <td>Café</td>
      <td>Restaurant</td>
      <td>Salad Place</td>
      <td>Seafood Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>American Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Deli / Bodega</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Restaurant</td>
      <td>Café</td>
      <td>Hotel</td>
      <td>American Restaurant</td>
      <td>Gym</td>
      <td>Seafood Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Cocktail Bar</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Central Toronto</td>
      <td>0</td>
      <td>Sandwich Place</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>BBQ Joint</td>
      <td>Middle Eastern Restaurant</td>
      <td>Pub</td>
      <td>Donut Shop</td>
      <td>Pizza Place</td>
      <td>Pharmacy</td>
      <td>History Museum</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Bookstore</td>
      <td>Bar</td>
      <td>Italian Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Restaurant</td>
      <td>Bakery</td>
      <td>Yoga Studio</td>
      <td>Beer Bar</td>
      <td>Beer Store</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Vietnamese Restaurant</td>
      <td>Coffee Shop</td>
      <td>Mexican Restaurant</td>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>Bakery</td>
      <td>Bar</td>
      <td>Grocery Store</td>
      <td>Pizza Place</td>
      <td>Dessert Shop</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Airport Lounge</td>
      <td>Airport Service</td>
      <td>Airport Terminal</td>
      <td>Coffee Shop</td>
      <td>Boutique</td>
      <td>Rental Car Location</td>
      <td>Plane</td>
      <td>Boat or Ferry</td>
      <td>Bar</td>
      <td>Harbor / Marina</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Seafood Restaurant</td>
      <td>Café</td>
      <td>Hotel</td>
      <td>Beer Bar</td>
      <td>Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Italian Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Cheese Shop</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Restaurant</td>
      <td>Hotel</td>
      <td>Gym</td>
      <td>American Restaurant</td>
      <td>Salad Place</td>
      <td>Steakhouse</td>
      <td>Asian Restaurant</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Grocery Store</td>
      <td>Café</td>
      <td>Park</td>
      <td>Candy Store</td>
      <td>Italian Restaurant</td>
      <td>Diner</td>
      <td>Nightclub</td>
      <td>Baby Store</td>
      <td>Coffee Shop</td>
      <td>Restaurant</td>
    </tr>
    <tr>
      <th>31</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Bakery</td>
      <td>Pharmacy</td>
      <td>Supermarket</td>
      <td>Bar</td>
      <td>Middle Eastern Restaurant</td>
      <td>Café</td>
      <td>Bank</td>
      <td>Pizza Place</td>
      <td>Park</td>
      <td>Music Venue</td>
    </tr>
    <tr>
      <th>32</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Bar</td>
      <td>Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>Café</td>
      <td>Vegetarian / Vegan Restaurant</td>
      <td>Men's Store</td>
      <td>Asian Restaurant</td>
      <td>Yoga Studio</td>
      <td>Cupcake Shop</td>
      <td>Record Shop</td>
    </tr>
    <tr>
      <th>33</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Breakfast Spot</td>
      <td>Coffee Shop</td>
      <td>Bakery</td>
      <td>Intersection</td>
      <td>Bar</td>
      <td>Italian Restaurant</td>
      <td>Restaurant</td>
      <td>Climbing Gym</td>
      <td>Furniture / Home Store</td>
    </tr>
    <tr>
      <th>34</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Café</td>
      <td>Mexican Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Park</td>
      <td>Furniture / Home Store</td>
      <td>Fast Food Restaurant</td>
      <td>Bookstore</td>
      <td>Flea Market</td>
      <td>Italian Restaurant</td>
      <td>Speakeasy</td>
    </tr>
    <tr>
      <th>35</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Breakfast Spot</td>
      <td>Gift Shop</td>
      <td>Dessert Shop</td>
      <td>Movie Theater</td>
      <td>Eastern European Restaurant</td>
      <td>Dog Run</td>
      <td>Italian Restaurant</td>
      <td>Bar</td>
      <td>Bank</td>
      <td>Restaurant</td>
    </tr>
    <tr>
      <th>36</th>
      <td>West Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Pizza Place</td>
      <td>Diner</td>
      <td>Italian Restaurant</td>
      <td>Pub</td>
      <td>Sushi Restaurant</td>
      <td>Spa</td>
      <td>Bank</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Downtown Toronto</td>
      <td>0</td>
      <td>Coffee Shop</td>
      <td>Sushi Restaurant</td>
      <td>Diner</td>
      <td>Arts &amp; Crafts Store</td>
      <td>Burrito Place</td>
      <td>Distribution Center</td>
      <td>Smoothie Shop</td>
      <td>Italian Restaurant</td>
      <td>Discount Store</td>
      <td>Beer Bar</td>
    </tr>
    <tr>
      <th>38</th>
      <td>East Toronto</td>
      <td>0</td>
      <td>Gym / Fitness Center</td>
      <td>Smoke Shop</td>
      <td>Auto Workshop</td>
      <td>Brewery</td>
      <td>Burrito Place</td>
      <td>Comic Shop</td>
      <td>Farmers Market</td>
      <td>Fast Food Restaurant</td>
      <td>Garden</td>
      <td>Garden Center</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 2


```python
toronto_merged.loc[toronto_merged['Cluster Labels'] == 1, toronto_merged.columns[[1] + list(range(5, toronto_merged.shape[1]))]]
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
      <th>Borough</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>Central Toronto</td>
      <td>1</td>
      <td>Lawyer</td>
      <td>Women's Store</td>
      <td>Fast Food Restaurant</td>
      <td>Falafel Restaurant</td>
      <td>Event Space</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 3


```python
toronto_merged.loc[toronto_merged['Cluster Labels'] == 2, toronto_merged.columns[[1] + list(range(5, toronto_merged.shape[1]))]]
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
      <th>Borough</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>Central Toronto</td>
      <td>2</td>
      <td>Park</td>
      <td>Swim School</td>
      <td>Bus Line</td>
      <td>Department Store</td>
      <td>Event Space</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 4


```python
toronto_merged.loc[toronto_merged['Cluster Labels'] == 3, toronto_merged.columns[[1] + list(range(5, toronto_merged.shape[1]))]]
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
      <th>Borough</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10</th>
      <td>Downtown Toronto</td>
      <td>3</td>
      <td>Park</td>
      <td>Playground</td>
      <td>Trail</td>
      <td>Dance Studio</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
      <td>Dog Run</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Central Toronto</td>
      <td>3</td>
      <td>Park</td>
      <td>Jewelry Store</td>
      <td>Trail</td>
      <td>Sushi Restaurant</td>
      <td>Deli / Bodega</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 5


```python
toronto_merged.loc[toronto_merged['Cluster Labels'] == 4, toronto_merged.columns[[1] + list(range(5, toronto_merged.shape[1]))]]
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
      <th>Borough</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>22</th>
      <td>Central Toronto</td>
      <td>4</td>
      <td>Home Service</td>
      <td>Music Venue</td>
      <td>Garden</td>
      <td>Women's Store</td>
      <td>Deli / Bodega</td>
      <td>Ethiopian Restaurant</td>
      <td>Electronics Store</td>
      <td>Eastern European Restaurant</td>
      <td>Donut Shop</td>
      <td>Doner Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
