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

Add a new profile to the CLI corresponding to the 4front instance you will be deploying to. Here we are using [aerobatic.io](https://aerobatic.io), but the steps are the same for a private instance.

~~~sh
$ 4front add-profile --profile-name aerobatic \
  --identity-provider github \
  --platform-url https://aerobatic.io
~~~

### 2. Create a new app
Here we'll use the Jekyll starter template, but you can choose a different starter template or specify a GitHub repo.

~~~sh
$ 4front create-app --starter-template jekyll
~~~

You'll be prompted to select an app name which will comprise the URL where it will be deployed, i.e. `http://appname.aerobatic.io`. Now `cd` into the newly created directory.

### 3. Preview your app locally

The following command will launch a web browser pointing to your private sandbox environment:

~~~sh
$ 4front dev
~~~

Feel free to edit the code and see your changes locally.

### 4. Deploy your app
When you're ready to deploy simply run:

~~~sh
$ 4front deploy
~~~

### 5. Go See your App
Your app is now live at `http://appname.aerobatic.io`.

### Next steps
[Learn to use plugins](/docs/plugins.html)
