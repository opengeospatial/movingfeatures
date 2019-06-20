
# How to use OGC Moving Features CSV with Python

## To read Moving Features CSV data files
Python provides CSV package to read CSV data. A file encoded with Moving Features CSV can be read easily with the package. The following function reads OGC Moving Features CSV and returns tuple of a metadata dictionary and pandas.DataFrame.


```python
import csv
import pandas
import numpy
import datetime

def read_mfcsv(filename):
    metadata = {}
    with open(filename, 'r') as f:
        reader = csv.reader(f)
        stboundedby = next(reader)
        metadata['stboundedby'] = {
            'coordinate' : stboundedby[1:3],
            'bounds' : stboundedby[3:5],
            'timerange' : 
            {
               'starttime': datetime.datetime.strptime(stboundedby[5],'%Y-%m-%dT%H:%M:%Sz'),
               'endtime': datetime.datetime.strptime(stboundedby[6],'%Y-%m-%dT%H:%M:%Sz'),
               'unit': stboundedby[7]
            }
        }
        columninfo = next(reader)
        metadata['columninfo'] = {
            'attr_setting': dict(zip(columninfo[3::2],columninfo[4::2]))
        }
    databody = pandas.read_csv(filename, skiprows=2, header=None)
    databody.columns=['mfidref','starttime','endtime','trajectory']+list(metadata['columninfo']['attr_setting'].keys())
    databody['trajectory'] = databody['trajectory'].apply(lambda x: [[float(x),float(y)] for x,y in zip(x.split(' ')[0::2],x.split(' ')[1::2])])
    return metadata, databody
```

A sample file 'test_mf.csv' can be read as follows:


```python
metadata, databody = read_mfcsv('mf.csv')
databody
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
      <th>mfidref</th>
      <th>starttime</th>
      <th>endtime</th>
      <th>trajectory</th>
      <th>state</th>
      <th>type code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a</td>
      <td>10</td>
      <td>120</td>
      <td>[[35.6815, 139.7651], [35.682, 139.7661]]</td>
      <td>walking</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>b</td>
      <td>10</td>
      <td>190</td>
      <td>[[35.6811, 139.7662], [35.6818, 139.7661]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a</td>
      <td>120</td>
      <td>150</td>
      <td>[[35.682, 139.7661], [35.6834, 139.7662]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a</td>
      <td>150</td>
      <td>190</td>
      <td>[[35.6834, 139.7662], [35.6835, 139.7663]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



The startime and the endtime can be easily converted into absolute style.


```python
time_converter ={
     'sec' : lambda s,o: datetime.timedelta(seconds=s)+o,
     'minute' : lambda s,o: datetime.timedelta(minutes=s)+o,
     'absolute' : lambda s,o: s
}

origin = metadata['stboundedby']['timerange']['starttime']
unit = metadata['stboundedby']['timerange']['unit']

databody['starttime']=databody['starttime'].apply(lambda x: time_converter[unit](x,origin))
databody['endtime']=databody['endtime'].apply(lambda x: time_converter[unit](x,origin))
databody
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
      <th>mfidref</th>
      <th>starttime</th>
      <th>endtime</th>
      <th>trajectory</th>
      <th>state</th>
      <th>type code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a</td>
      <td>2012-01-17 12:33:51</td>
      <td>2012-01-17 12:35:41</td>
      <td>[[35.6815, 139.7651], [35.682, 139.7661]]</td>
      <td>walking</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>b</td>
      <td>2012-01-17 12:33:51</td>
      <td>2012-01-17 12:36:51</td>
      <td>[[35.6811, 139.7662], [35.6818, 139.7661]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a</td>
      <td>2012-01-17 12:35:41</td>
      <td>2012-01-17 12:36:11</td>
      <td>[[35.682, 139.7661], [35.6834, 139.7662]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a</td>
      <td>2012-01-17 12:36:11</td>
      <td>2012-01-17 12:36:51</td>
      <td>[[35.6834, 139.7662], [35.6835, 139.7663]]</td>
      <td>walking</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Plot the loaded tracks
folium is useful package for plotting geospatial data.


```python
import folium
m = folium.Map(location=[35.6815,139.7651], zoom_start=17)
m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVM9ZmFsc2U7IExfTk9fVE9VQ0g9ZmFsc2U7IExfRElTQUJMRV8zRD1mYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4zLjQvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4zLjQvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdjZG4uZ2l0aGFjay5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLAogICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgIDxzdHlsZT4jbWFwXzZmZmIwZWFiYTQzMDQxZTc4YTc5M2Q4ZmIzZWVkN2RkIHsKICAgICAgICBwb3NpdGlvbjogcmVsYXRpdmU7CiAgICAgICAgd2lkdGg6IDEwMC4wJTsKICAgICAgICBoZWlnaHQ6IDEwMC4wJTsKICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgIHRvcDogMC4wJTsKICAgICAgICB9CiAgICA8L3N0eWxlPgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgPGRpdiBjbGFzcz0iZm9saXVtLW1hcCIgaWQ9Im1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCIgPjwvZGl2Pgo8L2JvZHk+CjxzY3JpcHQ+ICAgIAogICAgCiAgICAKICAgICAgICB2YXIgYm91bmRzID0gbnVsbDsKICAgIAoKICAgIHZhciBtYXBfNmZmYjBlYWJhNDMwNDFlNzhhNzkzZDhmYjNlZWQ3ZGQgPSBMLm1hcCgKICAgICAgICAnbWFwXzZmZmIwZWFiYTQzMDQxZTc4YTc5M2Q4ZmIzZWVkN2RkJywgewogICAgICAgIGNlbnRlcjogWzM1LjY4MTUsIDEzOS43NjUxXSwKICAgICAgICB6b29tOiAxNywKICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICBsYXllcnM6IFtdLAogICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgfSk7CgogICAgCiAgICAKICAgIHZhciB0aWxlX2xheWVyXzgzZDIyYzJlMTNiYzQ0OTU5OTRhODRhNGM4Mzg3ZTdiID0gTC50aWxlTGF5ZXIoCiAgICAgICAgJ2h0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nJywKICAgICAgICB7CiAgICAgICAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAgICAgICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgICAgICAgIm1heE5hdGl2ZVpvb20iOiAxOCwKICAgICAgICAibWF4Wm9vbSI6IDE4LAogICAgICAgICJtaW5ab29tIjogMCwKICAgICAgICAibm9XcmFwIjogZmFsc2UsCiAgICAgICAgIm9wYWNpdHkiOiAxLAogICAgICAgICJzdWJkb21haW5zIjogImFiYyIsCiAgICAgICAgInRtcyI6IGZhbHNlCn0pLmFkZFRvKG1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCk7Cjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



The list in column 'trajectory' should be added to the map for the visualization.


```python
for l in databody['trajectory']:
    folium.PolyLine(locations=l).add_to(m)
m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVM9ZmFsc2U7IExfTk9fVE9VQ0g9ZmFsc2U7IExfRElTQUJMRV8zRD1mYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4zLjQvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4zLjQvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdjZG4uZ2l0aGFjay5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLAogICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgIDxzdHlsZT4jbWFwXzZmZmIwZWFiYTQzMDQxZTc4YTc5M2Q4ZmIzZWVkN2RkIHsKICAgICAgICBwb3NpdGlvbjogcmVsYXRpdmU7CiAgICAgICAgd2lkdGg6IDEwMC4wJTsKICAgICAgICBoZWlnaHQ6IDEwMC4wJTsKICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgIHRvcDogMC4wJTsKICAgICAgICB9CiAgICA8L3N0eWxlPgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgPGRpdiBjbGFzcz0iZm9saXVtLW1hcCIgaWQ9Im1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCIgPjwvZGl2Pgo8L2JvZHk+CjxzY3JpcHQ+ICAgIAogICAgCiAgICAKICAgICAgICB2YXIgYm91bmRzID0gbnVsbDsKICAgIAoKICAgIHZhciBtYXBfNmZmYjBlYWJhNDMwNDFlNzhhNzkzZDhmYjNlZWQ3ZGQgPSBMLm1hcCgKICAgICAgICAnbWFwXzZmZmIwZWFiYTQzMDQxZTc4YTc5M2Q4ZmIzZWVkN2RkJywgewogICAgICAgIGNlbnRlcjogWzM1LjY4MTUsIDEzOS43NjUxXSwKICAgICAgICB6b29tOiAxNywKICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICBsYXllcnM6IFtdLAogICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgfSk7CgogICAgCiAgICAKICAgIHZhciB0aWxlX2xheWVyXzgzZDIyYzJlMTNiYzQ0OTU5OTRhODRhNGM4Mzg3ZTdiID0gTC50aWxlTGF5ZXIoCiAgICAgICAgJ2h0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nJywKICAgICAgICB7CiAgICAgICAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAgICAgICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgICAgICAgIm1heE5hdGl2ZVpvb20iOiAxOCwKICAgICAgICAibWF4Wm9vbSI6IDE4LAogICAgICAgICJtaW5ab29tIjogMCwKICAgICAgICAibm9XcmFwIjogZmFsc2UsCiAgICAgICAgIm9wYWNpdHkiOiAxLAogICAgICAgICJzdWJkb21haW5zIjogImFiYyIsCiAgICAgICAgInRtcyI6IGZhbHNlCn0pLmFkZFRvKG1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCk7CiAgICAKICAgICAgICAgICAgICAgIHZhciBwb2x5X2xpbmVfODVmZTIwNTYyNzMzNGNhYjk5MmNhZGI5M2Q1OWI4Y2QgPSBMLnBvbHlsaW5lKAogICAgICAgICAgICAgICAgICAgIFtbMzUuNjgxNSwgMTM5Ljc2NTFdLCBbMzUuNjgyLCAxMzkuNzY2MV1dLAogICAgICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMzMzg4ZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IGZhbHNlLAogICJmaWxsQ29sb3IiOiAiIzMzODhmZiIsCiAgImZpbGxPcGFjaXR5IjogMC4yLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAibm9DbGlwIjogZmFsc2UsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInNtb290aEZhY3RvciI6IDEuMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfNmZmYjBlYWJhNDMwNDFlNzhhNzkzZDhmYjNlZWQ3ZGQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICAgICAgdmFyIHBvbHlfbGluZV84YjJkMjFhYzQ2ZGM0YmQ0ODg4MjJmOTBhMGRjZjkzZiA9IEwucG9seWxpbmUoCiAgICAgICAgICAgICAgICAgICAgW1szNS42ODExLCAxMzkuNzY2Ml0sIFszNS42ODE4LCAxMzkuNzY2MV1dLAogICAgICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiMzMzg4ZmYiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IGZhbHNlLAogICJmaWxsQ29sb3IiOiAiIzMzODhmZiIsCiAgImZpbGxPcGFjaXR5IjogMC4yLAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAibm9DbGlwIjogZmFsc2UsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInNtb290aEZhY3RvciI6IDEuMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfNmZmYjBlYWJhNDMwNDFlNzhhNzkzZDhmYjNlZWQ3ZGQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICAgICAgdmFyIHBvbHlfbGluZV8wODBjOGQyZjlmZWE0MzdkOTEzOTY2MmYwNjNmNzU5ZCA9IEwucG9seWxpbmUoCiAgICAgICAgICAgICAgICAgICAgW1szNS42ODIsIDEzOS43NjYxXSwgWzM1LjY4MzQsIDEzOS43NjYyXV0sCiAgICAgICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzMzODhmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogZmFsc2UsCiAgImZpbGxDb2xvciI6ICIjMzM4OGZmIiwKICAiZmlsbE9wYWNpdHkiOiAwLjIsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJub0NsaXAiOiBmYWxzZSwKICAib3BhY2l0eSI6IDEuMCwKICAic21vb3RoRmFjdG9yIjogMS4wLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAgICAgLmFkZFRvKG1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgICAgICB2YXIgcG9seV9saW5lXzNlODJiMmEyMGVjNDQzYjc4OTJhOTg5ZTg3ODRiMjY3ID0gTC5wb2x5bGluZSgKICAgICAgICAgICAgICAgICAgICBbWzM1LjY4MzQsIDEzOS43NjYyXSwgWzM1LjY4MzUsIDEzOS43NjYzXV0sCiAgICAgICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzMzODhmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogZmFsc2UsCiAgImZpbGxDb2xvciI6ICIjMzM4OGZmIiwKICAiZmlsbE9wYWNpdHkiOiAwLjIsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJub0NsaXAiOiBmYWxzZSwKICAib3BhY2l0eSI6IDEuMCwKICAic21vb3RoRmFjdG9yIjogMS4wLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAgICAgLmFkZFRvKG1hcF82ZmZiMGVhYmE0MzA0MWU3OGE3OTNkOGZiM2VlZDdkZCk7CiAgICAgICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python

```
