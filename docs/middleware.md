---
layout: docs
title: Middleware
menu: docs
lang: en
---

4front comes pre-packaged with the minimum set of middleware required to stand up a bare-bones multi-tenant platform. Additional middleware are available as stand-alone modules that can be used in conjunction with 4front or on their own merits.

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


## virtual-app-loader
Loads the correct virtual app object by parsing the `req.hostname` and extracting the second level domain name. For example if `req.hostname` is "test-app.platformhost.com" then "test-app" must be the unique virtual app name.

It's also possible for virtual apps to be requested via a custom domain like __www.custom-app-domain.com__. This requires that a CNAME record be setup resolving to "app-name.platformhost.com".

#### Usage
~~~js
app.use(virtualAppHost.appLoader({
	virtualHost: process.env['4FRONT_VHOST_DOMAIN']
}));
~~~

#### Options
__`virtualHostDomain`__

The top level domain for the platform, for example "platformhost.com". It should be configured with your DNS provider as a [wildcard DNS record](http://en.wikipedia.org/wiki/Wildcard_DNS_record) in the form "\*.platformhost.com".

It's recommended to store this string in an environment variable to enable multiple instances 4front powered by the exact same codebase.


There are two ways that the virtual app can be declared in the URL: as a second level domain

[see code](https://github.com/4front/apphost/blob/master/lib/middleware/app-loader.js)

## traffic-control

#### Configuration
~~~js
{
 	"_virtualApp": {
 		"router": [
 			{
 				"module": "traffic-control",
 				"options": {}
 				}
 			}
 		]
 	}
}
~~~

[see code](https://github.com/4front/apphost/blob/master/lib/middleware/traffic-control.js)


## authenticated
Checks for a user object in the session state. If it exists, sets `req.user` and `req.ext.isAuthenticated` to true. Relies upon the [session middleware](https://www.npmjs.com/package/express-session) being declared first.

~~~js
app.use(virtualAppHost.authenticated(options));
~~~

#### Options
__`sessionUserKey`__
The name of the key in session to look for the user object.

## logout
Destroys the session and redirects to a specified login page.

#### Configuration
~~~js
{
 	"_virtualApp": {
 		"router": [
 			{
 				"module": "logout",
 				"path": "/logout",
 				"options": {
 					"redirectTo": "/"
 				}
 			}
 		]
 	}
}
~~~

## static-asset
Route for serving static assets like .js and .css files. If you are using a CDN, be sure to configure the `assetPath` option in the [webpage](#webpage) route.

Because the asset paths are [fingerprinted](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#invalidating-and-updating-cached-responses) by the webpage middleware, very aggressive cache headers should be set. Everytime a new version is deployed, the asset paths will be modified, so there's no concern about the browser holding on to a stale version in its cache.

#### Configuration

~~~js
{
	"module": "static-asset",
	"path": "/static/*",
	"options": {}
}
~~~

#### Options
__`cacheControl`__

The value of [Cache-Control](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en#cache-control) header.

__`maxAge`__

Shorthand for setting the Cache-Control to `max-age=X`. Defaults to 86400 (24 hours), unless overriden by the `cacheControl` option.

## webpage
Serves HTML web pages.

#### Configuration
~~~js
{
	"module": "webpage",
	"path": "/*",
	"options": {}
}
~~~

#### Options

__`defaultPage`__

The name of the .html page to return for requests to `/`. Defaults to `index.html`.

__`cacheControl`__

Value to return as the [Cache-Control](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en#cache-control) header.

__`maxAge`__

Shorthand equivalent to setting `cacheControl` option to `max-age=X`.

__`auth`__

Boolean indicating if the user must be authorized to access this URL.

__`noAuthPage`__

The name of the page to render if `auth` is true but `req.ext.isAuthenticated` is not true.

<div class="alert">
Because the URL doesn't actually change with this option, be sure to to leave the `maxAge` option to the default value of `no-cache`.
</div>

__`noAuthUrl`__

Rather than rendering the `noAuthPage`, redirect to this URL when the user is not authenticated.

__`assetPath`__

The path where static asset references in the HTML should be re-written to point at. For a CDN, this would be a full domain name like  `"//company.cdn.com/assets"`. The value must use the double leading slash to identify it as an absolute path. This is considered a best-practice as it will automatically inherit the protocol of the parent page.

In addition to the assetPath, the virtual appId and versionId will appear as additional path segments in the final rendered asset URL. This is a process known as [fingerprinting](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#invalidating-and-updating-cached-responses), a best-pratice for maximizing use of the client browser cache. For example if the following code appeared in the original source HTML:

~~~html
<script src="js/main.js"></script>
~~~

then the actual rendered HTML sent to the browser would look like (where "12345" is the virtual appId and 21 is the versionId):

~~~html
<script src="//company.cdn.com/12345/21/js/main.js"></script>
~~~


If a CDN is not in use, then the [static-asset](#static-asset) route should also appear in the virtual router list. Whatever value is used for the `path` option should be repeated here. For example if the static-asset route is configured like so:

```js
{
	module: 'static-asset',
	path: '/asset/*'
}
```

then `assetPath` would be set to:

~~~js
{
	"module": "webpage",
	"path": "/*",
	"options": {
		"assetPath": "/asset"
	}
}
~~~

If no value is provided, it defaults to `"/static"`. Note that only assets with relative paths in the HTML source are impacted by this, absolute paths are ignored. This functionality requires that the `htmlprep` option __not__ be set to `false`.

__`pushState`__

Boolean indicating if [HTML5 pushState](http://www.staticapps.org/articles/routing-urls-in-static-apps) is enabled on the server. If `true`, route all requests without a file extension to the value of the `defaultPage` option. If pushState is enabled, then no 404 errors will be returned for extension-less URLs. 404 pages would need to be handled by the client JavaScript framework.

__`htmlprep`__

Boolean indicating if the page should be piped through the [htmlprep] (https://github.com/4front/htmlprep) pre-processor before being piped on to the http response. If set to an object, it is passed as the options to htmlprep. Defaults to `true`.

__`clientConfigVar`__

The name of the global variable that is injected into the `<head>` of the page exposing a virtual app config data to client script. Defaults to `__config__`. Has no effect if `htmlprep` is set to `false`.

~~~html
<script>window.__config__ = {appName: "the-app-name", ...};</script>
~~~

## error-page
Allows customizing error pages for different http status codes. Make sure this route is declared after `webpage`.

#### Configuration
~~~js
{
 	"_virtualApp": {
 		"router": [
 			{
 				"module": "error-page",
 				"options": {
 					"statusCodes": {
 						"404": "404.html",
 						"500": "500.html",
 						"401": "401.html"
 					}
 				}
 			}
 		]
 	}
}
~~~

#### Options

__`statusCodes`__

A map of HTTP status codes and the name of the page to render.


## dev-sandbox

## dynamic-router
Builds and executes a dynamic [Express Router](http://expressjs.com/4x/api.html#router) based on the configuration of the virtual app for the current http request. See the [virtual app configuration](/docs/configuration.html) specs for how the dynamic router is declared.

#### Virtual App Configuration
Here's a sample snippet from a `package.json` demonstrating a virtual app router setup:

~~~js
"_virtualApp": {
	"router": [
		{
			"module": "plugin::express-api-proxy",
			"path": "_namespace/proxy",
			"options": {
			}
		},
		{
			"path": "/_namespace/server-time",
			"verb": "get",
			"module": "plugin::server-time",
			"options": {
				"format": "utc"
			}
		}
	]
}
~~~

TODO: Also talk about a global router config

### Usage
~~~js
app.use(virtualAppHost.dynamicRouter({
	pluginsDir: process.env['4FRONT_PLUGINS_DIR']
}));
~~~

## Additional Middleware
This is a list of middleware modules that have been tested with 4front, however there is nothing inherently special about any of them, they're just ordinary Node.js modules that conform to a very common Connect middleware pattern.

* [express-api-proxy](https://github.com/4front/express-api-proxy)
* oauth
* parse-auth
*
