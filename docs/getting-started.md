---
layout: docs
title: Getting Started
menu: docs
lang: en
---

The 4front platform is comprised of 4 major components (naturally):

* [Virtual App Host](https://github.com/4front/apphost)
* API
* CLI
* Dashboard

## Virtual App Host
At the core of 4front is a virtual app host acting as the multi-tenant container. Execution of the HTTP request is passed through a pipeline of Express middleware functions that is dynamiclly composed based on the unique configuration of the virtual app.
