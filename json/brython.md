---
layout: default
title: AJAX or Bust
---
<h1>Data Transfer without an API key</h1>

<div id="mapid"></div>
<div id="debugme"></div>

<form name="geofix" id="geofix">
Sequence: <input type="number" id="geoseq" name="geoseq" value = "0" /> As of: <input id="geoasof" name="geoasof" value = "" />  <br />
Latitude: <input type="number" id="geolat" name="geolat" value = "0.0" /> Longitude: <input type="number" id="geolon" name="geolon" value="-179" /> <br />
Temperature: <input type="number" id="geotemp" name="geotemp" value = "0.0" /> Pressure: <input type="number" id="geoatm" name="geoatm" value="0" /> <br />
Wind speed: <input type="number" id="geowspd" name="geowspd" value = "0.0" /> Direction: <input type="number" id="geowdir" name="geowdir" value="0" />
</form>

<div id="myplot" ></div>

<!--  src="https://geo.weather.gc.ca/geomet?service=WFS&version=2.0.0&request=GetFeature&typename=CURRENT_CONDITIONS&filter=<Filter><PropertyIsEqualTo><PropertyName>name</PropertyName><Literal>Deer Lake</Literal></PropertyIsEqualTo></Filter>&OUTPUTFORMAT=GeoJSON">
-->
<script type='application/json'>
var jsonpfixes=[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ]]
</script>


<script type="text/python">
from browser import document, window
from browser import timer
from browser.timer import request_animation_frame as raf
from browser.timer import cancel_animation_frame as caf
import time
import math
from datetime import datetime
import json
from browser import aio

geofixes=dict()

feeds = 0;
def showText(owmfix,
    enumOwmlat = 0,
    enumOwmlon = 1,
    enumOwmtemp = 2,
    enumOwmatm = 3,
    enumOwmwspd = 4,
    enumOwmwdir=5,
    enumOwmasof = 6
):
    global feeds;
    if not (owmfix is None):
        form = document;
        feeds = feeds + 1
        form["geolat"].value = owmfix[enumOwmlat]
        form["geolon"].value  = owmfix[enumOwmlon]
        form["geotemp"].value = "%0.3f"%(owmfix[enumOwmtemp])
        form["geoatm"].value = "%0.3f"%(owmfix[enumOwmatm])
        form["geowspd"].value = owmfix[enumOwmwspd]
        form["geowdir"].value = owmfix[enumOwmwdir]
        form["geoasof"].value = owmfix[enumOwmasof]; 
        form["geoseq"].value = feeds; 
        
pickkey = ""
async def queueData():
    global geofixes
    global pickkey
    url = "https://geo.weather.gc.ca/geomet?service=WFS&version=2.0.0&request=GetFeature&typename=CURRENT_CONDITIONS&OUTPUTFORMAT=GeoJSON"
    req = await aio.get(url)
    data = json.loads(req.data)
    document["debugme"].innerHTML="Received Data"
    if data and ("features" in data):
        picklat = 47.54
        picklon = -54.47
        for feature in data["features"]: 
            if all([key in feature for key in ["properties","geometry"]]): 
                language="en"
                properties=feature["properties"]
                geometry=feature["geometry"]
                if all([(key in properties) for key in ["station_en","timestamp","temp","pres_en","speed","bearing"] ]):
                    if "coordinates" in geometry:
                        coordinates = geometry["coordinates"]
                        lon = float(coordinates[0])
                        lat = float(coordinates[1])
                        station = properties["station_en"]
                        timeOfFix = properties["timestamp"]
                        if pickkey == "":
                           pickkey = station
                           geofixes[station] = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0 , timeOfFix ]
                        try:
                            #     #enumOwmlat = 0,
                            #     #enumOwmlon = 1,
                            #     #enumOwmtemp = 2,
                            #     #enumOwmatm = 3,
                            #     #enumOwmwspd = 4,
                            #     #enumOwmwdir=5
                            if (picklat-4.0 < lat) and (lat < picklat+4.0) and (picklon-4.0 <lon) and (lon <picklon+4.0):
                                geofixes[station] = [
                                    lat,
                                    lon, 
                                    float(properties["temp"]) ,float(properties["pres_"+language]),
                                    float(properties["speed"]), float(properties["bearing"]), 
                                    timeOfFix 
                                ]
                                pickkey = station                            
                            # Put marker on map
                            # leaflet.marker([lat, lon], {"icon": icon}).addTo(mymap)
                        except:
                            document["debugme"].innerHTML=station
        if pickkey in geofixes:
            document["debugme"].innerHTML=pickkey
            showText(geofixes[pickkey])
        else:
            document["debugme"].innerHTML="No station nearby"
    
async def main():
    while True:
        await queueData()
        await aio.sleep(10)

aio.run(main())</script>
