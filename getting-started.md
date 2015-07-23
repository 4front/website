---
layout: home
title: Getting Started
menu: getting-started
lang: en
---

# Getting Started

### 1. Install and configure the CLI

~~~sh
$ npm install -g 4front-cli
~~~

Choose a 4front instance to connect to. You can use a public hosted instance such as aerobatic.io, or your own private instance - either a [local developer instance](/docs/install/local.html) on OSX, or on [your AWS account](/docs/install/aws.html).

Add a new profile to the CLI corresponding to the 4front instance you will be deploying to. This will write a credentials file `.4front.json` in your user home directory.

~~~sh
$ 4front add-profile --profile-name acmecorp \
  --endpoint https://webapp.acmecorp.net
~~~

Now login to the CLI. For a private instance on a corporate AWS VPC, you will login with your LDAP credentials. For the local developer instance, you can use any username you like as long as the password is "4front". Upon successful login a JWT will be written to your credentials file that remains valid for however long the 4front platform instance is configured (defaults to 1 hour).

~~~sh
$ 4front login
~~~

### 2. Create an Organization
If you are not already a member of an organization, you'll need to create one for your app to live in.

~~~sh
$ 4front create-organization --org-name "First Org"
~~~

### 3. Create a new app

~~~sh
$ 4front create-app
~~~

You'll be prompted to select an app name which will comprise the URL where it will be deployed, i.e. `http://appname.webapp.acmecorp.net`. Now `cd` into the newly created directory.

### 4. Preview your app locally

The following command will launch a web browser pointing to your private sandbox environment:

~~~sh
$ 4front dev
~~~

Feel free to edit the code and see your changes locally.

### 5. Deploy your app
When you're ready to deploy simply run:

~~~sh
$ 4front deploy
~~~

### 6. Go See your App
Your app is now live at `http://appname.webapp.acmecorp.net`.

### Next steps
[Learn to use plugins](/docs/plugins.html)
