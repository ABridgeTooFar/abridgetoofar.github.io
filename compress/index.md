<html>
<head>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/brython/3.7.1/brython.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/brython/3.7.1/brython_stdlib.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/lz-string/1.4.4/lz-string.min.js"></script>
</head>
<body onload="brython(1)">
<form name="analyze" id="analyze" action="analyze.html" method="GET">
    <div name="stations" id="stations"></div><br />

    <input type="hidden" id="csv" name="csv" disabled="1" readonly="1" style="display:'none';" value="" />
</form>

<script>
var string = "This is my compression test.";
alert("Size of sample is: " + string.length);
var compressed = LZString.compress(string);
alert("Size of compressed sample is: " + compressed.length);
string = LZString.decompress(compressed);
alert("Sample is: " + string);
</script>

<script type="text/python">
from browser import document, window, aio

LZString = window.LZString

async def refreshPageSize():
    matrix = {}
    url2CanadianWeather = "https://anonymous:anonymous@dd.meteo.gc.ca/nowcasting/matrices/SCRIBE.NWCSTG.08.17.06Z.n.Z"
    request = await aio.get(url2CanadianWeather)#,format="application/json")
    matrix = LZString.decompress(request.data)
    document["csv"].value=matrix
    window.alert("Size of sample is: " + matrix.length)
 
def updatePageSize(ev):
    aio.run(refreshPageSize())

updatePageSize(0)
</script>
</body>
</html>
