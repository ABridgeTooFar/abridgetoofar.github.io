---
layout: default
title: Call Me Back
---
<h1>Accepting Data from Trusted External Sites</h1>

<form>
<input type="number" id="feeds" value="0" />
</form>

<div id="myplot" ></div>

<script type="application/javascript">
var feeds = 0;
function showText(jcontent) {
    var field = document.getElementById('feeds');
    feeds = feeds + 1;
    field.value=feeds;
}
	
function load_js() {
	var parms = window.location.search.substr(1).split('&');
	var i;
	for (i = 0; i < parms.length; i++) {
		text = parms[i].split('=')
		if (text[0]=="password") {
			var url = "https://api.openweathermap.org/data/2.5/weather?q=London,uk&APPID="+text[1]+"&callback=showText";
			var old = document.getElementById('jsonp');
			var head= document.getElementsByTagName('body')[0];
			var script= document.createElement('script');
			if (old) {
				old.remove();
			}
			script.id = 'jsonp';
			script.src= url+"&feed="+feeds;
			head.appendChild(script);
			break;
		}
	}
}
</script>

<script type="text/python">
from browser import document, window
from browser.timer import request_animation_frame as raf
from browser.timer import cancel_animation_frame as caf
import time
import math
from datetime import datetime

# paramters of graph
theta0 = 0.0
falseTheta = 0 
nx = 10

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
source = Bokeh.ColumnDataSource.new({
    'data': {'x': [x * 360.0/nx for x in range(nx+1)], 'y': [0.0]*(nx+1) }
})
# create some ranges for the plot
xdr = Bokeh.Range1d.new({ "start": -0.01, "end": 360.01 });
ydr = Bokeh.Range1d.new({ "start": -10.01, "end": 10.01 });

# make the plot and add some tools
tools = "pan,zoom_in,zoom_out,reset"
fig1 = plt.figure({'title': "Data Visualization (1 RPM)", 'tools': tools})
fig1.line({"x": {"field" : "x"}, "y": {"field": "y"}, "source" : source,
    "line_color": "#666699",
    "line_width": 2
})
fig1.x_range=xdr
fig1.y_range=ydr

# show the plot
mydiv = document['myplot']
plt.show(fig1, mydiv.elt)

def UpdateFig1(theta0):
    global nx
    # generate the source data
    delta = (360.0/nx)%360.0    
    lx = [x * delta for x in range(nx+1)]
    ly = [ 10.0 * math.sin(math.radians(theta0+dTheta)) for dTheta in lx]
    #update the source data
    #source.data.x = lx
    source.data.y = ly
    source.change.emit()
    
#animation/timed updates
def TimerUpdate(o):
    global stopRequested
    global id
    global counter
    #
    if stopRequested:
        id = None
    else:
        now = datetime.now()
        elapsed = now - counter
        if elapsed.total_seconds()>=20.0:
            counter = now
            theta0 = UpdateTheta0(120.0) #6-degrees per second
            window.load_js()
            #UpdateFig1(theta0)
        #
        id = raf(TimerUpdate)

def StartHandler(ev):
    global stopRequested
    global timerInstances
    global id
    global counter
    #
    stopRequested = False
    if (timerInstances == 0) and (id is None):
        timerInstances = 1
        counter = datetime.now()
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

#UpdateFig1(theta0)
StartHandler(0)
</script>
