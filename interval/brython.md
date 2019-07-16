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
from datetime import datetime

counter = datetime.now()

def show():
    global counter
    elapsed = datetime.now() - counter
    document["timer"].innerHTML = "<p>%.2f</p>"%elapsed.total_seconds()
    
def start_hold_timer(ev):
    counter = datetime.now()

def stop_timer(ev):
    global counter
    show()

document["start"].bind("click", start_hold_timer)
document["stop"].bind("click", stop_timer)
</script>
