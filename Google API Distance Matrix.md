

```python
import requests
from pandas.io.json import json_normalize
from flatten_json import *
import pandas as pd
import json
import googlemaps
import time

# URL of the Google Maps API
GOOGLE_MAPS_API_URL = 'https://maps.googleapis.com/maps/api/distancematrix/json'
GOOGLE_MAPS_DIR_URL = 'https://maps.googleapis.com/maps/api/directions/json'

# Google API Key File Path; KEEP PRIVATE
api_key_file = 'Google_API_Key.txt'

# Reading the API Key
mykey = open(api_key_file,"r").readline()
```

<div class="alert alert-block alert-info">
# Distance Matrix API - Duration & Distance

## Required Parameters
- `origins`, can supply multiple locations separated by pipe character, in the form of address, lat/lon, or a place ID
    - `'Bobcaygeon+ON|24+Sussex+Drive+Ottawa+ON'`, `'41.43206,-81.38992|-33.86748,151.20699'`, `'47906'`
- `destinations`, same format as `origins`
- `key`, API key
***
## Optional Parameters
- `mode`, *driving*, walking, bicycling, transit
- `language`
- `region`, region code, specified as a ccTLD two-character value
- `avoid`, tolls, highways, ferries, indoor
- `units`, *metric*, imperial (only relevaqnt for the text output)
- `arrival_time` `departure_time`, desired time of arrival for transit requests, in seconds since midnight, January 1, 1970 UTC (specify either but not both, should be specified in integers)
- `traffic_model`, *best_guess*, pessimistic, optimistic
- `transit_mode`, preferred modes of transit, bus, subway, train, tram, rail
- `transit_routing_preference`, less_walking, fewer_transfers
***
## Response Elements
Each row in the response corresponds to an origin, and each element within that row corresponds to a pairing of the origin with a specific destination.
- `status`, status of the request, useful for debugging. For codes other than `OK`, look at `error_message` field for more information.
    - #### Top-level Status Codes
        - `OK`, response contains a valid `result`
        - `INVALID_REQUEST`, provided request was invalid
        - `MAX_ELEMENTS_EXCEEDED`, product of origins and destinations exceeds the per-query limit
        - `OVER_QUERY_LIMIT`, too many requests within the allowed time period
        - `REQUEST_DENIED`, service denied
        - `UNKNOWN_ERROR`, server error
    - #### Element-level Status Codes
        - `OK`, response contains a valid `result`
        - `NOT_FOUND`, origin and/or destination pairing could not be geocoded
        - `ZERO_RESULTS`, no route found
        - `MAX_ROUTE_LENGTH_EXCEEDED`, requested route is too long to process
- `origin_addresses`, array of addresses returned by the API from your original request
- `destination_addresses`, array of addresses returned by the API from your original request
- `rows`, contains an array of elements, which in turn contain a `status`, `duration`, and `distance` element
- If available, response might also contain `fare` of the route
- When `departure_time` parameter is supplied for a `driving` mode request along with valid API key, `duration_in_traffic` is also returned based upon the `traffic_model` if traffic conditions are available


```python
# Sample Maps Duration and Distance using Distance Matrix API

params = {
    'units': 'imperial',
    'origins': '47906',
    'destinations': '20850',
    'key': mykey
}
try:
    req = requests.get(GOOGLE_MAPS_API_URL,params=params)
    res = req.json()
    print(json.dumps(res, sort_keys=True, indent=3))
except:
    import pdb; pdb.set_trace()
    print(res)

# json_normalize(res)
json_normalize(flatten_json(res))
```

    {
       "destination_addresses": [
          "Rockville, MD 20850, USA"
       ], 
       "origin_addresses": [
          "West Lafayette, IN 47906, USA"
       ], 
       "rows": [
          {
             "elements": [
                {
                   "distance": {
                      "text": "641 mi", 
                      "value": 1031794
                   }, 
                   "duration": {
                      "text": "9 hours 40 mins", 
                      "value": 34791
                   }, 
                   "status": "OK"
                }
             ]
          }
       ], 
       "status": "OK"
    }





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
      <th>destination_addresses_0</th>
      <th>origin_addresses_0</th>
      <th>rows_0_elements_0_distance_text</th>
      <th>rows_0_elements_0_distance_value</th>
      <th>rows_0_elements_0_duration_text</th>
      <th>rows_0_elements_0_duration_value</th>
      <th>rows_0_elements_0_status</th>
      <th>status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rockville, MD 20850, USA</td>
      <td>West Lafayette, IN 47906, USA</td>
      <td>641 mi</td>
      <td>1031794</td>
      <td>9 hours 40 mins</td>
      <td>34791</td>
      <td>OK</td>
      <td>OK</td>
    </tr>
  </tbody>
</table>
</div>




```python
params = {
    'origins': '2550+Yeager+Rd|New+York+City',
    'destinations': 'The+Bean+Chicago|NASA+Langley|Disneyworld+Florida|New+York+City',
    'mode': 'driving',
    'key': mykey
}

req1 = requests.get(GOOGLE_MAPS_API_URL, params=params)
res1 = req1.json()
res1
```




    {u'destination_addresses': [u'Chicago, IL 60601, USA',
      u'1 Nasa Dr, Hampton, VA 23666, USA',
      u'Walt Disney World Resort, Orlando, FL 32830, USA',
      u'New York, NY, USA'],
     u'origin_addresses': [u'2550 Yeager Rd, West Lafayette, IN 47906, USA',
      u'New York, NY, USA'],
     u'rows': [{u'elements': [{u'distance': {u'text': u'192 km', u'value': 192227},
         u'duration': {u'text': u'2 hours 7 mins', u'value': 7605},
         u'status': u'OK'},
        {u'distance': {u'text': u'1,235 km', u'value': 1234731},
         u'duration': {u'text': u'11 hours 38 mins', u'value': 41860},
         u'status': u'OK'},
        {u'distance': {u'text': u'1,676 km', u'value': 1676172},
         u'duration': {u'text': u'15 hours 3 mins', u'value': 54189},
         u'status': u'OK'},
        {u'distance': {u'text': u'1,235 km', u'value': 1234921},
         u'duration': {u'text': u'11 hours 57 mins', u'value': 43048},
         u'status': u'OK'}]},
      {u'elements': [{u'distance': {u'text': u'1,273 km', u'value': 1272730},
         u'duration': {u'text': u'12 hours 13 mins', u'value': 43963},
         u'status': u'OK'},
        {u'distance': {u'text': u'661 km', u'value': 661259},
         u'duration': {u'text': u'6 hours 29 mins', u'value': 23332},
         u'status': u'OK'},
        {u'distance': {u'text': u'1,781 km', u'value': 1780515},
         u'duration': {u'text': u'16 hours 9 mins', u'value': 58119},
         u'status': u'OK'},
        {u'distance': {u'text': u'1 m', u'value': 0},
         u'duration': {u'text': u'1 min', u'value': 0},
         u'status': u'OK'}]}],
     u'status': u'OK'}




```python
# time.gmtime(0)
int(time.time())
```




    1510847993




```python
x = json.loads(req1.text)

for isrc, src in enumerate(x['origin_addresses']):
    for idst, dst in enumerate(x['destination_addresses']):
        row = x['rows'][isrc]
        cell = row['elements'][idst]
        if cell['status'] == 'OK':
            print('{} to {}: {}, {}.'.format(src, dst, cell['distance']['text'], cell['duration']['text']))
        else:
            print('{} to {}: status = {}'.format(src, dst, cell['status']))
```

    2550 Yeager Rd, West Lafayette, IN 47906, USA to Chicago, IL 60601, USA: 192 km, 2 hours 7 mins.
    2550 Yeager Rd, West Lafayette, IN 47906, USA to 1 Nasa Dr, Hampton, VA 23666, USA: 1,235 km, 11 hours 38 mins.
    2550 Yeager Rd, West Lafayette, IN 47906, USA to Walt Disney World Resort, Orlando, FL 32830, USA: 1,676 km, 15 hours 3 mins.
    2550 Yeager Rd, West Lafayette, IN 47906, USA to New York, NY, USA: 1,235 km, 11 hours 57 mins.
    New York, NY, USA to Chicago, IL 60601, USA: 1,273 km, 12 hours 13 mins.
    New York, NY, USA to 1 Nasa Dr, Hampton, VA 23666, USA: 661 km, 6 hours 29 mins.
    New York, NY, USA to Walt Disney World Resort, Orlando, FL 32830, USA: 1,781 km, 16 hours 9 mins.
    New York, NY, USA to New York, NY, USA: 1 m, 1 min.



```python
n_orig = params['origins'].count('|')+1
n_dest = params['destinations'].count('|')+1

print(n_orig)

json_normalize(flatten_json(res1))
```

    1





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
      <th>destination_addresses_0</th>
      <th>destination_addresses_1</th>
      <th>origin_addresses_0</th>
      <th>origin_addresses_1</th>
      <th>rows_0_elements_0_status</th>
      <th>rows_0_elements_1_status</th>
      <th>rows_1_elements_0_distance_text</th>
      <th>rows_1_elements_0_distance_value</th>
      <th>rows_1_elements_0_duration_text</th>
      <th>rows_1_elements_0_duration_value</th>
      <th>rows_1_elements_0_status</th>
      <th>rows_1_elements_1_distance_text</th>
      <th>rows_1_elements_1_distance_value</th>
      <th>rows_1_elements_1_duration_text</th>
      <th>rows_1_elements_1_duration_value</th>
      <th>rows_1_elements_1_status</th>
      <th>status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San Francisco, CA, USA</td>
      <td>1 Nasa Dr, Hampton, VA 23666, USA</td>
      <td>West Lafayette, IN 47906, USA</td>
      <td>610 Purdue Mall, West Lafayette, IN 47907, USA</td>
      <td>ZERO_RESULTS</td>
      <td>ZERO_RESULTS</td>
      <td>3,876 km</td>
      <td>3876406</td>
      <td>2 days 13 hours</td>
      <td>220333</td>
      <td>OK</td>
      <td>1,464 km</td>
      <td>1464467</td>
      <td>1 day 11 hours</td>
      <td>126779</td>
      <td>OK</td>
      <td>OK</td>
    </tr>
  </tbody>
</table>
</div>




```python
d = {'origin': ['47906', '2550+Yeager+Rd','NASA+Langley'],
    'destination': ['60666', '585 Purdue Mall', 'Purdue University'],
    'mode': ['driving', 'transit', 'driving']
    }
df = pd.DataFrame(data=d)
df = df[['origin','destination','mode']]
df
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
      <th>origin</th>
      <th>destination</th>
      <th>mode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47906</td>
      <td>60666</td>
      <td>driving</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2550+Yeager+Rd</td>
      <td>585 Purdue Mall</td>
      <td>transit</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NASA+Langley</td>
      <td>Purdue University</td>
      <td>driving</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(df)
```




    3




```python
gmaps = googlemaps.Client(key=mykey)
matrix = gmaps.distance_matrix(df['origin'][0], df['destination'][0], mode=df['mode'][0])
json_normalize(matrix,['rows','elements'])
# matrixdf
# matrix
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
      <th>distance</th>
      <th>duration</th>
      <th>status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>{u'text': u'223 km', u'value': 222902}</td>
      <td>{u'text': u'2 hours 12 mins', u'value': 7898}</td>
      <td>OK</td>
    </tr>
  </tbody>
</table>
</div>




```python
for i in range(0,len(df)):
    
```


```python
params = {
    'origins': '2550+Yeager+Rd',
    'destinations': '585 Purdue Mall, West Lafayette',
    'key': mykey,
    'mode': 'cycling',
}

req2 = requests.get(GOOGLE_MAPS_API_URL, params=params)
req2 = req2.json()
# print(req2)
json_normalize(flatten_json(req2))
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
      <th>destination_addresses_0</th>
      <th>origin_addresses_0</th>
      <th>rows_0_elements_0_distance_text</th>
      <th>rows_0_elements_0_distance_value</th>
      <th>rows_0_elements_0_duration_text</th>
      <th>rows_0_elements_0_duration_value</th>
      <th>rows_0_elements_0_status</th>
      <th>status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>585 Purdue Mall, West Lafayette, IN 47907, USA</td>
      <td>2550 Yeager Rd, West Lafayette, IN 47906, USA</td>
      <td>3.9 km</td>
      <td>3855</td>
      <td>9 mins</td>
      <td>519</td>
      <td>OK</td>
      <td>OK</td>
    </tr>
  </tbody>
</table>
</div>




```python
params = {
    'origins': '47906',
    'destinations': 'Wellington+Ohio',
    'key': mykey,
    'mode': 'driving',
}

req2 = requests.get(GOOGLE_MAPS_API_URL, params=params)
req2 = req2.json()
# print(req2)
json_normalize(flatten_json(req2))
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
      <th>destination_addresses_0</th>
      <th>origin_addresses_0</th>
      <th>rows_0_elements_0_distance_text</th>
      <th>rows_0_elements_0_distance_value</th>
      <th>rows_0_elements_0_duration_text</th>
      <th>rows_0_elements_0_duration_value</th>
      <th>rows_0_elements_0_status</th>
      <th>status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wellington, OH 44090, USA</td>
      <td>West Lafayette, IN 47906, USA</td>
      <td>477 km</td>
      <td>476982</td>
      <td>4 hours 53 mins</td>
      <td>17604</td>
      <td>OK</td>
      <td>OK</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Sample Maps Directions using Maps Directions API

params = {
    'units': 'imperial',
    'origin': 'Purdue+University',
    'destination': 'NASA+Langley',
    'mode': 'driving',
    'key': mykey
}

req = requests.get(GOOGLE_MAPS_DIR_URL,params=params)
res = req.json()
# print(res)
json_normalize(flatten_json(res))
# print(res)
# print(json.dumps(res, sort_keys=True, indent=3))
# import pdb; pdb.set_trace()
print(res['routes'][0]['legs'][0]['distance']['text'])
print(res['routes'][0]['legs'][0]['duration']['text'])
```

    765 mi
    11 hours 46 mins

