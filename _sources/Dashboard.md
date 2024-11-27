# Interactive Dashboard Creation with Streamlit
This notebook demonstrates how to create an interactive dashboard using **Streamlit**. We'll walk through building a dashboard for analyzing motor vehicle collisions in New York City.

<a href="https://github.com/shivesh235/web_app_vehicle_collision" class="project-link" target="_blank">
                        View Github Repo <i class="fas fa-external-link-alt" style="margin-left: 0.5rem;"></i>

---

<img src="img/web1.png" alt="dashboard" height="600">
<img src="img/web2.png" alt="dashboard" height="600">
<img src="img/web3.png" alt="dashboard" height="600">

---


## Step 1: Install Required Libraries
Before we begin, ensure you have the following libraries installed:
- `streamlit`
- `pandas`
- `numpy`
- `pydeck`
- `plotly.express`

You can install them using:
```bash
pip install streamlit pandas numpy pydeck plotly
```


---

```python
# Step 2: Import Required Libraries
import streamlit as st
import pandas as pd
import numpy as np
import pydeck as pdk
import plotly.express as px
```

---


## Step 3: Load and Preprocess Data
We'll load the dataset containing motor vehicle collision data for New York City. We'll also preprocess the data to remove missing values and rename columns for convenience.


```python
# Define the file path
DATA_URL = "/home/sie/Desktop/prj/web_app/Motor_Vehicle_Collisions_-_Crashes.csv"

# Define a function to load the data
@st.cache(persist=True)
def load_data(nrows):
    data = pd.read_csv(DATA_URL, nrows=nrows, parse_dates=[['CRASH_DATE', 'CRASH_TIME']])
    data.dropna(subset=['LATITUDE', 'LONGITUDE'], inplace=True)
    lowercase = lambda x: str(x).lower()
    data.rename(lowercase, axis='columns', inplace=True)
    data.rename(columns={'crash_date_crash_time': 'date/time'}, inplace=True)
    return data

# Load the data
data = load_data(100000)
original_data = data
```

---


## Step 4: Create the Dashboard Title and Description
Set up the Streamlit title and description for the dashboard.


```python
st.title("Motor Vehicle Collisions in New York City")
st.markdown("This application is a Streamlit dashboard that analyzes motor vehicle collisions in NYC 🚗")
```

---


## Step 5: Visualize Injured People on a Map
Use a slider to filter data based on the number of injured persons and plot their locations on a map.


```python
# Filter and visualize data
st.header("Where are the most people injured in NYC?")
injured_people = st.slider("Number of persons injured", 0, 19)
st.map(data.query("injured_persons >= @injured_people")[["latitude", "longitude"]].dropna(how="any"))
```

---


## Step 6: Analyze Collisions by Hour
Use a slider to select an hour of the day and display vehicle collisions during that hour using PyDeck.


```python
# Filter data by hour
st.header("How many collisions occur during a given hour of the day?")
hour = st.slider("Hour to look at", 0, 23)
data = data[data['date/time'].dt.hour == hour]

# Display map with collisions
st.markdown("Vehicle collisions between %i:00 and %i:00" % (hour, (hour + 1) % 24))
midpoint = (np.average(data['latitude']), np.average(data['longitude']))

st.write(pdk.Deck(
    map_style="mapbox://styles/mapbox/light-v9",
    initial_view_state={
        "latitude": midpoint[0],
        "longitude": midpoint[1],
        "zoom": 11,
        "pitch": 50,
    },
    layers=[
        pdk.Layer(
            "HexagonLayer",
            data=data[['date/time', 'latitude', 'longitude']],
            get_position=['longitude', 'latitude'],
            radius=100,
            extruded=True,
            elevation_scale=4,
            elevation_range=[0, 1000],
        ),
    ],
))
```

---
  

## Step 7: Breakdown Collisions by Minute
Create a histogram to analyze the frequency of collisions by minute within the selected hour.


```python
st.subheader("Breakdown by minute between %i:00 and %i:00" % (hour, (hour + 1) % 24))
filtered = data[
    (data['date/time'].dt.hour >= hour) & (data['date/time'].dt.hour < (hour + 1))
]
hist = np.histogram(filtered['date/time'].dt.minute, bins=60, range=(0, 60))[0]
chart_data = pd.DataFrame({'minute': range(60), 'crashes': hist})

# Create bar chart
fig = px.bar(chart_data, x='minute', y='crashes', hover_data=['minute', 'crashes'], height=400)
st.write(fig)
```

---


## Step 8: Display Top 5 Dangerous Streets
Use a dropdown menu to select an affected type (Pedestrians, Cyclists, or Motorists) and display the top 5 dangerous streets for that group.


```python
st.header("Top 5 Dangerous Streets by Affected Type")
select = st.selectbox('Affected type of people', ['Pedestrians', 'Cyclists', 'Motorists'])

if select == 'Pedestrians':
    st.write(original_data.query("injured_pedestrians >= 1")[["on_street_name", "injured_pedestrians"]].sort_values(by=['injured_pedestrians'], ascending=False).dropna(how='any')[:5])

elif select == 'Cyclists':
    st.write(original_data.query("injured_cyclists >= 1")[["on_street_name", "injured_cyclists"]].sort_values(by=['injured_cyclists'], ascending=False).dropna(how='any')[:5])

else:
    st.write(original_data.query("injured_motorists >= 1")[["on_street_name", "injured_motorists"]].sort_values(by=['injured_motorists'], ascending=False).dropna(how='any')[:5])
```

---
 

## Step 9: Display Raw Data
Add an optional checkbox to display the raw data used in the dashboard.


```python
if st.checkbox("Show Raw Data", False):
    st.subheader('Raw Data')
    st.write(data)
```
