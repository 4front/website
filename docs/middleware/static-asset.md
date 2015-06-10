---
layout: docs
title: static-asset
menu: docs
submenu: router
lang: en
---

Route for serving static assets like .js and .css files. If you are using a CDN, be sure to configure the `assetPath` option in the [webpage](/docs/router/webpage.html) route. 

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
