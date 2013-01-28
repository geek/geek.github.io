---
layout: post
title: "Deploying hapi to heroku"
description: "Learn how to deploy a hapi site to heroku"
category: "Node"
tags: ["Node", "hapi"]
---
{% include JB/setup %}

### Introduction

Deploying a site that depends on hapi is quite easy.  However, I still think it is important to walk through the process as is may not seem completely obvious.

### Prerequisites

Before beginning create an account on [heroku](http://www.heroku.com/) and install the [Heroku Toolbelt](https://toolbelt.heroku.com/).

After this you will need to create a hapi project.  To make it easy I created a simple project that you can clone.

	git clone git://github.com/wpreul/hapi-heroku.git

### Site code

The important thing to note is that heroku sets an environmental variable for the PORT that it expects your site to listen and that hapi expects the port to be a number.  Therefore, you will need to convert the `process.env.PORT` to a number in the server constructor.  Below is the code to do this:

	var Hapi = require('hapi');

	var server = new Hapi.Server(+process.env.PORT, '0.0.0.0');

After this, everything is business as usual; add your routes and handlers then start the server.

### Deploying to heroku

When you are ready to deploy to heroku follow the terminal commands below to create and push your code to your heroku site.

1. `heroku login`
2. `heroku create`
3. `git push heroku master`
4. `heroku ps:scale web=1`
5. `heroku open`


Thats it, the browser should open and your site should appear.
