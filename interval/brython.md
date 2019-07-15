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

counter = time.time()

def show():
    elapsed = time.time() - counter
    document["timer"].innerHTML = "<p>DEBUG:%f</p>"%elapsed
    
def start_hold_timer(ev):
    show()

def stop_timer(ev):
    global counter;
    counter = time.time()

document["start"].bind("click", start_hold_timer)
document["stop"].bind("click", stop_timer)
</script>
