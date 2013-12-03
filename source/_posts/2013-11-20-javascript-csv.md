---
layout:     post
title:      "Reading CSV File With Javascript and HTML5 File API"
date:       2012-11-20 20:43
categories: [javascript,csv,fileapi,html5]
published:  true
keywords: javascript, csv, fileapi, html5, read, file, html, js, api, file api
description: How to read a file via html5 and javascript, using modern browsers to instantly view and parse data in client side.
---

In a specific project, the client want to read a CSV file, process and show data into an [OpenLayers][1] Map.

So it come in my mind to write a simple html/javascript application, without any backend server.


## How to read a file via browser ?

The unique constraint was how can I read the CSV file in browser.

HTML5 finally provides a standard way to interact with local files, via the [File API][2]

We are going to use [FileReader][3]

<!-- more -->

First, let’s check if the browser support FileReader API

{% codeblock read-csv.js lang:javascript %}
	function handleFiles(files) {
	  // Check for the various File API support.
	  if (window.FileReader) {
	      // FileReader are supported.
	  } else {
	      alert('FileReader are not supported in this browser.');
	  }
	}
{% endcodeblock %}


## How to read a file via browser ?

To select a file we’re going to use a simple html `<input type="file">`

We just call handleFile to make JavaScript returns the list of selected File objects but we are going to use only one.

{% codeblock index.html lang:html %}
	<script type="text/javascript" src="static/js/read-csv.js"></script>
  	<input type="file" id="csvFileInput" onchange="handleFiles(this.files)"
            accept=".csv">
{% endcodeblock %}

##Reading the File

After we’ve obtained a File reference, we instantiate a FileReader object to read its contents into memory. When the load finishes, the reader’s onload event is fired and its result attribute can be used to access the file data.

FileReader includes four options for reading a file, asynchronously:
- `FileReader.readAsBinaryString(Blob|File)` – The result property will contain the file/blob’s data as a binary string. Every byte is represented by an integer in the range [0..255].
- `FileReader.readAsText(Blob|File, opt_encoding)` – The result property will contain the file/blob’s data as a text string. By default the string is decoded as ‘UTF-8’. Use the optional encoding parameter can specify a different format.
- `FileReader.readAsDataURL(Blob|File)` – The result property will contain the file/blob’s data encoded as a data URL.
- `FileReader.readAsArrayBuffer(Blob|File)` – The result property will contain the file/blob’s data as an ArrayBuffer object.

For our example, we want to read a CSV file, so we can read it as text. And perfom data processing after in `processData(csv)` function.

{% codeblock read-csv.js lang:javascript %}
	function handleFiles(files) {
	  // Check for the various File API support.
	  if (window.FileReader) {
	      // FileReader are supported.
	      getAsText(files[0]);
	  } else {
	      alert('FileReader are not supported in this browser.');
	  }
	}

	function getAsText(fileToRead) {
	  var reader = new FileReader();
	  // Read file into memory as UTF-8      
	  reader.readAsText(fileToRead);
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

That’s it.


### Resources

- [Source code][4] for this article.

[1]: http://openlayers.org/
[2]: http://www.w3.org/TR/file-upload/
[3]: http://dev.w3.org/2006/webapi/FileAPI/#dfn-filereader
[4]: https://github.com/MounirMesselmeni/html-fileapi