---
layout: docs
title: express-request-proxy
menu: docs
submenu: addons
lang: en
---

The [express-request-proxy](https://www.npmjs.com/package/express-request-proxy) add-on is a high performance, intelligent proxy that supports proxying AJAX requests to remote http endpoints. In addition to simple pass-through proxying, it also supports caching, parameter injection (to querystring, path, and body), as well as response transforms. In the package.json virtual router setup, you can define one or more instances of the proxy add-on. Here's a sample configuration for the [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/) taken from the [4front-forecast-demo](https://github.com/4front/forecast-demo):

## Basic Configuration
~~~json
{
  "_virtualApp": {
    "router": [
      "path": "/api/geocode",
      "module": "express-request-proxy",
      "options": {
        "url": "https://maps.googleapis.com/maps/api/geocode/json",
        "query": {
          "key": "env:GOOGLE_API_KEY"
        }
      }
    ]
  }
}
~~~

We're defining our own `/api/geocode` endpoint on our 4front app which proxies to `maps.googleapi.com`. Additionally a `key` querystring parameter is tacked onto the remote call with our Google API key which is stored in an environment variable.

### Client Code
To invoke this proxy endpoint in client JavaScript, we simply do something like so:

~~~js
$.ajax({
  url: "/api/geocode",
  data: {
    address: "Timbuktu"
  },
  success: function(data) {
    console.log(data);
  },
  error: function(data) {
    console.error(data);
  }
});
~~~

The actual URL sent off to Google would then be: `https://maps.googleapis.com/maps/api/geocode/json?key=XYZ&address=Timbuktu`. Importantly the client JavaScript never has anything to do with the API key, that remains entirely on the server-side.

## Named Parameters Route Setup
If you are proxying many different operations to a remote API, it would be tedious to setup a separate virtual route for each and every endpoint. Instead you can use named parameters to configure all these routes in one go. For example say you are communicating with a remote API that exposes the following endpoints:

* GET /widgets
* GET /widgets/123
* POST /widgets
* DELETE /widgets/123
* GET /users

You can configure named parameters with optional path segments to proxy all of these endpoints in a single add-on declaration:

~~~json
{
  "_virtualApp": {
    "router": [
      {
        "module": "express-request-proxy",
        "path": "/api-proxy/:resource/:id?",
        "options": {
          "url": "https://some-remote-api.com/:resource/:id?",
          "method": "*",
          "query": {
            "apikey": "env:SOME_REMOTE_API_KEY"
          }
        }
      }
    ]
  }
}
~~~

## Caching
To force the origin response to be cached within 4front, simple specify a `cacheMaxAge` property in the options corresponding to the number of seconds you want to hold the response in the cache. Subsequent requests to this URL will serve the cached response directly along with a `Cache-Control` header of `maxage=X` where X is the TTL of the cached response.

~~~json
{
  "_virtualApp": {
    "router": [
      {
        "module": "express-request-proxy",
        "path": "/api/cacheable",
        "options": {
          "url": "http://someapi.com/rarely-changes",
          "cacheMaxAge": "600"
        }
      }
    ]
  }
}
~~~
