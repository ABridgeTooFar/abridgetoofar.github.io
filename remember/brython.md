---
layout: default
title: Do Undo Others!
---
<script type="text/python">
from browser import document

keep = []
discard = dict()

def showout():
    document["result"].innerHTML='<textarea readonly rows="4" cols="100%">'+'\n'.join(keep)+'</textarea>'

def append(event):
    if not '/textarea' in document["zone"].value:
        lines = length(keep)
        discard[lines]=keep.append(document["zone"].value)
        lines += 1
        document["enoz"].value=(discard[lines] if lines in discard else "")
        document["zone"]=""
        showout()
        document["zone"].focus()

def undo(event):
    lines = length(keep)
    if lines:
        lines -= 1
        document["zone"].value=document["enoz"].value=keep.pop(lines)        
        showout()
    document["zone"].focus()

def redo(event):
    lines = length(keep)
    if lines in discard:
        keep.append(discard[lines])        
        lines += 1
        if lines in discard:
            document["zone"].value=document["enoz"].value=discard[lines]        
        else:
            document["zone"].value=document["enoz"].value=""
        showout()
    document["zone"].focus()

document["enter"].bind("click", append)
document["undo"].bind("click", undo)
document["redo"].bind("click", redo)
</script>

<textarea id="zone" rows="4" cols="50"></textarea><textarea readonly disabled id="enoz" rows="4" cols="50"></textarea>
<button id="enter">append</button>
<button id="undo">undo</button>
<button id="redo">redo</button>
<div id="result"></div>
