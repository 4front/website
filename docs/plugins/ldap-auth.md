---
layout: docs
title: ldap-auth Plugin
menu: docs
submenu: plugins
lang: en
---

The [4front-ldap-auth plugin](https://www.npmjs.com/package/4front-ldap-auth) provides an easy and secure mechanism to require end users to authenticate with an LDAP compliant identity provider (such as Active Directory) in order to access all or portions of a 4front app. In your [package.json](/docs/package-json.html), you simply declare what route patterns should be password protected and implement a login HTML page.

The plugin takes care of all the typical auth flows such as:

* Ensuring a valid authenticated session is present for all protected routes
* Redirecting to the login URL if the user is not authenticated
* Establishing a server session and encrypted session cookie upon login
* Returning the user back to the originally requested URL after logging in
* Optionally ensure that user is a member of a designated group in order to access a URL and return a 403 error if not.

### Implementation steps

Ensure that the parent 4front instance has been configured for LDAP - see the [Single Sign-On guide](/docs/single-signon.html) for details.

#### Configuration
The ldap-auth plugin needs to work in conjunction with several other plugins to provide a complete auth flow including:

* [session](/docs/plugins/session.html) &mdash; provides session functionality to track the logged in user state.
* [authorized](/docs/plugins/authorized.html) &mdash; used to specify which URL patterns require authentication.
* [logout](/docs/plugins/logout.html) &mdash; provides a logout endpoint that wipes out the session and redirects back to a login page.

Here's a sample manifest that demonstrates all these plugins working together:

~~~json
"_virtualApp": {
  "router": [
    {
      "module": "session",
      "options": {
        "timeoutMinutes": 7200
      }
    },
    {
      "module": "logout",
      "path": "/logout",
      "method": "GET"
    },
    {
      "module": "4front-ldap-auth",
      "path": "/auth",
      "method": "POST",
      "options": {
        "successUrl": "/",
        "failureUrl": "/?loginfailed=1"
      }
    },
    {
      "module": "authorized",
      "options": {
        "loginPage": "login.html",
        "routes": [
          {
            "path": "/*",
            "allowed": {
              "groups": ["Marketing Leads", "Marketing Admins"]
            }
          }
        ]
      }
    }
  ]
}
~~~

#### Login page
The other step is to implement a login html page with a form that POSTs to the path the 4front-ldap-auth plugin is mounted (`/auth` in the example above). The names of the form fields by default should be `username` and `password`.

~~~html
<form method="post" action="/auth">
  <input type="text" name="username" />
  <input type="password" name="password" />
  <input type="submit" value="Submit" />
</form>
~~~
