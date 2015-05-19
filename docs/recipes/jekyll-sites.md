# Jekyll Sites

4front is a great way to deploy sites and blogs built with [Jekyll](http://jekyllrb.com/), a popular static site generator from GitHub.

## Configuration
In your package.json, add `scripts` section shown below. This will instruct the 4front CLI to watch for changes in dev sandbox mode and how to build the content prior to a deploy.

~~~js
{
  "name": "jekyll-pixel",
  "version": "0.0.1",
  "scripts": {
    "build": "jekyll build",
    "watch": "jekyll build --watch"
  }
}
~~~

Additionally add the `baseDir` pointing to `_site` which is the default name of the directory that Jekyll compiles the generated site to.

~~~js
"_virtualApp": {
  "appId": "XXX",
  "baseDir": "_site",
  "router": [
    {
      "module": "webpage"
    }
  ]
}
~~~

## Editing and Deploying
Start the development sandbox by running `4front dev`. When you're ready to deploy: `4front deploy`.

## Using a Theme
There are lots of pre-build themes to start from. You can create a new 4front Jekyll app directly from a theme by picking one from a site like [http://jekyllthemes.org/](http://jekyllthemes.org/). Copy the download link and run the following command:

~~~sh
$ 4front create-app --template-url https://github.com/clayh53/tufte-jekyll/archive/master.zip
~~~

This will create a `package.json` for you with `_virtualApp` section. You will then just need to add the `scripts` and `baseDir` as described above.
