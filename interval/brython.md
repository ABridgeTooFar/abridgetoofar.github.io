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
<div id="_timer" style="background-color:black;color:#0F0;padding:15px;font-family:courier;font-weight:bold;font-size:23px;">0.00</div>
</td>
</tr>
</tbody></table>
<script type="text/python">
import time
from browser import document as doc
from browser.timer import request_animation_frame as raf
from browser.timer import cancel_animation_frame as caf

id = None
counter = 0

def show(o):
    doc["_timer"].innerHTML = "%.2f"%(time.time()-counter)

def animate(ev):
    global id
    counter = time.time()
    id = raf(animate)
    show()

def stop_timer(ev):
    global id
    caf(id)

doc["start"].bind("click", show)
doc["stop"].bind("click", stop_timer)
</script>
