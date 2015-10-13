---
layout: docs
title: basic-auth
menu: docs
submenu: plugins
lang: en
---

Sometimes you just need to provide a global basic username/password protection on an app. Often [HTTP basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) is a "good enough" solution. This plugin lets you secure all or part of your app with a username and password that you will likely store in
[environment variables](/docs/plugins.html#environment-variables).

## Usage

Simply declare the `basic-auth` plugin in your `package.json` manifest:

~~~json
"_virtualApp": {
  "router": [
    {
      "module": "basic-auth",
      "options": {
        "username": "env:BASIC_AUTH_USERNAME",
        "password": "env:BASIC_AUTH_PASSWORD",
        "realm": "App Realm"
      }
    },
    {
      "module": "webpage"
    }
  ]
}
~~~

The above example will protect the entire app. If you want to only require the username/password for a section, just add a `path` option to the plugin declaration. In the following example, the plugin only applies to routes nested within the `protected` directory:

~~~json
{
  "module": "basic-auth",
  "path": "/protected",
  "options": {
    "username": "env:BASIC_AUTH_USERNAME",
    "password": "env:BASIC_AUTH_PASSWORD",
    "realm": "App Realm"
  }
}
~~~
