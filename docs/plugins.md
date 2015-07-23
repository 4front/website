---
layout: docs
title: Plugins
submenu: plugins
menu: docs
lang: en
---

4front is architected around Express [middleware](http://expressjs.com/guide/using-middleware.html) and [routers](http://expressjs.com/4x/api.html#router). The [`req.hostname`](http://expressjs.com/api.html#req.hostname) is used to identity which virtual app an HTTP request corresponds to. Then a custom router is constructed in memory providing an isolated pipeline specific to the virtual app tenant.

Platform extensibility is achieved through plugins which are nothing more than Express middleware modules that conform to a very [common design pattern](http://bites.goodeggs.com/posts/export-this/#higher_order_function) where a higher order function accepting a single `options` object parameter returns a middleware function taking three arguments: `(req, res, next)`.

~~~js
module.exports = function(options) {
	return function(req, res, next) {
		// Return a response using res.send, res.json, etc.
		// or take some action and invoke next().
	};
};
~~~

## Router Setup
Rather than defining routes in JavaScript code, with 4front, you declare them with JSON in the package.json file. Here's what the basic structure looks like:

~~~js
{
	"_virtualApp": {
		"router": [
			{
	 			"module": "plugin-name",
	 			"path": "/path-to-mount",
	 			"method: "get",
	 			"options": {
		 			"option1": "some_value",
					"option2": "env:SOME_ENV_VAR"
		 		}
	 		}
	 	]
	}
}
~~~

The router property contains an array of middleware declarations that logically translate to JavaScript code. The example above roughly equates to the following:

~~~js
router.get("/path-to-mount", require("addon-name")({
	option1: 5,
	option2: virtualApp.env.SOME_ENV_VAR
}));
~~~

<div class="doc-box doc-info" markdown="1">
The properties of the `options` object are restricted to types that can be represented in JSON, namely numbers, strings, booleans, arrays, and objects, but __not__ functions.
</div>

Just like when building Express apps, middleware can choose to return a response (skipping any subsequent middleware), or somehow alter the `req` object and invoke `next` to pass control along to the next module.

If you omit the `method` parameter, then the plugin middleware will be mounted using the `router.use()` rather than `router.METHOD()`. This works with or without a `path` parameter.

The example below demonstrates the different combinations of valid plugin declaration parameters:

~~~js
router: [
	{
		// Invoke for all requests
		"module": "express-session"
	},
	{
		// Invoke for all requests to /public/* regardless of method
		"module": "public-plugin",
		"path": "/public"
	},
	{
		// Invoke for GET requests to /get-time
		"module": "get-time-plugin",
		"path": "/get-time",
		"method": "get"
	}
]
~~~


<div class="doc-box doc-info" markdown="1">
4front registers middleware in the same sequence as declared in the router JSON array, so just like in Express, <strong>order matters</strong>.
</div>

Through composition of different router stacks, it's possible to run everything from simple static HTML websites to sophisticated single page apps with authentication, secure API integration, database integration, and more.

## Default Router
If there is no package.json file or the router array is empty, the virtual app will automatically be configured with an instance of the [webpage add-on](/docs/plugins/webpage.html). Out of the box, this middleware provides basic static hosting of web assets, i.e. html, js, css, etc. However it can be configured to do much more including auth checks, HTML5 pushState, and more. If no router configuration is provided in the package.json, the app is automatically configured for basic static hosting.

## Environment Variables
Oftentimes it is desirable to not hardcode add-on option values in package.json that will be committed to source control. For example the [express-request-proxy](/docs/plugins/express-request-proxy.html) uses options for configuring things like API keys. For these scenarios 4front supports environment variables that are specified in place of the actual values. The convention is to prefix the value with `"env:"`.

~~~json
{
	"router": [
		{
 			"module": "addon-name",
 			"path": "/path-to-mount",
 			"method: "get",
 			"options": {
				"sensitiveValue": "env:SENSITIVE_VALUE"
	 		}
 		}
 	]
}
~~~

### Setting Environment Variables
Environment variables are set via the CLI with the `set-env` command:

~~~sh
$ 4front set-env --key SENSITIVE_KEY --value "some_sensitive_value"
~~~

The environment variable value can also differ between virtual environments, i.e. Test, Production, Dev, etc. See the [CLI docs](/docs/cli.html#set-env) for more details.


## Built-In Plugins
4front comes packaged with a small set of built-in plugins that provide a full-featured static hosting solution.

* [webpage](/docs/plugins/webpage)
* [authorized](/docs/plugins/authorized.html)
* [redirects](/docs/plugins/redirects.html)
* [custom-errors](/docs/plugins/custom-errors.html)
* [logout](/docs/plugins/logout.html)
* [client-config](/docs/plugins/logout.html)

## 3rd Party Plugins
These middleware modules have been explicitly tested with the 4front virtual router:

* [express-request-proxy](/docs/plugins/express-request-proxy.html)

## Custom Plugins

Here's an example of a very basic `get-time` custom plugin that simply returns the current server time in a JSON response:

~~~js
require("date-format-lite");

module.exports = function(options) {
	return function(req, res, next) {
		res.json({time: new Date().format(options.format)});
	};
};
~~~

The plugin would be declared in package.json like so:

~~~json
"_virtualApp": {
  "router": [
    {
      "module": "get-time",
      "path": "/get-time",
      "method": "get",
      "options": {
        "format": "UTC:hh:mm"
      }
    }
  ]
}
~~~

## Installing Plugins
To register an plugin, define an environment variable named `FF_PLUGINS` set to a comma-delimited list of the node_modules to install. Each value in the list can utilize any of the formats recognized by the npm CLI [install command](https://docs.npmjs.com/cli/install).

~~~sh
export FF_PLUGINS="4front/express-request-proxy,4front/http-basic-auth"
~~~

For custom plugins, you can also point to a private NPM repo or git repo:

~~~sh
export FF_PLUGINS="git+https://git.company.net/4front/custom-plugin.git"
~~~

Plugins are installed automatically with NPM upon application initialization. You can always modify the environment variable which will cause Elastic Beanstalk to re-install plugins.
