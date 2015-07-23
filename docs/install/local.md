---
layout: docs
title: Installing Locally on OSX
menu: docs
submenu: install
lang: en
---

This guide walks through the process of getting 4front up and running on a local OSX developer machine. This setup simulates AWS services with [DynamoDB Local](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html) and [Fake S3](https://www.npmjs.com/package/s3rver).

### Virtual App Host
You'll need to choose a top level hostname that will serve as the URL of your platform. You can choose anything you like so long as it has a `.dev` top level domain. The instructions below assume `4front.dev`. Virtual apps deployed to the platform are accessed via a URL in the form: `appname.4front.dev`.

### Pow.cx
[Pow.cx](http://pow.cx/) is a zero config server for OSX from [37 Signals](https://37signals.com/). It provides an extremely simple way to get a web server running that routes incoming `.dev` http requests to an app running on the local machine. Although it is designed for Rack apps, it can also route traffic to a process running on any arbitrary port number, in our case the 4front node process listening on port 1903.

To install Pow simply run:

~~~sh
$ curl get.pow.cx | sh
~~~

Now create a file in the `~/.pow` directory with a single line corresponding to the port number. This assumes the 4front node app will listen on port 1903 and `4front.dev` is the hostname.

~~~sh
$ echo 1903 > ~/.pow/4front.dev
~~~

### DynamoDB Local
The production version of 4front is designed to run on AWS using DynamoDB as a metadata store. Fortunately for local development there exists [DynamoDb Local](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html) which can be installed via [Homebrew](http://brew.sh/):

~~~sh
$ brew install dynamodb-local
~~~

Follow the instructions to automatically launch upon startup with `launchctl`.

### Redis

[Redis](http://redis.io/) is used for caching various things including session state and developer sandbox static assets. If you don't already have it, just run:

~~~sh
$ brew install redis
~~~

Once again follow the instructions to automatically launch it upon startup with `launchctl`.

## Installation
Clone the [4front/aws-platform](https://github.com/4front/aws-platform) repo:

~~~sh
$ git clone https://github.com/4front/aws-platform.git 4front
~~~

<a id="starting-4front-platform"></a>

### Starting the 4front Platform

Now you are ready to fire up the server; `cd` into the new 4front directory and run:

~~~sh
$ npm start
~~~

You should be able to load the 4front portal in your browser:

[http://4front.dev/portal](http://4front.dev/portal)

Since this is a local test installation, you can login with any username and the password "4front". A real deployment on AWS would be configured to use a proper identity provider such as LDAP.

### Install the CLI

~~~sh
$ npm install -g 4front/cli
~~~

Setup your local instance as a profile. This will write a credentials file at `~/.4front.json`.

~~~sh
$ 4front add-profile --profile-name local --profile-url http://4front.dev
~~~

Now login to the CLI. Your username is whatever you like and your password is "4front". This will write a JWT token to the credentials file so you don't have to login every time.

~~~sh
$ 4front login
~~~

Now create an initial organization for you apps to live:

~~~sh
$ 4front create-organization --org-name "Demo Org"
~~~

Finally go ahead and create your first app!

~~~sh
$ 4front create-app
~~~

Launch the developer sandbox with `4front dev` and run `4front deploy` to push your app.
