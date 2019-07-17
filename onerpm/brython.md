---
layout: default
title: One Minute Revolution
---
<h1>An Animated Sinusoidal Plot</h1>
<div id="myplot" ></div>

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

# animation/timer state variables
stopRequested = False
timerInstances = 0
counter = datetime.now()
id = None

# 'importing' the library
Bokeh = window.Bokeh
plt = Bokeh.Plotting

def draw(theta0,nx):
    lx = []
    theta = theta0
    delta = (360.0/nx)%360.0    
    for x in range(nx):
        if falseTheta == 0:
            theta += delta
        else:
            theta -= delta

        if theta>360.0:
            theta = 360.0 - (theta%360.0)
            falseTheta = 360
        if theta<0.0:
            theta = - (theta%-360.0)
            falseTheta = 0

        lx.append((360.0 - theta) if falseTheta else theta)

    ly = [ 10.0 * math.sin(math.radians(v)) for v in lx]

    # create some ranges for the plot
    # xdr = Bokeh.Range1d.new({ "start": lx[0], "end": lx[-1] });
    ydr = Bokeh.Range1d.new({ "start": -0.5, "end": 20.5 });

    # make the plot and add some tools
    tools = "pan,zoom_in,zoom_out,reset"
    p = plt.figure({'title': "Sine wave (1 RPM)", 'tools': tools})
    #add a Line glyph
    p.line({"x": lx, "y": ly,
        "line_color": "#666699",
        "line_width": 2
    })
    p.y_range=ydr

    # show the plot
    mydiv = document['myplot']
    plt.show(p, mydiv.elt)

#animation/timed updates
def TimerUpdate(o):
    global stopRequested
    global id
    global counter
    global theta0
    global nx

    if stopRequested:
        id = None
    else:
        now = datetime.now()
        elapsed = now - counter
        if elapsed.total_seconds()>=1.0:
            counter = now
            theta0 = (theta0 + 6.0)%360.0 #6-degrees per second
            draw(theta0,nx)

        id = raf(TimerUpdate)

def StartHandler(ev):
    global stopRequested
    global timerInstances
    global id
    global counter

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

draw(theta0,nx)
StartHandler(0)
</script>
