---
layout: default
title: Stop Watching Me!
---
<h1>Example of a Interval-based Stop-watch</h1>
<button id="start">Start</button>
<br /><button id="stop">Stop</button>
<br />
<div id="timer"></div>

<script type="text/python">
from browser import document
import time
import math
import datetime

counter = 0

def show():
    global counter
    counter = counter + 1
    document["timer"].innerHTML = "<p>DEBUG%i</p>"%counter
    
def start_hold_timer(ev):
    show()

def stop_timer(ev):
    global counter
    counter = datetime.now()

document["start"].bind("click", start_hold_timer)
document["stop"].bind("click", stop_timer)
</script>
