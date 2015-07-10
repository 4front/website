---
layout: docs
title: Introduction
menu: docs
lang: en
---

The 4front platform is comprised of 4 major components (naturally):

* [Virtual App Host](https://github.com/4front/apphost)
* [API](/docs/api)
* [CLI](/docs/cli)
* [Portal](/docs/portal)

## Virtual App Host
At the core of 4front is a virtual app host acting as the multi-tenant container. Execution of the HTTP request is passed through a pipeline of dynamically composed Express middleware based on the unique configuration of the virtual app. Through the power of server-side middleware, 4front enables front-end developers to build more secure, robust, and full-featured apps than otherwise possible with just pure client code.

[Learn about virtual apps](/docs/virtual-apps.html)

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

Command line interface that wraps access to the API. Let's you do stuff like:

~~~sh
$ 4front create-app
$ 4front dev
$ 4front set-env --key SOME_API_KEY --value 324l5kjk34thj
~~~

[Learn more](/docs/cli)

## Portal

Web based portal with dashboards and GUI administration of access control, policies, etc.
