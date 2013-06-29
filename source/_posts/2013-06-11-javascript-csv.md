---
layout:     post
title:      "Reading CSV file with Javascript"
date:       2013-06-11 20:43
categories: [javascript,csv]
published:  true
---

In a specific project, the client want to read a CSV file, process and show data into an OpenLayers Map.

So it come in my mind to write a simple html/javascript application, without any backend server.

The unique constraint was how can I read the CSV file in browser.

{% codeblock read-csv.js lang:javascript %}
function handleFiles(files) {
	getAsText(files[0]);
}

function getAsText(readFile) {
	var reader = new FileReader();
	// Read file into memory as UTF-16      
	reader.readAsText(readFile);
	// Handle errors load
	reader.onload = loadHandler;
	reader.onerror = errorHandler;
}

function loadHandler(event) {
	var csv = event.target.result;
	processData(csv);             
}

function processData(csv) {
    var allTextLines = csv.split(/\r\n|\n/);
    var lines = [];
    for (var i=0; i<allTextLines.length; i++) {
        var data = allTextLines[i].split(';');
            var tarr = [];
            for (var j=0; j<data.length; j++) {
                tarr.push(data[j]);
            }
            lines.push(tarr);
    }
	console.log(lines);
}

function errorHandler(evt) {
	if(evt.target.error.name == "NotReadableError") {
		alert("Canno't read file !");
	}
}
{% endcodeblock %}


{% codeblock index.html lang:html %}
	<script type="text/javascript" src="static/js/read-csv.js"></script>
	<input type="file" id="csvFileInput" onchange="handleFiles(this.files)"
            accept=".csv"></div>
{% endcodeblock %}