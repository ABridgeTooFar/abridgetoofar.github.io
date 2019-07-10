---
layout: default
title: In Click Out!
---
<script type="text/python">
from browser import document

def echo(event):
    document["result"].innerHTML='<textarea rows="4" cols="100%">'+document["zone"].value+'</textarea>'

document["mybutton"].bind("click", echo)
</script>

<textarea id="zone" rows="4" cols="50"></textarea><button id="mybutton">click !</button>
<div id="result"></div>
