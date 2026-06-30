---
layout:     post
title:      "Ejemplo de Javascript Ajax"
subtitle:   "javascript trick"
date:       2010-05-15 12:20:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       es
lang-ref:   javascript-ajax-example
---

Code

{% highlight javascript %}
function createXMLHttpRequest() {  
	try { return new XMLHttpRequest(); } catch(e) {}  
	try { return new ActiveXObject(“Msxml2.XMLHTTP”); } catch (e) {}  
	alert(“XMLHttpRequest not supported”);  
	return null;  
}  
function reloadGPSData() {  
	var xhReq = createXMLHttpRequest();  
	xhReq.open(“GET”, “gps.php?mode=ajax”, true);  
	xhReq.onreadystatechange = function() {  
		if (xhReq.readyState != 4) { return; }  
		if ( xhReq.status &gt;= 400 ) {  
			alert(xhReq.status + ‘: ‘ + xhReq.statusText);  
			return;  
		}  
		if ( xhReq.status == 0 ) {  
			alert(“Imposible conectar con el servidor.”);  
			return;  
		}  
		var json_text = xhReq.responseText;  
		var data = JSON.parse(json_text);  
		var latitude = data[0];  
		var longitude = data[1];  
		var satellites = data[2];  
		var error_desc = data[3];  
		document.getElementById(“longitude”).value = longitude;  
		document.getElementById(“latitude”).value = latitude;  
		document.getElementById(“satellites”).value = satellites;  
		document.getElementById(“error_desc”).innerHTML = error_desc;  
		setTimeout(‘reloadGPSData()’,1000);  
	}  
	xhReq.send(null);  
}  

{% endhighlight %}