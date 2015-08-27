---
layout: docs
title: Configuring Auto-Reload
menu: docs
lang: en
---

The 4front dev sandbox server exposes a `/autoreload` endpoint that you can invoke passing along a list of changed files. This will send a message to your browser causing the page to reload. If the changed file is a linked stylesheet, then the stylesheet will be automatically reloaded without incurring a full page refresh. This provides a really productive workflow by providing instant visual feedback when fiddling with CSS styles.

### Configuring
To enable the auto-reload functionality, just set `autoReload` to `true`
in the `_virtualApp` section of your [package.json](http://4front.io/docs/package-json).

~~~json
"_virtualApp": {
  "autoReload": true
}
~~~

This will cause 4front to inject the following script block into all web pages
when in development mode:

~~~html
<script src='//localhost:3000/static/autoreload.js'></script>
<script>
  window.__autoReload = new AutoReload({port:3000});
  window.__autoReload.watch();
</script>
~~~

You then need to configure a watch task that monitors for changes and invokes the `/autoreload` endpoint on the sandbox server. The easiest way to do this is with [gulp-watch](https://www.npmjs.com/package/gulp-watch) or [grunt-contrib-watch](https://www.npmjs.com/package/grunt-contrib-watch).

Here's an example using Gulp:

__package.json__

~~~json
{
  "scripts": {
    "watch": "gulp watch"
  },
  "_virtualApp": {
    "autoReload": true
  }
}
~~~

__gulpfile.js__

~~~js
var gulp = require('gulp');
var watch = require('gulp-watch');
var request = require('request');

gulp.task('watch', function() {
  // Watch specific files for changes and trigger an autoreload
  gulp.watch(["*.html", "css/*", "js/*"], function(event) {
    request({
      url: 'https://localhost:3000/autoreload/notify',
      method: 'post',
      json: {
        files: [event.path]
      }
    });
  });
});
~~~

If you have additional build steps to perform, such as compilation of [Sass](http://sass-lang.com/) to css, you would configure it like so:

__gulpfile.js__

~~~js
var gulp = require('gulp');
var watch = require('gulp-watch');
var request = require('request');
var path = require('path');
var sass = require('gulp-sass');
var gutil = require('gulp-util');

gulp.task('sass', function () {
  gulp.src('./sass/**/*.scss')
    .pipe(sass().on('error', sass.logError))
    .pipe(gulp.dest('./css'));
});

gulp.task('watch:sass', function () {
  gulp.watch('./sass/**/*.scss', ['sass']);
});

gulp.task('watch:static', function() {
  gulp.watch(["*.html", "css/*", "js/*"], function(event) {
    gutil.log('autoreload', event.path);
    request({
      url: 'https://localhost:3000/autoreload/notify',
      method: 'post',
      json: {
        files: [event.path]
      },
      strictSSL: false
    });
  });
});

gulp.task('watch', ['watch:static', 'watch:sass']);
~~~

While you can still use something like [LiveReload](http://livereload.com/) or [BrowserSync](http://www.browsersync.io/), but their is some extra configuration and is more problematic when used in conjunction with https. Because the built-in auto-reload is embedded in the sandbox server, the self-signed certificate warning will already have been dealt with.
