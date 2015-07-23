---
layout: docs
title: webpage
menu: docs
submenu: plugins
lang: en
---

This is the principal plugin for serving HTML web pages.

Generally this add-on is declared at the end of the router array without a `path` property. The webpage module will automatically map the request path to a file with the same path and a `.html` extension. For example if a request is made to `/pages/about` (and the request made it all the way to the webpage middleware), it would look for a deployed file with the path `/pages/about/html`. If the request has a trailing slash, i.e. `/pages/about/`, then it would look for the file `/pages/about/index.html`. Similarly if the request is to the root of the `/` root of the site, `/index.html` is rendered.

If the html file doesn't exist, the middleware will invoke `next` with a 404 `Error` object that will to be handled by error handling middleware, including [custom-errors](/docs/plugins/custom-errors) if declared.

### Configuration
~~~js
{
    "module": "webpage",
    "options": {}
}
~~~

### Options

__`defaultPage`__

Defaults to `"index.html"`

The name of the .html page to return for requests to the root of the site, `appname.apphost.com/`.

__`pushState`__

Boolean indicating if [HTML5 pushState](http://www.staticapps.org/articles/routing-urls-in-static-apps) is enabled on the server. If `true`, route all requests without a file extension to the value of the `defaultPage` option. If pushState is enabled, then no 404 errors will be returned for extension-less URLs. 404 pages would need to be handled by the client JavaScript framework.

__`htmlprep`__

Default is `true`.

Boolean indicating if the page should be piped through the [htmlprep](https://github.com/4front/htmlprep) pre-processor before being piped on to the http response. If set to an object, it is passed as the options to htmlprep.

__`clientConfigVar`__

Defaults to `"__config__"`.

The name of the global variable that is injected into the `<head>` of the page exposing a virtual app config data to client script. Has no effect if `htmlprep` is set to `false`.

~~~html
<script>window.__config__ = {appName: "the-app-name", ...};</script>
~~~

__`canonicalRedirects`__

Defaults to `false`.

Detect non-canonical permutations of a url and issue a 301 redirect to the canonical equivalent. The canonical URL is assumed to be extension-less and all lowercase. So for example, 301 redirect a request to `/About.html` to `/about`.

## htmlprep

4front comes bundled with [htmlprep](https://github.com/4front/htmlprep), a super fast, lightweight HTML post-preprocessor that can perform a number of transforms such as injecting blocks of HTML at prescribed positions and rewriting asset URLs on the fly to point to a CDN. The reason for a run-time post-processor is to allow for dynamic variation of the page based on the current user or other variables. But it's not intended as a full blown server templating engine. Tasks such as compiling views would need to be part of the client build process using something like Gulp or Grunt.

One of the key functions of htmlprep is rewriting the paths to static assets to an absolute CDN url that incorporates the virtual appId and versionId as additional path segments. This is a process known as [fingerprinting](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#invalidating-and-updating-cached-responses), a best-pratice for maximizing use of the client browser cache. The path to the CDN is configured for the 4front installation with the `FF_DEPLOYED_ASSETS_PATH` environment variable.

For example if the following code appeared in the original source HTML:

~~~html
<script src="js/main.js"></script>
~~~

the actual rendered HTML sent to the browser would look like (where "12345" is the virtual appId and "absd3" is the versionId):

~~~html
<script src="//company.cdn.com/12345/absd3/js/main.js"></script>
~~~

Note that only assets with relative paths in the HTML source are impacted by this, absolute paths are ignored.

<div class="doc-box doc-warn" markdown="1">
This functionality requires that the `htmlprep` option __not__ be set to `false`.
</div>

## HTTP Headers
The html-page appends several custom HTTP headers to the response:

* `virtual-app-version`
* `virtual-app-page`

Additionally the `Content-Type` is automatically set to `"text/html"`.
