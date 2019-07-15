---
layout: default
title: Stop Watching Me!
---
<h1>Example of a Interval-based Stop-watch</h1>
<button id="start">Start</button>
<br /><button id="stop">Stop</button>
<br /><div id="timer"></div>

<script type="text/python">
from browser import document

def show(o):
    document["timer"].innerHTML = "<p>DEBUG</p>"

document["start"].bind("click", show)
</script>
