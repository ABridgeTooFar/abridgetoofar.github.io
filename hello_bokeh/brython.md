---
layout: default
title: Hello Bokeh!
---
<h1>Example of a Bokeh plot</h1>
<p>Pretty cool!!</p>
<div id="myplot" ></div>
<script type="text/python">
from browser import document, window
import random

# 'importing' the library
Bokeh = window.Bokeh
plt = Bokeh.Plotting

# create some data and a ColumnDataSource
lx = [-0.5 + i * (20.5 - -0.5)/10.0 for i in range(10) ]
ly = [ v * 0.5 + 3.0 for v in lx]
source = Bokeh.ColumnDataSource.new({
    'data': {'x': lx, 'y': ly}
})
# create some ranges for the plot
#xdr = Bokeh.Range1d.new({ "start": lx[0], "end": lx[-1] });
ydr = Bokeh.Range1d.new({ "start": -0.5, "end": 20.5 });

# make the plot and add some tools
tools = "pan,crosshair,zoom_in,zoom_out,reset,save"
p = plt.figure({'title': "Simple Line Graph", 'tools': tools})
#add a Line glyph
p.line({"x": {"field" : "x"}, "y": {"field": "y"}, "source" : source,
    "line_color": "#666699",
    "line_width": 2
})
p.y_range=ydr
#p.yaxis.axis_line_color = "green"
#p.xaxis.axis_line_color = "blue"

# show the plot
mydiv = document['myplot']
plt.show(p, mydiv.elt)
</script>
