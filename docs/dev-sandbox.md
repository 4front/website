---
layout: docs
title: Developer Sandbox
menu: docs
lang: en
---

The developer sandbox is a hybrid environment where the index html page is served from the 4front platform, but all client assets are loaded from a `localhost` server. As JavaScript and CSS files are modified locally, the CLI will auto [livereload](http://livereload.com/) the browser enabling a rapid development experience. If any index pages are saved to disk in the local working directory, the CLI will automatically upload them to the 4front platform and trigger a livereload. Because the app is being run from the platform host, the developer's private sandbox instance will execute all of the same server functionality as when the app is running outside the sandbox.

When the sandbox is started from the CLI, a browser window will be launched with the virtual environment specified like so: `http://appname--dev.platformhost.com`. The sandbox middleware only kicks in if the virtual environment is set to `dev`.

## Usage

~~~js
var virtualAppHost = require('4front-apphost');
var app = express();

app.use(virtualAppHost.devSandbox({
	cache: redis.createClient(),
   	ensureAuthenticated: function(req) {
   		return req.ext.appHasAuthentication;
   },
   htmlprep: {}
}));
~~~



