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

def append(value):
    lines = len(keep)
    discard[lines]=value
    keep.append(value)
    lines += 1
    showout()
    return lines

def undo():
    rv = []
    if any(keep):
        rv.append(keep.pop(-1))
        showout()
    return rv

def redo():
    lines = len(keep)
    rv = []
    if lines in discard:
        keep.append(discard[lines])        
        lines += 1
        rv.append(discard[lines] if lines in discard else "")
        showout()
    return rv
    
def appendHandler(event):
    if not '/textarea' in document["zone"].value:
        lines = append(document["zone"].value)
        document["enoz"].value=(discard[lines] if lines in discard else "")
        document["zone"].value=""
        showout()
    document["zone"].focus()

def undoHandler(event):
    rv = undo()
    if any(rv):
        document["zone"].value=document["enoz"].value="".join(rv)
        showout()
    document["zone"].focus()

def redoHandler(event):
    rv = redo()
    if any(rv):
        document["enoz"].value="".join(rv)
        document["zone"].value=""
        showout()
    document["zone"].focus()

document["enter"].bind("click", appendHandler)
document["undo"].bind("click", undoHandler)
document["redo"].bind("click", redoHandler)
</script>

<textarea id="zone" rows="4" cols="50"></textarea><textarea readonly disabled id="enoz" rows="4" cols="50"></textarea>
<button id="enter">append</button>
<button id="undo">undo</button>
<button id="redo">redo</button>
<div id="result"></div>
