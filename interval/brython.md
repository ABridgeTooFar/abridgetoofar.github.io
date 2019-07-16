---
layout: default
title: Stop Watching Me!
---
<h1>Example of a Interval-based Stop-watch</h1>
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

id = None
counter = datetime.now()

def show():
    global counter
    elapsed = datetime.now() - counter
    document["timer"].innerHTML = "<p>%.2f</p>"%(elapsed.total_seconds())

def start_hold_timer(ev):
    global counter
    global id
    if id is None:
        counter = datetime.now()
    id = raf(start_hold_timer)
    show()

def stop_timer(ev):
    global id
    caf(id)
    id = None
    
document["start"].bind("click", start_hold_timer)
document["stop"].bind("click", stop_timer)
</script>
