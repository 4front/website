---
layout: docs
title: authorized
menu: docs
submenu: addons
lang: en
---

The `authorized` add-on verifies that the current user is authorized to access the requested resource. It depends upon an additional authentication add-on that runs earlier in the middleware pipeline which handles logging the user in and assigning the user object to `req.ext.user`. The add-on can be configured to protect all requests matching a specific path pattern which could be the entire site, or just certain paths, i.e. everything nested beneath
`/protected`. It's also possible to further restrict access based on group membership or role privileges.

The add-on differentiates between an authorization failure due to the user not being logged in vs. not having sufficient privileges. If the user is not logged in, either because they went straight to a protected URL without logging in or their session timed out, generally the best course of action is to redirect them to a login page. The add-on provides a `loginUrl` option for this purpose. A `returnUrl` cookie will be set containing the URL of the request so that the authentication add-on can return the user directly to the URL the user was attempting to access upon successful authentication.

## Usage

~~~js
{
  "_virtualApp": {
    "router": [
      {
        // Some auth-addon must be declared first
        "module": "auth-addon",
      },
      {
        "module": "authorized",
        "path": "protected/*",
        "options": {
          "loginUrl": "/login",
          "allowed": {
            "groups": ["sysadmins", "project-managers"]
          }
        }
      }
    ]
  }
}
~~~

### Options

__`allowed`__

An object with two possible properties: `roles` and `groups`. Each is an array of strings representing the allowed groups or roles. The user must have at least one possible matching value in their correspondingly names properties. This requires that the authorization add-on in use specifies the `groups` or `roles` array properties on the `req.ext.user` object.

~~~js
{
  userId: 'xyz'
  username: 'username',
  groups: ['sysadmins', 'front-end developers']
}
~~~

__`loginUrl`__

The URL to redirect to if the user is not logged in.

__`loginPage`__

The path to a .html file to render if the request path is `/` and the user is not logged in. This is relevant for single page apps where the login view is rendered from the root path `/` rather than a dedicated URL like `/login`. Note that the `loginUrl` should still be specified as "/" for single page apps that use HTML5 pushState.

## Advanced Access Control
Oftentimes a site has multiple logical sections, each with their own access requirements. In these cases you can declare multiple instances of the add-on with different options. For example, let's say your app has a public section, a logged-in section, and an admin section. Your router declaration might look something like this:

~~~js
{
  "_virtualApp": {
    "router": [
      {
        // Some auth-addon must be declared first
        "module": "auth-addon",
      },
      // Lock /protected/* down to logged-in users
      {
        "module": "authorized",
        "path": "protected/*",
        "options": {
          "loginUrl": "/login"
        }
      },
      // Only users with the admin role can access /admin/*
      {
        "module": "authorized",
        "path": "admin/*",
        "options": {
          "loginUrl": "/login",
          "allowed": {
            "roles": ["admin"]
          }
        }
      }
    ]
  }
}
~~~
