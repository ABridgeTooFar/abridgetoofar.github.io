---
layout: default
title: AJAX or Bust
---
<h1>Data Transfer without an API key</h1>

<div id="mapid"></div>

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
    enumOwmwdir=5
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
        form["geoseq"].value = feeds; 

        
async def queueData():
    """Get position from window.navigator.geolocation and put marker on the
    map.
    """
    url = "https://geo.weather.gc.ca/geomet?service=WFS&version=2.0.0&request=GetFeature&typename=CURRENT_CONDITIONS&OUTPUTFORMAT=GeoJSON"
    req = await aio.get(url)
    data = json.loads(req.data)
    language="en"
    picklat=47.54
    picklon=-54.47
    pickkey=""
    for feature in data["features"]: 
        properties = feature["properties"]
        if not all([key in properties for key in ["station_"+language,"timestamp","temp","pres_"+language,"speed","bearing"]]):
            continue
        station = properties["station_"+language];
        if station:
            geometry = feature["geometry"]
            lon, lat = [float(v) for v in geometry["coordinates"]]
            timeOfFix = properties["timestamp"]
            #enumOwmlat = 0,
            #enumOwmlon = 1,
            #enumOwmtemp = 2,
            #enumOwmatm = 3,
            #enumOwmwspd = 4,
            #enumOwmwdir=5
            geofixes[station]=[
                lat,lon,float(properties["temp"]),float(properties["pres_"+language]),
                float(properties["speed"]),float(properties["bearing"]),timeOfFix 
            ]
            if (picklat-4.0<lat<picklat+4.0) and (picklon-4.0<lon<picklon+4.0):
                pickkey = station
            #
            # Put marker on map
            #leaflet.marker([lat, lon], {"icon": icon}).addTo(mymap)
    if pickkey in geofixes:        
        pass #showText(geofixes[pickkey])

async def main():
    await queueData()
    await aio.sleep(10)

aio.run(main())</script>
