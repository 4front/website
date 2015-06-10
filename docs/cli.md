---
layout: docs
title: CLI
menu: docs
lang: en
---

The `4front-cli` let's you manage most all aspects of 4front right from the command line.

### Installation
~~~sh
npm install 4front-cli -g
~~~

The first time you attempt to run any command, you will be prompted to login and setup a new 4front instance. Multiple profiles are supported so you can work with more than one 4front platform instance from the same CLI.

Other than the `create-app` command, the CLI should be run from the root of your app directory. It will read the `appId` from the [virtualApp section](/docs/manifest.html) of the `package.json` manifest.

### Usage
~~~
$ 4front [command] [options]
~~~

### Authentication
Authentication utilizes JWT tokens that are valid for 30 minutes (or whatever value is configured the target 4front platform instance). After that amount of time you are required to re-authenticate and receive a refreshed token.


### Global Options

The following options can be specified with any command:

__`--debug`__

Turn on debug level logging.

__`--profile`__

Specify a specific profile from your `.4front.json` file.

__`--token`__

JWT token to authenticate with the server. This is only necessary for an unattended processes like CI jobs. If not specified the command line interface will prompt for credentials.


### Commands

* [create-app](#create-app)
* [dev](#dev)
* [set-env](#set-env)
* [list-env](#list-env)
* [delete-env](#delete-env)
* [list-apps](#list-apps)
* [add-profile](#add-profile)
* [delete-profile](#delete-profile)
* [set-profile](#set-profile)

####`create-app`

Create a new 4front application. You can optionally provide a `--template-url` of a zip archive to use as a starting point. If you don't specify a template, the CLI will provide the option to start from a configured set of scaffoling templates.

~~~sh
$ 4front create-app --template-url https://github.com/4front/react-starterify/master/archive.zip
~~~

####`dev`

Starts the developer sandbox environment. This will upload your app manifest to the 4front platform for the current profile, start the localhost server, and open a browser window to your private development app instance.

~~~sh
$ 4front dev
~~~

If your `package.json` specifies values in `scripts` section for `build:debug` or `watch` those will be executed in that order. The values of these entries can refer to any valid command, but will often times refer to a build tool like Gulp or Grunt.

~~~js
{
	"scripts": {
		"build:debug": "gulp build:debug",
		"watch": "gulp watch"
	}
}
~~~

You can also specify a `--release` option which will launch the app in release mode. If there is a `build:release` script specified in `package.json`, it will be executed.

~~~js
{
	"scripts": {
		"build:release": "gulp build:release"
	}
}
~~~

####`set-env`

Set an environment variable. By default the specified the value will be used for all environments. However you can override a value for a specific virtual environment, i.e. dev, test, production, by specifying a `--virtual-env` option.

~~~sh
$ 4front env:set --key SOME_API_SECRET_KEY --value 29850u234asgfasg  \
--virtual-env production
~~~

####`list-env`

List the environment variables for the current app.

~~~sh
$ 4front env:list
~~~

####`delete-env`

Delete an environment variable, either at the global level or by specifying a `--virtual-env` option.

~~~sh
$ 4front env:del --key SOME_API_SECRET_KEY --virtual-env test
~~~

####`list-apps`

List all the applications. If you belong to multiple organizations you will first be prompted to select which one.

~~~sh
$ 4front app:list
~~~

####`add-profile`

Register a new 4front profile in the `.4front.json` file. If no `--platform-url` is specified you will be prompted to enter one by the CLI.

~~~sh
$ 4front add-profile  --platform-url https://some4frontinstance.com
~~~

####`remove-profile`

Unregister an existing 4front instance from the `.4front.json` file.
