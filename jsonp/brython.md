---
layout: default
title: Call Me Back
---
<h1>Accepting Data from Trusted External Sites</h1>

<form name="owmfix" id="owmfix">
Sequence: <input type="number" id="owmseq" name="owmseq" value = "0" /> <br />
Latitude: <input type="number" id="owmlat" name="owmlat" value = "0.0" /> Longitude: <input type="number" id="owmlon" name="owmlon" value="-179" /> <br />
Temperature: <input type="number" id="owmtemp" name="owmtemp" value = "0.0" /> Pressure: <input type="number" id="owmatm" name="owmatm" value="0" /> <br />
Wind speed: <input type="number" id="owmwspd" name="owmwspd" value = "0.0" /> Direction: <input type="number" id="owmwdir" name="owmwdir" value="0" />
</form>

<div id="myplot" ></div>

<script type="application/javascript">
var feeds = 0;
var owmfixes = []

function recordContent(jcontent) {
    owmfixes.push([
        parseFloat(jcontent.coord.lat),
        parseFloat(jcontent.coord.lon),
        parseFloat(jcontent.main.temp),
        parseFloat(jcontent.main.pressure),
        parseFloat(jcontent.wind.speed),
    	parseFloat(jcontent.wind.deg)
   ]);
}

function showText(jcontent) {
    var form = document.getElementById('owmfix');
    feeds = feeds + 1
    form["owmlat"].value = jcontent.coord.lat
    form["owmlon"].value  = jcontent.coord.lon
    form["owmtemp"].value = jcontent.main.temp
    form["owmatm"].value = jcontent.main.pressure
    form["owmwspd"].value = jcontent.wind.speed
    form["owmwdir"].value = jcontent.wind.deg
    if (owmfixes.length<360) {
        recordContent(jcontent);
    }
    form["owmseq"].value = feeds; //update the sequence ID last 
}

function load_js(apikey) {
    var i;
    var owm = "https://api.openweathermap.org/data/2.5/weather"
    var form = document.getElementById('owmfix');
    var lat = 0.0;
    var lon = -179.0
    if (feeds > 0) {
        lon = parseFloat(form["owmlon"].value) + 1
        lat = parseFloat(form["owmlat"].value)
        if (lon>180.0) {
            lon -= 360.0
        }
    }
    var url = owm+"?APPID="+apikey+"&lat="+lat+"&lon="+lon+"&callback=showText&seq="+Math.floor(feeds/360);
    var old = document.getElementById('jsonp');
    var head= document.getElementsByTagName('body')[0];
    var script= document.createElement('script');
    if (old) {
        old.remove();
    }
    script.id = 'jsonp';
    script.src= url;
    head.appendChild(script);
}
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

# paramters of graph
theta0 = 0.0
falseTheta = 0 
nx = 360

def UpdateTheta0(delta):
    global theta0,falseTheta
    #    
    delta = delta % 360.0 #make sure delta is positive and modulo 360
    if falseTheta == 0:
        theta0 += delta
    else:
        theta0 -= delta
    #fi
    if theta0>360.0:
        theta0 = 360.0 - (theta0%360.0)
        falseTheta = 360
    if theta0<0.0:
        theta0 = - (theta0%-360.0)
        falseTheta = 0
    #fi
    return ((360.0 - theta0) if falseTheta else theta0)
    
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

def UpdateFig1(theta0):
    global sources
    global lines
    # generate the source data
    queue=[]
    while len(window.owmfixes)>0:
        owmfix=window.owmfixes.pop(0)
        queue.append([
	        0.1*ownfix[window.owmatm],
            ownfix[window.owmtemp]-273.15,
            ownfix[window.owmwspd]*math.cos(math.radians(ownfix[window.owmwdir])),
            ownfix[window.owmwspd]*math.sin(math.radians(ownfix[window.owmwdir]))
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
    
#animation/timed updates
feeds = -1
def TimerUpdate(o):
    global stopRequested
    global id
    global feeds
    #
    if stopRequested:
        id = None
    else:
        if feeds<0:
            feeds = 0
            theta0 = UpdateTheta0(0.0)
            UpdateFig1(theta0)
        else:
            field = document['owmseq']
            seq = int(field.value)
            if seq > feeds:
                feeds = seq
                theta0 = UpdateTheta0(12.0) #6-degrees per second
                UpdateFig1(theta0)
        #
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

timer.set_timeout(Every500ms, 500)
#UpdateFig1(theta0)
StartHandler(0)
</script>
