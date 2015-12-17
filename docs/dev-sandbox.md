---
layout: docs
title: Developer Sandbox
menu: docs
lang: en
---

The developer sandbox is a private instance of the application hosted on the 4front platform at a URL like `http://appname--local.apphost.com`. It's actually a hybrid environment where some requests are served by the 4front platform and others from localhost. The goal is to provide the best of both worlds: a fully integrated environment that closely mirrors production, along with the smooth and rapid development workflow of localhost.

### How it works
When developers run the [dev command](/docs/cli#dev) the following tasks are carried out:

1. Examine the `scripts` section `package.json` for a `buildDebug` (or `buildRelease` in release mode) and/or `watch` command. If they exist, run them in that order.
2. Launch a localhost server listening on port 3000 (or an override).
3. Open a new browser to `http://appname--local.apphost.com`.

There are two different types of http requests in sandbox mode: web pages and static assets. Each is handled a bit differently:

#### Web Pages
When a web page is requested, the `dev-sandbox` middleware intercepts the request and performs an interstitial redirect to localhost. The localhost server check if it has a more up to date version of the requested page. If so, it uploads it to 4front and redirects back to the original `--local` url which then renders the page.

In the first redirect to localhost, a hash parameter is passed along representing the contents of the file that 4front has cached. If the CLI determines that the hash of the local version matches the parameter, the file is considered unchanged. In this case the upload step is skipped and immediately performs the second redirect.

While it may sound complex, the experience should feel very natural, particularly when combined with [auto-reload](/docs/autoreload.html).

#### Static Assets
With static assets such as JavaScripts or stylesheets, no redirect loop is necessary. Instead relative assets paths are rewritten by 4front (using the [htmlprep](https://github.com/4front/htmlprep) module) to point directly to localhost.
