---
layout: default
title: Hello From Brython!
---
<script type="text/python">
from browser import document, alert

def echo(event):
    alert(document["zone"].value)

document["mybutton"].bind("click", echo)
</script>

<input id="zone"><button id="mybutton">click !</button>
