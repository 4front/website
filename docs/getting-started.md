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
* Portal

## Virtual App Host
At the core of 4front is a virtual app host acting as the multi-tenant container. Execution of the HTTP request is passed through a pipeline of Express middleware functions that is dynamiclly composed based on the unique configuration of the virtual app.

## API
The API provides REST endpoints for programmatically managing all aspects of the platform including:

* Creating virtual apps
* Deploying virtual apps
* Deleting virtual apps
* Managing users (of the platform)
* Configuring traffic control rules
* Managing organizations
* Setting app configuration and environment variables
* Registering new users (of the platform)
* Get usage stats for a virtual app

## CLI

Command line interface that wraps access to the API.

## Portal

Web based portal with dashboards and GUI administration of access control, policies, etc.
