---
layout: default
title: Stop Watching Me!
---
<h1>Example of a Interval-base Stop-watch</h1>
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
from browser import timer

_timer = None
counter = 0

def show():
    doc['_timer'].text = '%.2f' %(time.time()-counter)

def start_timer(ev):
    global _timer,counter
    if _timer is None:
        counter = time.time()
        _timer = timer.set_interval(show,10)
        doc['start'].text = 'Hold'
    elif _timer == 'hold': # restart
        # restart timer
        counter = time.time()-float(doc['_timer'].text)
        _timer = timer.set_interval(show,10)
        doc['start'].text = 'Hold'
    else: # hold
        timer.clear_interval(_timer)
        _timer = 'hold'
        doc['start'].text = 'Restart'

def stop_timer(ev):
    global _timer
    timer.clear_interval(_timer)
    _timer = None
    t = 0
    doc['_timer'].text = '%.2f' %0
    doc['start'].text = 'Start'

doc['start'].bind('click', start_timer)
doc['stop'].bind('click', stop_timer)
</script>
