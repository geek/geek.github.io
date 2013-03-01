---
layout: post
title: "Node HTTP Response Splitting"
description: ""
category: "node"
tags: ["Node"]
---

Web application developer should be aware of the HTTP Response Splitting threat.  Fortunately, there have efforts to add protections against this threat in the core Node.js code.  That being said, the protections are not widespread and there is a possibility for them to be bypassed.

## Overview of HTTP Response Splitting

Within a single HTTP response message header it is possible to terminate the response headers with carriage return line feed (CRLF) characters and send multiple response [message headers](http://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html).   As a result, when an attacker has control of any part of the HTTP response header they are able to trick a browser into rendering their response instead of the one that the developer intends to send.  What is more, since the response comes from the same origin as the web application the attacker has access to any origin dependent properties, namely cookies.

To learn more about HTTP response splitting please visit the Web Application Security Consortiums (WASC) Threat Classification [page](http://projects.webappsec.org/w/page/13246931/HTTP%20Response%20Splitting).

## Splitting a Response

The following is a perfectly valid HTTP Response to a single request.


	HTTP/1.1 200 OK
	Content-Length: 0
	 
	HTTP/1.1 200 OK
	Content-Type: text/html
	Content-Length: 19
	 
	<html>HACKED</html>
	Date: Sat, 02 Feb 2013 18:35:04 GMT
	Connection: keep-alive

The above response was generated using only the `ServerResponse.prototype.writeHead` function.  Below is the code that generates the previous response (Node.js 0.8.18).

	var http = require('http');
	 
	http.createServer(function (req, res) {
	  res.writeHead(200, { 'Content-Length': '0\r\n\r\nHTTP/1.1 200 OK\r\nContent-Type: text/html\r\nContent-Length: 19\r\n\r\n<html>HACKED</html>' });
	  res.end();
	}).listen(8000, '127.0.0.1');

When Safari makes a request to this server it will render the message body from the second part of the response (figure 1.1).

<img src="/assets/img/fig-1-1.png" style="max-width: 638px" />

Figure 1.1

## Browser Defenses

In Chrome there is a warning that will appear for responses that have duplicate headers.  The previous example will not trigger the Chrome warning, but if you remove the double CRLF after the Content-Length header you will see the warning.  Below is the server code to generate a response that causes Chrome to display this warning.

	var http = require('http');

	http.createServer(function (req, res) {
	  res.writeHead(200, { 'Content-Length': '0\r\nHTTP/1.1 200 OK\r\nContent-Type: text/html\r\nContent-Length: 19\r\n\r\n<html>HACKED</html>' });
	  res.end();
	}).listen(8000, '127.0.0.1');
		
The error that is displayed by Chrome is shown below in figure 1.2.

<img src="/assets/img/fig-1-2.png" style="max-width: 638px" />

Figure 1.2

The above error only shows in Chrome, other browsers do not have similar messages for this response.  Even though the protection exists, it’s not reliable to depend entirely on it.  When users encounter an error message like this one, they are likely not going to understand what it means.  Furthermore, the attacker only needs to add the double CRLF to terminate the first response header for Chrome to resume the assumption that there are two valid responses to the single request.

## Vulnerability Examples

The previous example didn’t demonstrate how an attacker is able to compromise a response; instead it demonstrated how sloppy coding is able to split a response.  In the following we will explore several vulnerabilities that result from allowing untrusted data to enter the HTTP response head.  It should be noted at this point that there are Node.js defenses that were added in versions 0.8.20 and 0.9.4.  However, these defenses do not exist in Node.js versions prior to 0.9.4 or 0.8.20.  The next section will explore these protections and how they mitigate most response splitting attacks. 

## Location Header

While in the previous examples we looked at creating an entirely new HTTP status line and response headers, this is not always necessary to split a response.  Instead, if an attacker controls a header value and the Content-Length value hasn’t been sent the attacker can prematurely send the response body.  In other words, the attacker sets the Content-Length to that of their custom message body and then sends their body above the response the application intends to send.  This is easily demonstrated using the Location header. 

Typically in a RESTful application after a resource is created a 201 response will be sent to the client along with the URI of where the new resource can be found.  If the resource name is in the URI and is not encoded before being added to the Location header value then the application is vulnerable to HTTP response splitting.  Below is an example of an application that meets these criteria.

	var http = require('http');
	var url  = require("url");
	 
	http.createServer(function (req, res) {
	  var item = url.parse(req.url, true).query.item;
	  res.writeHead(201, { 'Location': 'http://127.0.0.1:8080/' + item });
	  res.end();
	 
	}).listen(8000, '127.0.0.1');
		
Now when a request comes in with the following URL it will split the response.

http://localhost:8000/create?item=%0d%0aContent-Length:%205%0d%0a%0d%0asplit

Figure 1.3 below shows the output from the request.

<img src="/assets/img/fig-1-3.png" style="max-width: 638px" />

Figure 1.3

The entire response is still sent to the client, only the browser is told to only render the first 5 characters of the body.  If the request is changed to have a Content-Length of 11 the headers after Location will be rendered (Figure 1.4 below).

<img src="/assets/img/fig-1-4.png" style="max-width: 638px" />

Figure 1.4

## Node.js Protections

The latest versions of node have protections to strip out any CRLF sequences found within a header value.  Please note that this doesn’t protect against header fields themselves, only their values.  Of course, it is bad practice to even consider setting any header field itself from a client input.  That being said, special care should still be taken to ensure that all header fields are encoded and properly escaped.

Below is the code in that is used to protect against response splitting in Node.js, it can be found in the http module.

	if (/[\r\n]/.test(value))
  		value = value.replace(/[\r\n]+[ \t]*/g, '');
  			
The above code is found in Node.js versions 0.8.20+ and 0.9.4+.  It replaces a CR or a LF along with any following spaces or tabs with an empty string.  This means that you cannot bypass the protection with a payload like \r\r\n\n as every unwanted character is removed.  Also, double encoding CRLF into %250D%250A won’t work either, as it will not be decoded after the value is set.  

## Bypassing the Protections

The standard protections that exist in Node.js are sufficient for any standard deployment.  However, there are still a couple of ways that the protection can be bypassed or even used to help an attacker.  If the attacker double encodes the CRLF characters and your setup has an intermediary that decodes the response before it reaches the client then it’s likely the application is vulnerable to response splitting.  Therefore, special care should be given to all downstream processing that occurs after the application sends a response.

Another potential threat is the ability for an attacker to smuggle an invalid header value past a validator.  This is achievable by combining the \r or \n characters with the invalid header value.  For example, if an application has a blacklisted header value of ‘HACK’ an attacker could bypass the check by making the value ‘H\nA\rC\nK’ and letting the protection code strip out the \n and \r characters.  Of course, this is a non-issue if the application uses a whitelist of allowed values.

## Recommendations

In order to help safe guard an application against HTTP response splitting, ensure that no part of any header value originates externally.  Whenever it is necessary for values to come from a client always remove any unsafe characters and encode the remaining value.

If the users of the application should never have a \r\n value in the request then perhaps, instead of allowing the http module to remove these characters, the application should return a bad request response to the client.



{% include JB/setup %}
