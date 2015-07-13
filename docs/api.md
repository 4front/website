---
layout: docs
title: API
menu: docs
lang: en
---

4front offers a comprehensive REST API for managing all aspects of the platform. Both the portal and the CLI are consumers.

## Endpoint
The url for the API is `https://yourvirtualhost.com/api` where "yourvirtualhost" is the value specified at the time of installation in the `FF_VIRTUAL_HOST` environment variable. This is the same hostname used for addressing virtual apps, but without the second-level domain qualifier.

## Authentication
Authenticating with the API is done via [JSON web tokens](https://en.wikipedia.org/wiki/JSON_Web_Token) (aka JWTs). First issue a request to the login endpoint to obtain a JWT, then pass it along in subsequent requests in the `X-Access-Token` header.

~~~sh
$ curl -H "Content-Type: application/json" \
  -X POST -d '{"username":"bob","password":"xyz"}' \
  https://your4fronthost.com/api/profile/login
~~~

This returns a JSON response like so:

~~~json
{
  "userId": "123",
  "username": "bob",
  "jwt": {
    "token": "encrypted_token",
    "expires": 1436820041097
  }
}
~~~

The token is good for 30 minutes or whatever the value of the `FF_JWT_TOKEN_EXPIRE` environment variable. The token should be passed along in subsequent API requests in the `X-Access-Token` header.

~~~sh
$ curl -H "X-Access-Token: <jwt_token>" https://your4fronthost.com/api/profile
~~~

## Operations
* [Applications](/docs/api/applications.html)
* [Organizations](/docs/api/organizations.html)
* [Versions](/docs/api/versions.html)
* [Profile](/docs/api/profile.html)
* [Environment Variables](/docs/api/env-vars.html)
