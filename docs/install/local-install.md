---
layout: docs
title: Installing Locally on OSX
menu: docs
submenu: install
lang: en
---

This guide walks through the process of getting 4front up and running on a local OSX developer machine.

### Virtual App Host
You'll need to choose a top level hostname that will serve as the URL of your platform. You can choose anything you like so long as it has a `.dev` top level domain. The instructions below assume `4front.dev`. Virtual apps deployed to the platform are accessed via a URL in the form: `appname.4front.dev`.

### Pow.cx
[Pow.cx](http://pow.cx/) is a zero config server for OSX from [37 Signals](https://37signals.com/). It provides an extremely simple way to get a web server running that routes incoming `.dev` http requests to an app running on the local machine. Although it is designed for Rack apps, it can also route traffic to a process running on any arbitrary port number, in our case the `4front-local` node process listening on port 5000.

To install Pow simply run:

~~~sh
$ curl get.pow.cx | sh
~~~

Now create a file in the `~/.pow` directory with a single line corresponding to the port number. This assumes the 4front node app will listen on port 5000 and `4front.dev` is the hostname.

~~~sh
$ echo 5000 > ~/.pow/4front.dev
~~~

### DynamoDB Local
The production version of 4front is designed to run on AWS using DynamoDB as a metadata store. Fortunately for local development there exists [DynamoDb Local](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html) which can be installed via Homebrew:

~~~sh
$ brew install dynamodb-local
~~~

Follow the instructions to automatically launch upon startup with `launchctl`.

### Redis

Redis is used for caching various things including session state and developer sandbox static assets.

~~~sh
$ brew install redis
~~~

Once again follow the instructions to automatically launch it upon startup with `launchctl`.

## Installation
Now we can install `4front-local` itself:

~~~sh
$ npm install 4front/local -g
~~~

<a id="starting-4front-platform"></a>

### Starting the 4front Platform

Now you are ready to fire up the server:

~~~sh
$ 4front-local
~~~

If you used a hostname other than `4front.dev`, pass it as an argument:

~~~sh
$ 4front-local -h myhostname.dev
~~~

You should be able to load the 4front portal in your browser:

[http://4front.dev/portal](http://4front.dev/portal)

Since this is a local test installation, you can login with any username and password you like. A real deployment would be configured to use  an identity provider like Active Directory.

### Install the CLI

~~~sh
$ npm install -g 4front/cli
~~~

Setup your local instance as a profile:

~~~sh
$ 4front add-profile --profile-name local --profile-url http://4front.dev
~~~

Finally go ahead and create your first app!

~~~sh
$ 4front create-app
~~~
