---
layout: default
title: Stop Watching Me!
---
<h1>Example of a Stop-watch using Animation Frame Technique</h1>
<table cellpadding="10">
<tbody><tr>
<td style="width:100px;">
<button id="start">Start</button>
<br><button id="stop">Stop</button>
</td>
<td>
<div id="timer" style="background-color:black;color:#0F0;padding:15px;font-family:courier;font-weight:bold;font-size:23px;">0.00</div>
</td>
</tr>
</tbody></table>

<script type="text/python">
from browser import document
from browser.timer import request_animation_frame as raf
from browser.timer import cancel_animation_frame as caf
import time
import math
from datetime import datetime

stopRequested = False
timerInstances = 0
counter = datetime.now()
id = None

def TimerUpdate(o):
    global stopRequested
    global id
    global counter

    if stopRequested:
        id = None
    else:
        elapsed = datetime.now() - counter
        document["timer"].innerHTML = "%0.2f"%(elapsed.total_seconds())
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
    if timerInstances>1:
        timerInstances -= 1
    stopRequested = True

document["start"].bind("click", StartHandler)
document["stop"].bind("click", StopHandler)
</script>
