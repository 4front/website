---
layout: docs
title: webpage
menu: docs
submenu: router
lang: en
---

Middleware route for serving web pages. Multiple instances can be declared to match different path patterns with varying options. Works both for html pages as well as meta-pages like sitemap.xml or robots.txt. Assumes extension-less URLs. Can be configured to redirect `.html`, trailing slash, and non-lowercase URLs to their canonical lowercase extension-less equivalents.

4front comes bundled with [htmlprep](https://github.com/4front/htmlprep), a super fast, lightweight HTML post-preprocessor that can perform a number of basic transforms such as injecting blocks of HTML at prescribed positions and rewriting asset URLs on the fly to point to a CDN. The reason for a run-time post-processor is to allow for dynamic variation of the page based on the current user or other variables.

### Configuration
~~~js
{
    "module": "webpage",
    "path": "/*",
    "options": {}
}
~~~

### Authentication Sample
A common scenario is to have a site where some pages require authentication and others do not. With 4front, it's as easy as declaring two different webpage entries in the virtual app router array.

The order in which entries appear is critical:

* First the [express-session](https://github.com/expressjs/session) middleware is declared.
* Next is the [auth-check](/docs/router/auth-check.html) module. This will set the `req.ext.isAuthenticated` property based on the presence of a `session.user` object.
* Next comes the webpage entry matching paths requiring authentication, in this case any page nested beneath `/private`.
* Finally a catch-all route handles anything that makes it through.

~~~js
{
	"module": "express-session",
	"options": {
		"secret": "env:EXPRESS_SESSION_SECRET"
	}
},
{
    "module": "auth-check"
},
{
    "module": "html-page",
    "path": "/private/*",
    "options": {
        "auth": true,
        "noAuthUrl": "/login"
    }
},
{
    "module": "html-page",
    "path": "/*",
    "options": {}
},

~~~

### Options

####`defaultPage`

Defaults to `"index.html"`

The name of the .html page to return for requests to the root of the site, `appname.apphost.com/`.

####`cacheControl`

Defaults to `"no-cache"`.

Value to return as the [Cache-Control](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en#cache-control) header.

####`maxAge`

Shorthand equivalent to setting `cacheControl` option to `max-age=X`.

####`contentType`

The value to return in the `Content-Type` header. Defaults to `text/html`.

####`auth`

Default is `false`.

Boolean indicating if the user must be authorized to access this URL.

####`noAuthPage`

The name of the page to render if `auth` is true but `req.ext.isAuthenticated` is not true.

<div class="doc-box doc-warn" markdown="1">
Because the URL doesn't actually change with this option, be sure to to leave the `maxAge` option to the default value of `no-cache`.
</div>

####`noAuthUrl`

Rather than rendering the `noAuthPage`, redirect to this URL when the user is not authenticated.

####`assetPath`

Defaults to `"/static"`.

The path where static asset references in the HTML should be re-written to point at. For a CDN, this would be a full domain name like  `"//company.cdn.com/assets"`. The value must use the double leading slash to identify it as an absolute path. This is considered a best-practice as it will automatically inherit the protocol of the parent page.

In addition to the assetPath, the virtual appId and versionId will appear as additional path segments in the final rendered asset URL. This is a process known as [fingerprinting](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#invalidating-and-updating-cached-responses), a best-pratice for maximizing use of the client browser cache. For example if the following code appeared in the original source HTML:

~~~html
<script src="js/main.js"></script>
~~~

the actual rendered HTML sent to the browser would look like (where "12345" is the virtual appId and 21 is the versionId):

~~~html
<script src="//company.cdn.com/12345/21/js/main.js"></script>
~~~


If a CDN is not in use, then the [static-asset](#static-asset) route should also appear in the virtual router list. Whatever value is used for the `path` option should be repeated here. For example if the static-asset route is configured like so:

~~~js
{
    module: 'static-asset',
    path: '/asset/*'
}
~~~

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

Note that only assets with relative paths in the HTML source are impacted by this, absolute paths are ignored.

<div class="doc-box doc-warn" markdown="1">
This functionality requires that the `htmlprep` option __not__ be set to `false`.
</div>

####`pushState`

Boolean indicating if [HTML5 pushState](http://www.staticapps.org/articles/routing-urls-in-static-apps) is enabled on the server. If `true`, route all requests without a file extension to the value of the `defaultPage` option. If pushState is enabled, then no 404 errors will be returned for extension-less URLs. 404 pages would need to be handled by the client JavaScript framework.

####`htmlprep`

Default is `true`.

Boolean indicating if the page should be piped through the [htmlprep](https://github.com/4front/htmlprep) pre-processor before being piped on to the http response. If set to an object, it is passed as the options to htmlprep.

####`clientConfigVar`

Defaults to `"__4front__"`.

The name of the global variable that is injected into the `<head>` of the page exposing a virtual app config data to client script. Has no effect if `htmlprep` is set to `false`.

~~~html
<script>window.__config__ = {appName: "the-app-name", ...};</script>
~~~

## HTTP Headers
The html-page appends several custom HTTP headers to the response:

* __virtual-app-version__
* __virtual-app-page__

Additionally the `Content-Type` is automatically set to `"text/html"`.

### Non-Html pages
