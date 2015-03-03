---
layout: docs
title: Plugins
menu: docs
lang: en
---

Platform extensibility is achieved through plug-ins. Plugins are nothing more than node modules that follow a very [common design pattern](http://bites.goodeggs.com/posts/export-this/#higher_order_function) in [Connect middleware](http://www.senchalabs.com/connect) whereby a higher order function accepting a single `options` object parameter returns a middleware function taking three arguments: `(req, res, next)`. 

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

The plugin repository should reside in the filesystem outside of the directory where the platform code is located. Here's one sensible way to organize your 4front installation:

~~~
.
├── app.js
├── platform
│   └── node_modules
│       ├── 4front-api
│       └── 4front-apphost
└── plugins
    ├── node_modules
    │   ├── custom-plugin
    │   └── express-api-proxy
    └── package.json
~~~

Plugins are installed with a simple `npm` command:

~~~
npm install custom-plugin --save
~~~

Once installed on the target platform, they can be declared in the _virtualApp.middleware section of package.json:

~~~js
"_virtualApp": {
	"middleware": [
		"name": "custom-plugin",
		"options": {
			"option1": 4,
			"option2": true
		}
	]
}
~~~