---
layout: docs
title: package.json
menu: docs
submenu: package-json
lang: en
---

Rather than introduce a proprietary manifest file, and in keeping with "the Node way", 4front simply extends `package.json` with a new section named `_virtualApp`. The CLI also looks for properties in the `scripts` section which is [described below](#scripts).

If you are using a build tool like [Grunt](http://gruntjs.com/) or [Gulp](http://gulpjs.com/) to power your client asset build process, you likely have a package.json anyway. But even if you aren't using Node powered build tool (for example a Jekyll site or plain markup), the addition of a package.json doesn't have any side effects.

Here's the basic structure of a virtual app's package.json. Each property is described in detail further below.

~~~json
{
  "name": "virtual-app-name",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "watch": "gulp watch",
    "build": "gulp build"
  },
  "devDependencies": {
    "babelify": "^5.0.4",
    "gulp": "^3.8.11",
    "gulp-uglify": "^1.1.0",
    "gulp-watch": "^4.2.4"
  },
  "_virtualApp": {
    "appId": "243534534",
    "autoReload": true,
    "baseDir": {
      "debug": "build/debug",
      "release": "build/release"
    },
    "router": [
      {
        "module": "session",
      },
      {
        "module": "webpage"
      }
    ]
  }
}
~~~

### scripts

The [scripts section](https://docs.npmjs.com/misc/scripts) is a standard npm element of package.json. The [4front CLI](/docs/cli) specifically looks for 2 named scripts: `build` and `watch`.

####`build`

Specifies a script to run that builds the client assets (writing to a folder with a name like `dist`). The CLI runs this script both before launching the [dev sandbox](/docs/dev-sandbox.html) environment and as part of the [deploy command](/docs/cli.html#deploy).

~~~json
"scripts": {
	"build": "gulp build"
}
~~~

Often times the build process differs based on a debug vs. release build. For example you may only uglify and concat your scripts for release builds. In this case you can split this script property into two separate properties: `build:debug` and `build:release`.

~~~json
"scripts": {
	"build:debug": "gulp build",
	"build:release": "gulp build --release"
}
~~~

It's also common to only have a build process for release builds, and not at all for debug. In this case you would only have a `build:release` script by itself:

~~~json
"scripts": {
	"build:release": "gulp build"
}
~~~

####`watch`

Specifies a script to run to start watching for changes. If present, the CLI will run this script when the `dev` command is executed to launch the developer sandbox environment. A common technique is to simply point to your build tool's watch command, i.e. `gulp watch` or `grunt watch`.

~~~json
{
	"scripts": {
		"watch": "gulp watch"
	}
}
~~~

This is often used in conjunction with the `autoReload` option described below to enable a streamlined development workflow where the browser automatically reloads whenever specific files are changed locally. See the [Auto-Reload guide](/docs/autoreload.html) for details on getting this setup.

### _virtualApp

####`appId`

The id of the 4front virtual app. This value is set for you automatically as part of the [create-app command](/docs/cli.html#create-app).

####`autoReload`

Specifies if the auto-reload script block should automatically be injected into html pages by the [webpage plugin](/docs/plugins/webpage.html) when developing in [sandbox mode](/docs/developer-sandbox.html). Defaults to `false`. See the [Auto-Reload guide](/docs/guides/autoreload.html) for a detailed explanation of getting this configured.

<a name="basedir"></a>

###`baseDir`

This informs the CLI where the root `index.html` of your app resides. For example your `package.json` (where the CLI should be run) is in the root of your repo, but you might choose to structure things such that your `index.html` is in a `src` sub-directory.

~~~sh
.
├── src
│   └── index.html
│   ├── css
│       └── main.css
└── package.json
~~~

In this case you'd need to specify the following:

~~~json
"_virtualApp": {
	"baseDir": "src"
}
~~~

Again, oftentimes a build process writes release assets to a separate directory like `dist`. In this case you can specify an object rather than a string:

~~~json
"_virtualApp": {
	"baseDir": {
		"debug": "src",
		"release": "dist"
	}
}
~~~

If your `index.html` is at the root of your repo alongside `package.json`, then you don't have to declare any `baseDir`. Or if you have a release build process but your development `baseDir` is the repo root, then you can only specify a `release` directory:

~~~sh
.
├──index.html
├── css
   └── main.css
├── dist
   └── index.html
└── package.json
~~~

~~~json
"_virtualApp": {
	"baseDir": {
		"release": "dist"
	}
}
~~~

###`router`

An array of middleware routes. This is a bigger subject that is covered in detail in the [Plugins documentation](/docs/plugins.html).
