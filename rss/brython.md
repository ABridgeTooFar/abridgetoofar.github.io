---
layout: default
title: RSS Feed with Graph
---

<h1>Visualization of RSS Data</h1>

<iframe id="html_version"></iframe>
<div id="text_version"></div>
<!-- script  type="application/javascript" / -->
<script type="application/javascript">
function showText(response) {
    var jcontent= JSON.parse({"status" : "OK"});//response);
    var output=document.getElementById('text_version');
    var list="<dt>Key</dt><dd>Term</dd>";
    for (key in jcontent) {
 	    list+="<dt>"+key+"</dt><dd>"+jcontent[key]+"</dd>";
    }	
    output.innerHTML="<dl>"+list+"</dl>";
}
	
function load_js() {
	var parms = window.location.search.substr(1).split('&');
	var i;
	for (i = 0; i < parms.length; i++) {
		text = parms[i].split('=')
		if (text[0]=="password") {
			var url = "https://api.openweathermap.org/data/2.5/weather?q=London,uk&APPID="+text[1]+"&callback=showText";
			var old = document.getElementById('jsonp');
			var head= document.getElementsByTagName('head')[0];
			var script= document.createElement('script');
			if (old) {
				old.remove();
			}
			script.id = 'jsonp';
			script.src= url;
			head.appendChild(script);
			break;
		}
	}
}

function updateWeather() {
	var parms = window.location.search.substr(1).split('&');
	var i;
	for (i = 0; i < parms.length; i++) {
		text = parms[i].split('=')
		if (text[0]=="password") {
			var output=document.getElementById('html_version');
			var url = "https://api.openweathermap.org/data/2.5/weather?q=London,uk&APPID="+text[1]+"&mode=html";
			output.src = url;			
			load_js();
			break;
		}
	}	
}

</script>

<p><button onclick="updateWeather();">Refresh</button></p>

<div id="myplot"></div>

<script type="text/python">
from browser import document, window
import time
import math
import json
from datetime import datetime
from browser import timer
from email import message_from_string 
from browser.timer import request_animation_frame as raf
from browser.timer import cancel_animation_frame as caf

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
fig1 = plt.figure({'title': "Sine wave (1 RPM)", 'tools': tools})
fig1.line({"x": {"field" : "x"}, "y": {"field": "y"}, "source" : source,
    "line_color": "#666699",
    "line_width": 2
})
fig1.x_range=xdr
fig1.y_range=ydr

# show the plot
mydiv = document['myplot']
#plt.show(fig1, mydiv.elt)

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
        if elapsed.total_seconds()>=1.0:
            counter = now
            theta0 = UpdateTheta0(6.0) #6-degrees per second
            UpdateFig1(theta0)
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


def fake_qs():
    return "?foo=%s"%time.time()
        
</script>
