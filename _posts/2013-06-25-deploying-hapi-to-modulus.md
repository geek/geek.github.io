---
layout: post
title: "Deploying hapi to modulus"
description: "Learn how to deploy a hapi site to modulus"
category: "node"
tags: ["Node"]
---
{% include JB/setup %}

### Introduction

Deploying a site that depends on hapi is quite easy.  However, I still think it is important to walk through the process as it may not be completely obvious.

### Prerequisites

Before beginning create an account on [modulus.io](http://modulus.io).

After this you will need to create a hapi project.  To make it easy I created a simple project that you can clone.

  git clone git://github.com/wpreul/hapi-modulus.git

### Site code

The important thing to note is that modulus sets an environmental variable for the PORT that it expects your site to listen and that hapi expects the port to be a number.  Therefore, you will need to convert the `process.env.PORT` to a number in the server constructor.  Below is the code to do this:

	var Hapi = require('hapi');

	var server = new Hapi.Server(+process.env.PORT);

After this, everything is business as usual; add your routes and handlers then start the server.

### Deploying to modulus

When you are ready to deploy to modulus follow the terminal commands below to create and push your code to your modulus site.

Before beginning, install the modulus npm module by running:

1. `npm install -g modulus`
2. `git clone git://github.com/wpreul/hapi-modulus.git && cd hapi-modulus`
3. `modulus login`
4. `modulus project create`
5. `modulus deploy`


Thats it, the browser should open and your site should appear.
