---
layout: post
title: "Node HTTP Response Splitting"
description: ""
category: 
tags: ["node", "security"]
---

Web application developer should be aware of the HTTP Response Splitting threat.  Fortunately, there have efforts to add protections against this threat in the core Node.js code.  That being said, the protections are not widespread and there is a possibility for them to be bypassed.

## Overview of HTTP Response Splitting

Within a single HTTP response message header it is possible to terminate the response headers with carriage return line feed (CRLF) characters and send multiple response message headers (http://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html).   As a result, when an attacker has control of any part of the HTTP response header they are able to trick a browser into rendering their response instead of the one that the developer intends to send.  What is more, since the response comes from the same origin as the web application the attacker has access to any origin dependent properties, namely cookies.

To learn more about HTTP response splitting please visit the Web Application Security Consortiums (WASC) Threat Classification page (http://projects.webappsec.org/w/page/13246931/HTTP%20Response%20Splitting).

## Splitting a Response

The following is a perfectly valid HTTP Response to a single request.

```
HTTP/1.1 200 OK
Content-Length: 0
 
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 19
 
<html>HACKED</html>
Date: Sat, 02 Feb 2013 18:35:04 GMT
Connection: keep-alive
```



{% include JB/setup %}
