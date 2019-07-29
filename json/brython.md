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

geofixes=[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ]]

# paramters of graph
nx = 360
    
# animation/timer state variables
stopRequested = False
timerInstances = 0
counter = datetime.now()
id = None

# 'importing' the library
Bokeh = window.Bokeh
plt = Bokeh.Plotting
colours = ["black","green","blue","red"]
sources = [Bokeh.ColumnDataSource.new({
    'data': {'x': [x * 360.0/nx for x in range(nx+1)], 'y': [0.0]*(nx+1) }
}) for i in colours]

# create some ranges for the plot
xdr = Bokeh.Range1d.new({ "start": -0.01, "end": 360.01 });
ldr = Bokeh.Range1d.new({ "start": -15.01, "end": 15.01 });
rdr = Bokeh.Range1d.new({ "start": -150.01, "end": 150.01 });

# make the plot and add some tools
tools = "pan,zoom_in,zoom_out,reset"
fig1 = plt.figure({'title': "Data Visualization (1 RPM)", 'tools': tools})
fig1.x_range=xdr
fig1.y_range=ldr
fig1.extra_y_ranges["times10"]=rdr
yra = Bokeh.LinearAxis.new({"y_range_name":"times10"})
fig1.add_layout(yra, 'right')

lines = [fig1.line({"x": {"field" : "x"}, "y": {"field": "y"}, "source" : source,
    "line_width": 2,
    "line_color": colour,
    "line_dash" : []
}) for source,colour in zip(sources,colours)]

#for i,source in enumerate([sourceP,sourceT,sourceWN,sourceWE]):
#    lines[i].y_range_name=("times10" if max(abs(source.data.y))>15 else None)ur
# show the plot
mydiv = document['myplot']
plt.show(fig1, mydiv.elt)

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

def UpdateFig1(
    enumOwmlat = 0,
    enumOwmlon = 1,
    enumOwmtemp = 2,
    enumOwmatm = 3,
    enumOwmwspd = 4,
    enumOwmwdir=5
):
    global sources
    global lines
    # generate the source data
    queue=[]
    owmfix = None
    while len(geofixes)>0:
        owmfix=geofixes.pop(0)
        queue.append([
            owmfix[enumOwmatm],
            owmfix[enumOwmtemp],
            owmfix[enumOwmwspd]*math.cos(math.radians(owmfix[enumOwmwdir])),
            owmfix[enumOwmwspd]*math.sin(math.radians(owmfix[enumOwmwdir]))
        ]);
    for values in queue:	
        for source,line,value in zip(sources,lines,values):
            ly = source.data.y[1:]
            if abs(value)>150.0:
                value = ly[-1]
            ly.append(value)
            if abs(value)>15:
                line.y_range_name="times10"
                line.glyph.line_dash=[6, 3]
            #update the source data
            source.data.y = ly
            source.change.emit()
    showText(owmfix)
        
    
#animation/timed updates
def TimerUpdate(o):
    global stopRequested
    global id
    #
    if stopRequested:
        id = None
    else:
        UpdateFig1()
        id = raf(TimerUpdate)

def StartHandler(ev):
    global stopRequested
    global timerInstances
    global id
    #
    stopRequested = False
    if (timerInstances == 0) and (id is None):
        timerInstances = 1
        id = raf(TimerUpdate)

def StopHandler(ev):
    global stopRequested
    global timerInstances
    global id
    if not (id is None):
        caf(id)
        id = None
    if timerInstances>0:
        timerInstances -= 1
    stopRequested = True

def Every500ms():
    global counter
    fakeapi="b6907d289e10d714a6e88b30761fae22"
    apikey=document.query.getvalue("password",fakeapi)
    if (apikey!=fakeapi):
        now = datetime.now()
        elapsed = now - counter
        if elapsed.total_seconds()>=2.0:
            counter = now
            window.load_js(apikey)
        timer.set_timeout(Every500ms, 500)
    else:
        window.alert("You must provide your own APIKEY")

async def Every10s():
    """Get position from window.navigator.geolocation and put marker on the
    map.
    """
    url = "https://geo.weather.gc.ca/geomet?service=WFS&version=2.0.0&request=GetFeature&typename=CURRENT_CONDITIONS&filter=<Filter><PropertyIsEqualTo><PropertyName>name</PropertyName><Literal>Deer Lake</Literal></PropertyIsEqualTo></Filter>&OUTPUTFORMAT=GeoJSON"
    req = await aio.get(url)
    data = json.loads(req.data)
    for feature in data["features"]: 
        properties = feature["properties"]
        geometry = feature["geometry"]
        lat, long = [float(v) for v in geometry["coordinates"]]
        timeOfFix = properties["timestamp"]
    document["geoasof"].value = timeOfFix
    #enumOwmlat = 0,
    #enumOwmlon = 1,
    #enumOwmtemp = 2,
    #enumOwmatm = 3,
    #enumOwmwspd = 4,
    #enumOwmwdir=5
    geofixes.append( [lat,long,float(properties["temp"]),float(properties["pres_en"]),
        float(properties["speed"]),float(properties["bearing"]) ] )
    #
    # Put marker on map
    #leaflet.marker([lat, long], {"icon": icon}).addTo(mymap)

async def main():
    StartHandler(0)
    while True:
        await Every10s()
        await aio.sleep(10)
    StopHandler(0)

aio.run(main())</script>
