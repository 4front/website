---
layout: docs
title: Installing Locally on OSX
menu: docs
submenu: install
lang: en
---

This guide walks through the process of getting 4front up and running on a local OSX developer machine.

## Prerequisites

### Dnsmasq
A common local development setup is to use [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) to resolve any `.dev` HTTP request to the local IP `127.0.0.1`. Follow the simple steps outlined in the following article to install and configure Dnsmasq on OSX:

[Using Dnsmasq for local development on OS X](http://passingcuriosity.com/2013/dnsmasq-dev-osx/)

### Apache
Apache is used as a reverse proxy that sits in front of the 4front node app. It is used to translate incoming URLs in the form `*.4front.dev` to the port where the node app is listening, i.e. `localhost:1903`. Apache comes pre-installed on OSX, but it is also possible to use Nginx.

### DynamoDB Local
The production version of 4front is designed to run on AWS using DynamoDB as a metadata store. Fortunately for local development there exists [DynamoDb Local](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html). If you're on OSX, I recommend installing it with Homebrew using the command:

~~~sh
$ brew install dynamodb-local
~~~

Follow the instructions to automatically launch it upon startup with `launchctl`.

### Redis
Install [redis](http://redis.io/topics/quickstart). If you're on OSX, once again Homebrew is your friend: `brew install redis` and follow the `launchctl` instructions to launch on startup.

## Installation

### Install 4front-local
Install the 4front local platform from npm.

~~~sh
$ npm install 4front-local -g
~~~

### Create DynamoDB Tables
Run the following node script to create the 4front DynamoDB tables:

~~~sh
$ node ./node_modules/4front-dynamodb/scripts/create-local-tables.js
~~~

### Virtual App Host
Choose a top level hostname that will serve as the URL of your platform. The default value is `4front.dev`. Virtual apps deployed to the platform are accessed via a URL in the form: `appname.4front.dev`.

<div class="doc-box doc-info" markdown="1">
If you use a different host name, replace `4front.dev` in subsequent steps with your value (which should still have a `.dev` extension).
</div>

### Configure Reverse Proxy
This step describes how to setup an [Apache vhost](http://httpd.apache.org/docs/2.2/vhosts/) that acts as the reverse proxy for all 4front requests. These instructions are specific to Apache, but if you prefer Nginx, this [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-virtual-hosts-server-blocks-on-ubuntu-12-04-lts--3) is a good place to start.

First you'll need to ensure that Apache is setup to allow vhosts. Open the file `/etc/apache2/httpd.conf` and ensure the "Include /private..." line shown below is uncommented:

~~~
# Virtual hosts

Include /private/etc/apache2/extra/httpd-vhosts.conf
~~~

Also ensure the following line is uncommented:

~~~
LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
~~~

Now edit `/etc/apache2/extra/httpd-vhosts.conf` and insert the following block (making sure to change `ServerName` and `ServerAlias` to your virtual app host):

~~~
<VirtualHost *:80>
  RequestHeader set X-Forwarded-Proto "http"
  ServerName 4front.dev
  ServerAlias *.4front.dev
  ProxyPreserveHost  on
  ProxyPass / http://localhost:1903/
  ProxyPassReverse / http://localhost:1903
</VirtualHost>
~~~

After saving your changes, restart Apache:

~~~sh
$ sudo apachectl restart
~~~

#### Enabling SSL
<div class="doc-box doc-warn" markdown="1">
Production 4front instances should always use SSL, but for local development, this step is __optional__.
</div>

First we'll need to create a self-signed certificate. Create a directory to place the certs. This example uses `/etc/ssl`:
~~~sh
$ sudo mkdir -p /etc/ssl/private && sudo mkdir -p /etc/ssl/certs
~~~

Now generate the private and public certs. When prompted for the "Common Name", be sure to specify `*.4front.dev` (the leading *. is critical) or whatever virtual host name you've chosen:

~~~sh
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 \
    -keyout /etc/ssl/private/4front.key \
    -out /etc/ssl/certs/4front.crt
~~~

[How to Create and Install an Apache Self Signed Certificate](https://www.sslshopper.com/article-how-to-create-and-install-an-apache-self-signed-certificate.html)

When OpenSSL prompts you for a common name, enter your virtual host name as a wildcard, i.e. `*.mywebapps.dev`. Once your certificate keys have been created, add an additional `VirtualHost` section to `/etc/apache2/extra/httpd-vhosts.conf` like so:

~~~
<VirtualHost *:443>
  RequestHeader set X-Forwarded-Proto "https"
  ServerName 4front.dev
  ServerAlias *.4front.dev
  ProxyPreserveHost  on
  ProxyPass / http://localhost:1903/
  ProxyPassReverse / http://localhost:1903
  SSLEngine on
  SSLCertificateFile "/etc/apache2/ssl/server.crt"
  SSLCertificateKeyFile "/etc/apache2/ssl/host.nopass.key"
</VirtualHost>
~~~

Once again, restart Apache:

~~~sh
$ sudo apachectl restart
~~~

### Starting the 4front Platform
Now you are ready to fire up the server:

~~~sh
$ FF_VIRTUAL_HOST=mywebapps.dev node app.js
~~~

You should be able to load the 4front portal in your browser at `http://mywebapps.dev/portal`. You'll also want to install the 4front CLI and add your new local instance as a profile:

~~~sh
$ npm install -g 4front-cli
$ 4front add-profile --profile-name local --profile-url http://mywebapps.dev
~~~

Finally go ahead and create your first app!

~~~sh
$ 4front create-app
~~~
