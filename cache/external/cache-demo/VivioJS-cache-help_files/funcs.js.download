//
// funcs.js
//
// Copyright (C) 2006 - 2021 jones@scss.tcd.ie
//
// 14/07/06 first version
// 11/09/06 added header1()
// 11/09/06 added subcounter parameter to footer()
// 12/09/06 append = to subcounter so that summed count is displayed
// 01/01/07 added lastUpdated()
// 03/01/07 added base parameter to header() function
// 30/10/16 VivioJS
// 27/11/16 used ajax to update counter
// 07/06/21 Irish green + border-radius
// 13/07/21 added footer options parameter
//

//
// get file lastModified date
//
let date = new Date(document.lastModified);

//
// header
//
function header(subtitle) {
    d = document;
    d.writeln('<table style="margin-left:2%; width:96%; margin-top:5px">');
    d.writeln('  <tr>');
    d.writeln('    <td style="background-color:#00902a; font-size:x-large; color:white; padding-left:10px; border-radius:5px 5px 5px 5px;">VivioJS</td>');
    d.writeln('  </tr>');
    d.writeln('  <tr style="height:5px;"></tr>');
    d.writeln('  <tr>');
    d.writeln('    <td id="subtitle" style="background-color:#c0c0c0; font-variant:small-caps; font-size:large; padding-left:10px; border-radius:5px 5px 5px 5px;">' + subtitle + '</td>');
    d.writeln('  </tr>');
    d.writeln('</table>');
}

//
// lastModified
//
function lastModified() {
	var month = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
  	return date.getDate() + "-" + month[date.getMonth()] + "-" + date.getFullYear() % 100;
}

//
// ajaxCounter
//
// (options & 1) ? "" : "4 hits since Jul-21"
// (options & 2) ? output sum of all subcounters : output subcounter count
// (options & 4) ? NO counter increment : increment counter
//
function ajaxCounter(counter, subcounter, options) {

	options = options === undefined ?  0 : options;
	var url = "https://www.scss.tcd.ie/Jeremy.Jones/counter/hit.php?counter=" + counter + "&subcounter=" + subcounter + "&url=" + document.URL + "&referrer=" + document.referrer + "&options=" + options;
	var request = new XMLHttpRequest();	// {joj 7/2/22}

	request.onreadystatechange = function() {
		var td;
		if (request.readyState == 4 && request.status == 200) {
			if (request.responseText) {
				if (td = document.getElementById("cnt"))
					td.innerHTML = request.responseText;
			}
		}
	}

	request.open("GET", url, true);
	request.send(null);

}

//
// footer
//
function footer(subcounter, options) {
	options = options === undefined ?  0 : options;
    d = document;
    d.writeln('<table style="width:96%; margin-left:2%; border-spacing:0">');
    d.writeln('  <tr style="font-size:small;">');
    d.writeln('    <td id="cnt"; style="width:33%; background-color:#00902a; color:white; padding-left:10px; border-radius:5px 0px 0px 5px;"></td>');
    d.writeln('    <td style="width:33%; background-color:#00902a; text-align:center; color:white;">Copyright &copy; ' + date.getFullYear() + ' jones@scss.tcd.ie</td>');
    d.writeln('    <td style="text-align:right; background-color:#00902a; color:white; padding-right:10px; border-radius: 0px 5px 5px 0px">last updated: ' + lastModified() + '</td>');
    d.writeln('  </tr>');
    d.writeln('</table>');
    d.writeln('<br>');
	ajaxCounter("vivio", subcounter, options);
}

// eof