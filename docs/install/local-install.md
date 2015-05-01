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
A common local development setup is to use [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) to resolve all requests for a particular top level domain to the local IP `127.0.0.1`. This avoids having to create a host file entry for every new virtual app created. The `4front-local` package assumes a `.dev` top level domain.

[Homebrew](http://brew.sh/) makes installation a breeze:

~~~sh
$ brew install dnsmasq
~~~

Follow the prompt to configure `launchctl` to automatically launch dnsmasq at startup.

Create a new file at `/usr/local/etc/dnsmasq.conf`. Open it in your favorite editor and add the following line:

~~~
address=/dev/127.0.0.1
~~~

Save the file and restart Dnsmasq.

~~~sh
$ sudo launchctl stop homebrew.mxcl.dnsmasq
$ sudo launchctl start homebrew.mxcl.dnsmasq
~~~

Now test that the Dnsmasq is working correctly with the `dig` utility:

~~~sh
$ dig testing.testing.one.two.three.dev @127.0.0.1
~~~

You should get back a response something like:

~~~
;; ANSWER SECTION:
testing.testing.one.two.three.dev. 0 IN	A	127.0.0.1
~~~

Assuming that worked, next we need to configure OSX to send `.dev` DNS queries to Dnsmasq. We can take advantage of OSX support for the unix [resolver command](http://unixhelp.ed.ac.uk/CGI/man-cgi?resolver+5) which allows configuring DNS resolvers for a specific top level domain like `.dev`.

If the directory `/etc/resolver` doesn't exist, go ahead and create it:

~~~sh
$ sudo mkdir -p /etc/resolver
~~~

Run the following command to create a new `dev` file that specify that `127.0.0.1` should be used to resolve any `.dev` request.

~~~sh
$ sudo tee /etc/resolver/dev >/dev/null <<EOF
nameserver 127.0.0.1
EOF
~~~

Finally test that DNS requests for `.dev` domains resolve correctly:

~~~sh
$ ping -c 1 test.local
~~~

You should see a response similar to this:

~~~
PING test.dev (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.028 ms
~~~

You can read more on this setup here:
[Using Dnsmasq for local development on OS X](http://passingcuriosity.com/2013/dnsmasq-dev-osx/)

### Nginx
Nginx is used as a reverse proxy that sits in front of the 4front node app. It is used to translate incoming URLs in the form `*.4front.dev` to the port where the node app is listening, i.e. `localhost:1903`.

Nginx can alos be installed with Homebrew. Disregard the instructions to configure `launchctl` for now.

~~~sh
$ brew install nginx
~~~

We need nginx to listen on port 80 which requires running as root. Create a file at `/Library/LaunchAgents/homebrew.mxcl.nginx.plist` with the following contents:

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>nginx</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <false/>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/opt/nginx/bin/nginx</string>
        <string>-g</string>
        <string>daemon off;</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/usr/local</string>
  </dict>
</plist>
~~~

<div class="doc-box doc-warn" markdown="1">
Be sure the path to the nginx executable matches where Homebrew put it on your system, other instructions on the web refer to __sbin__ rather than __bin__.
</div>

Now register nginx to automatically launch at startup:

~~~ sh
$ sudo launchctl load -w /Library/LaunchAgents/homebrew.mxcl.nginx.plist
$ sudo launchctl start nginx
~~~

### DynamoDB Local
The production version of 4front is designed to run on AWS using DynamoDB as a metadata store. Fortunately for local development there exists [DynamoDb Local](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html). Once again, Homebrew to the rescue:

~~~sh
$ brew install dynamodb-local
~~~

This time do follow the instructions to automatically launch it upon startup with `launchctl`.

## Installation
Now we can install `4front-local` itself:

~~~sh
$ npm install 4front-local -g
~~~

### Virtual App Host
Choose a top level hostname that will serve as the URL of your platform. These instructions assume the default value `4front.dev`. You can choose anything you like so long as it has a `.dev` extension. Virtual apps deployed to the platform are accessed via a URL in the form: `appname.4front.dev`.

### Configure Nginx Reverse Proxy
Now we need to configure Nginx to act as a reverse proxy that routes all inbound requests to `*.4front.dev` to `localhost:1903`. Open `/usr/local/etc/nginx/nginx.conf` in your favorite editor and the following [server block](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-14-04-lts):

~~~
server {
  listen       80;
  server_name  4front.dev;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://localhost:1903;
    break;
  }
}
~~~

Now restart nginx to ensure the changes are applied:

~~~sh
$ launchctl stop nginx & launchctl start nginx
~~~

<div class="doc-box doc-warn" markdown="1">
#### Configuring SSL
Production 4front instances should always use https. To keep things simple this guide does not walk through configuring this for a local installation, but you can add an additional server in the nginx.conf listenting on port 443. This also entails configuring a self-signed certificate which you can read about [here](https://gist.github.com/jed/6147872#create-a-wildcard-ssl-certificate-for-each-project).
</div>

### Starting the 4front Platform
Now you are ready to fire up the server:

~~~sh
$ 4front-local
~~~

If you configured nginx with a virtual host other than `4front.dev`, pass it as an argument:

~~~sh
$ 4front-local -h myhostname.dev
~~~

You should be able to load the 4front portal in your browser:

[http://4front.dev/portal](http://4front.dev/portal)

Since this is a local test installation, you can login with any username and password you like. A real deployment would be configured to use  an identity provider like Active Directory.

### Install the CLI

~~~sh
$ npm install -g 4front-cli
~~~

Setup your local instance as a profile:

~~~sh
$ 4front add-profile --profile-name local --profile-url http://4front.dev
~~~

Finally go ahead and create your first app!

~~~sh
$ 4front create-app
~~~
