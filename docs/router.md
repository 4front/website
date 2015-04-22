---
layout: docs
title: Virtual Router
menu: docs
submenu: router
lang: en
---

Virtual applications are nothing more than Express apps that run as sub-apps within the context of the main platform app (which is itself an Express app).

4front takes advantage of the [Express Router](http://expressjs.com/4x/api.html#router) object to provide an isolated pipeline of middleware and routes specific to each virtual app. What's different is that rather than authoring server JavaScript code to wire up the middleware, a declarative JSON manifest is used.
Through composition of different router stacks, it's possible to run everything from simple static HTML websites to sophisticated single page apps with authentication, secure API integration, database integration, and more.

## Middleware
If you understand how [Express middleware](http://expressjs.com/guide/using-middleware.html) works, you'll understand how the 4front virtual router operates. 4front comes with a small number of core built-in middleware, but many 3rd party Express/Connect middleware will work fine so long as it follows this common structure:

~~~js
module.exports = function(options) {
	return function(req, res, next) {
	}
};
~~~

<div class="doc-box doc-info" markdown="1">
The properties of the `options` object are restricted to types that can be represented in JSON, namely numbers, strings, booleans, arrays, and objects, but __not__ functions.
</div>

### Built-In Middleware
* [dev-sandbox](/docs/router/dev-sandbox.html)
* [html-page](/docs/router/html-page)
* [static-asset](/docs/router/static-asset.html)
* [custom-errors](/docs/router/custom-errors.html)
* [auth-check](/docs/router/auth-check.html)
* [logout](/docs/router/logout.html)
* [traffic-control](/docs/router/traffic-control.html)

### 3rd Party Middleware
These middleware modules have been explicitly tested with the 4front virtual router:

* [express-api-proxy](https://github.com/4front/express-api-proxy)
* [express-session](https://github.com/expressjs/session)

3rd party middleware is installed as [plugins](/docs/plugins.html). You can of course [write your own](/docs/plugins.html) plugins as well.


## App Manifest
Rather than introduce a proprietary manifest file, and in keeping with "the Node way", 4front simply extends `package.json` with a new section named `_virtualApp`. The `router` section is used to declare the middleware and routes that describe the server-side functionality of the app. Here's an example of a basic static HTML website:

~~~js
{
	"name": "virtual-app-name",
	"version": "1.0.0",
	"dependencies": {},
	"_virtualApp": {
		"appId": "243534534",
		"router": [
			{
				"module": "plugin::custom-logger",
				"options": {
					"database": "${CUSTOM_LOGGER_DATABASE_CONN}"
				},
				"environments": ["production"]
			},
			{
				"module": "static-asset",
				"method": "GET",
				"path": "/assets/*"
			},
			{
				"module": "webpage",
				"path": "/*",
				"options": {}
			}
		]
	}
}
~~~

### Declaring Middleware
The `router` attribute of `_virtualApp` is an array of middleware modules where each entry is a JSON structure with the following properties:

__`module`__

The name of the middleware module. For plugins, the name must be prefixed with `"plugin::"`. For example:

~~~js
{
	"module": "plugin::server-time",
	"path": "/server-time",
	"options" : {
		"utc": true
	}
}
~~~

__`path`__

The path pattern to match for the middleware to kick in. If omitted, the middleware will fire for all requests that make it that far down the request pipeline.

__`method`__

The HTTP method the middleware should be restricted to (GET, PUT, POST, or DELETE). If omitted the middleware will fire for all verbs (while still honoring the `path` option). If specified, translates into a call to `router[method](req, res, next)`.

__`options`__

The object used to invoke the middleware wrapper function.

__`environments`__

Optional array specifying which environments this middleware should execute in. If omitted the middleware is applicable in all environments.

<div class="doc-box doc-info" markdown="1">
__Order is important!__ This being Express, middleware is executed in the order in which they are declared so some thought needs to be put into the proper sequencing.
</div>

### Paths and Methods

When a `method` is provided, the middleware is registered as a route handler rather than as standard middleware. The example below demonstrates how the 4front virtual router registers the middleware:

~~~js
{
	"module": "html-page",
	"path": "/public",
	"options": {}
}

// Is translated to:
router.use('/public', htmlPage(options));
~~~

Now with a `method` option:

~~~js
{
	"module": "html-page",
	"path": "/public/about",
	"method": "get",
	"options": {}
}

// Translates to:
router.get("/public/about", htmlPage(options));
~~~
