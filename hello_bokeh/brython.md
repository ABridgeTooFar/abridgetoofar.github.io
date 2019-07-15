---
layout: default
title: Hello Bokeh!
---
<h1>Example of a Bokeh plot</h1>
<p>Pretty cool!!</p>
<div id="myplot" ></div>
<div id="thyplot"></div>
<script type="text/python">
from browser import document, window
import random

# 'importing' the library
Bokeh = window.Bokeh
plt = Bokeh.Plotting

# create some data and a ColumnDataSource
lx = [-0.5 + i * (20.5 - -0.5)/10.0 for i in range(10) ]
ly = [ v * 0.5 + 3.0 for v in lx]
sourcel = Bokeh.ColumnDataSource.new({ "data": { "x": lx, "y": ly } });

# create some ranges for the plot
xdr = Bokeh.Range1d.new({ "start": lx[0], "end": lx[-1] });
ydr = Bokeh.Range1d.new({ "start": -0.5, "end": 20.5 });

# make the plot
plot = Bokeh.Plot.new({
    "title": "BokehJS Plot",
    "x_range": xdr,
    "y_range": ydr,
    "plot_width": 400,
    "plot_height": 400,
    "background_fill_color": "#F2F2F7"
});

# add axes to the plot
xaxis = Bokeh.LinearAxis.new({ "axis_line_color": None });
yaxis = Bokeh.LinearAxis.new({ "axis_line_color": None });

plot.add_layout(xaxis, "below");
plot.add_layout(yaxis, "left");

# add grids to the plot
xgrid = Bokeh.Grid.new({ "ticker": xaxis.ticker, "dimension": 0 });
ygrid = Bokeh.Grid.new({ "ticker": yaxis.ticker, "dimension": 1 });
plot.add_layout(xgrid);
plot.add_layout(ygrid);

# make the plot and add some tools
tools = "pan,crosshair,wheel_zoom,box_zoom,reset,save"
p = plt.figure({'title': "Simple Line Graph", 'tools': tools})
#p.add_layout(xgrid);
#p.add_layout(ygrid);

#add a Line glyph
p.line({"x": lx, "y": ly,
    "line_color": "#666699",
    "line_width": 2
})
p.y_range=ydr

# show the plot
mydiv = document['myplot']
#thydiv = document['thyplot']
plt.show(p, mydiv.elt)
#Bokeh.Plotting.show(plot,thydiv.elt);
</script>
