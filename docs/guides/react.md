---
layout: docs
title: React Apps
menu: docs
submenu: guides
lang: en
---

<img src="/images/react.png" style="height: 90px"/>

React is a cutting edge JavaScript framework for building sophisticated web applications, particularly SPAs. We've provided a starter template [react-starterify](https://github.com/4front/react-starterify) that works seamlessly with 4front. The template provides a sophisticated Gulp powered build process pre-configured to:

* ES6 transpiling, JSX transformations, and inline sourcemap generation using [Babel](https://babeljs.io/)
* Module bundling with [Browserify](http://browserify.org/)
* [Sass](http://sass-lang.com/) compilation
* [Auto-Reload](http://4front.io/docs/guides/autoreload.html) browser assets upon save
* Release build that has been minified with [UglifyJS](http://lisperator.net/uglifyjs/)

To start a new 4front React app from the template, just run the following:

~~~sh
$ 4front create-app --template-url https://github.com/4front/react-starterify/archive/master.zip
~~~
