---
layout: docs
title: Developer Sandbox
menu: docs
lang: en
---

The developer sandbox is a private instance of the application hosted on the 4front platform at a URL like `http://appname--dev.apphost.com`. It's actually a hybrid environment where some requests are served by the 4front platform and others from localhost. The goal is to provide the best of both worlds: a fully integrated environment that closely mirrors production, along with the smooth and rapid development workflow of localhost.

### How it works
When developers run the [dev command](/docs/cli#dev) the following tasks are carried out:
1) Examine the `scripts` section `package.json` for a `buildDebug` (or `buildRelease` in release mode) and/or `watch` command. If they exist, run them in that order.
2) Launch a localhost server listening on port 7000 (or an override).
3) Open a new browser to `http://appname--dev.apphost.com`.

There are two different types of http requests in sandbox mode: web pages and static assets. Each is handled a bit differently:

#### Web Pages
When a web page is requested, the `dev-sandbox` middleware intercepts the request and performs an interstitial redirect to localhost. The localhost server check if it has a more up to date version of the requested page. If so, it uploads it to 4front and redirects back to the original `--dev` url which then renders the page.

In the first redirect to localhost, a hash parameter is passed along representing the contents of the file that 4front has cached. If the CLI determines that the hash of the local version matches the parameter, the file is considered unchanged. In this case the upload step is skipped and immediately performs the second redirect.

While it may sound complex, the experience should feel very natural, particularly when combined with LiveReload described further down.

#### Static Assets
With static assets such as JavaScripts or stylesheets, no redirect loop is necessary. Instead relative assets paths are rewritten by 4front (using the [htmlprep](https://github.com/4front/htmlprep) module) to point directly to localhost. 

### LiveReload
Live


The paths for static assets are rewritten to

 	this URL is automatically launched in a new browser tab. At the same time a localhost server is started. When a web page is requested, 4front redirects to

It also starts a localhost server.  In `dev` mode, the 4front platform rewrites all relative assets paths to `localhost:7000`. In this way the sandbox is a hybrid environment where web pages are served from 4front, and static assets from localhost. When a webpage is requested  also starts a localhost server where all  There is also a localhost server running on the

When a webpage is requested, the platform will r

 environment where html pages are served remotely from the 4front platform, but all client assets are loaded from a `localhost` server. As JavaScript and CSS files are modified on the developer's machine, the CLI will auto [livereload](http://livereload.com/) the browser enabling a rapid development experience. If any index pages are saved to disk in the local working directory, the CLI will automatically upload them to the 4front platform and trigger a livereload. Because the app is being run from the platform host, the developer's private sandbox instance will execute all of the same server functionality as when the app is running outside the sandbox.

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
