---
layout: docs
title: Add-ons
menu: docs
lang: en
---

Platform extensibility is achieved through add-ons. add-ons are nothing more than node modules that follow a very [common design pattern](http://bites.goodeggs.com/posts/export-this/#higher_order_function) in [Connect middleware](http://www.senchalabs.com/connect) where a higher order function accepting a single `options` object parameter returns a middleware function taking three arguments: `(req, res, next)`.

~~~js
module.exports = function plugin(options) {
	return function(req, res, next) {
		// do something
	};
};
~~~

Here's a trivial `get-time` plugin that returns the server time as JSON:

~~~js
module.exports = function getTime(options) {
	return function(req, res, next) {
		res.json({time: new Date().getTime()});
	};
};
~~~

## Built-In Add-Ons
A handful of core add-ons are include

## Installing Add-ons
To register an add-on, define an environment variable named `FF_ADDONS` set to a comma-delimited list of the node_modules to install. Each value in the list can utilize any of the formats recognized by the npm CLI [install command](https://docs.npmjs.com/cli/install).

~~~sh
export FF_ADDONS="4front/express-request-proxy,4front/http-basic-auth"
~~~

For custom add-ons, you can also point to a private NPM repo or git repo:

~~~sh
export FF_ADDONS="git+https://git.company.net/4front/custom-addon.git"
~~~

Add-ons are installed automatically upon application initialization. You can always modify the environment variable which will cause Elastic Beanstalk to re-install add-ons.

## Developing with Add-ons
Add-ons are declared by virtual applications in the `_virtualApp.router` section of package.json. Following the same paradigm as Express middleware, an add-on can return an HTTP response, or perform some logic and then invoke `next` passing control on to the next piece of middleware in the pipeline.

~~~json
"_virtualApp": {
	"router": [
		"name": "addon-name",
		"options": {
			"option1": 4,
			"option2": true
		}
	]
}
~~~
