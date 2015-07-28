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

### Configuration
<div class="doc-box doc-info" markdown="1">
You can configure multiple profiles enabling you to deploy different apps to different platform instances from the same CLI. The logic for selecting which profile to use is to first look for a `--profile` option, then a `FF_PROFILE` environment variable, then finally the default or first profile configured in the `~/.4front.json` credentials file.
</div>

### Authentication
Authentication utilizes JWT tokens that are valid for 30 minutes (or whatever value is configured the target 4front platform instance). After that amount of time you are required to re-authenticate and receive a refreshed token.


### Global Options

The following options can be specified with any command:

__`--debug`__

Turn on debug level logging.

__`--profile`__

Specify a specific profile from your `.4front.json` file. You can also define an environment variable `FF_PROFILE` with the name of the profile to use. This can be a more convenient way to set the value once in a terminal session and not have to re-type it every time. If no `--profile` option is specified and there is `FF_PROFILE` environment variable, then the default profile from `~/.4front.json` will be used.

__`--token`__

JWT token to authenticate with the server. This is only necessary for an unattended processes like CI jobs. If not specified the command line interface will prompt for credentials.

### Commands

* [create-app](#create-app)
* [delete-app](#delete-app)
* [dev](#dev)
* [deploy](#deploy)
* [set-env](#set-env)
* [list-env](#list-env)
* [delete-env](#delete-env)
* [list-apps](#list-apps)
* [add-profile](#add-profile)
* [remove-profile](#remove-profile)
* [set-profile](#set-profile)
* [login](#login)

###`create-app`

Create a new 4front application. You can optionally provide a `--template-url` of a zip archive to use as a starting point. If you don't specify a template, the CLI will provide the option to start from a configured set of scaffolding templates.

~~~sh
$ 4front create-app --template-url https://github.com/4front/react-starterify/master/archive.zip
~~~

###`delete-app`

Delete a 4front application. You can either specify the `--appid` option or run the command with no options from the same directory where the `package.json` file for a virtual app lives.

~~~sh
$ 4front delete-app --appid <appId>
~~~

###`dev`

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

###`deploy`

Deploy your local code as a new version. You will be prompted to provide an optional version name and message.

~~~sh
$ 4front deploy
~~~

###`set-env`

Set an environment variable. By default the specified the value will be used for all environments. However you can override a value for a specific virtual environment, i.e. dev, test, production, by specifying a `--virtual-env` option.

~~~sh
$ 4front set-env --key <SOME_ENV_VAR> --value <value> virtual-env <production>
~~~

###`list-env`

List the environment variables for the current app.

~~~sh
$ 4front list-env
~~~

###`delete-env`

Delete an environment variable, either at the global level or by specifying a `--virtual-env` option.

~~~sh
$ 4front delete-env --key <SOME_ENV_VAR> --virtual-env <test>
~~~

###`list-apps`

List all the applications. If you belong to multiple organizations you will first be prompted to select which one.

~~~sh
$ 4front list-apps
~~~

###`add-profile`

Register a new 4front profile in the `.4front.json` file. If no `--endpoint` is specified you will be prompted to enter one by the CLI.

~~~sh
$ 4front add-profile  --profile-name <profile_name> --endpoint <https://some4frontinstance.com>
~~~

###`remove-profile`

Unregister an existing 4front instance from the `.4front.json` file.

~~~sh
$ 4front remove-profile  --profile-name <profile_name>
~~~

###`login`

Login to the CLI. You usually don't have to use this, since all commands requiring authentication will automatically prompt for credentials if the JWT is missing or expired. However in the event you need to force a new login, this command is here for that.

~~~sh
$ 4front login
~~~
