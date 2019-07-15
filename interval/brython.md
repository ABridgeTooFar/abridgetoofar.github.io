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
<div id="timer" style="background-color:black;color:#0F0;padding:15px;font-family:courier;font-weight:bold;font-size:23px;"><p>0.00</p></div>
</td>
</tr>
</tbody></table>

<script type="text/python">
from browser import document

id = None
counter = 0.0

def show(o):
    counter = counter + 1.0
    document["timer"].innerHTML = "<p>%.2f</p>"%(counter)

document["start"].bind("click", show)
</script>
