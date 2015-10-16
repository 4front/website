---
layout: docs
title: lambda-invoker
menu: docs
submenu: plugins
lang: en
---

When your app requires running custom server code but the overhead of a standalone API service is overkill, this plugin allows you to invoke an [AWS Lambda](https://aws.amazon.com/lambda/) microservice directly from client JavaScript. Lambda provides a sandboxed execution environment that can run custom code without having to think about servers (much as 4front allows you to run website front-ends without thinking about servers).

## Usage

In your package.json manifest, specify the route where you want to mount calls to the Lambda function:

~~~json
{
  "_virtualApp": {
    "router": [
      {
        "path": "/microservice",
        "module": "lambda-invoker",
        "options": {
          "functionName": "lambda-microservice-name"
        }
      }
    ]
  }
}
~~~

### AWS Options

If 4front is running in the same AWS account as the Lambda function it's recommended to not to pass credentials explicilty, but rather rely on the IAM security context. In this case the IAM role profile attached to the 4front ElasticBeanstalk application should be granted the [lambda:Invoke](http://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html) action for the Lambda to be called.

However if the Lambda is in a different AWS account, then it becomes necessary to specify the access key and secret key.

~~~json
{
  "path": "/microservice",
  "module": "lambda-invoker",
  "options": {
    "functionName": "lambda-microservice-name",
    "accessKeyId": "env:AWS_LAMBDA_ACCESS_KEY",
    "secretAccessKey": "env:AWS_LAMBDA_SECRET_KEY"
  }
}
~~~


## Lambda Development

While you could create one Lambda function corresponding to each CRUD operation (in which case you would specify a `method` property alongside the `path` with a value like `GET` or `POST`), I find it cleaner to create a single Lambda function that encapsulates all the methods exposed by the microservice. This entails implementing a lightweight router within the Lambda function to direct execution based on the inbound event.

The `lambda-invoker` plugin passes the Express request object from the original ajax request as the `event` argument to the Lambda `handler` function. That way  version of the Express request object as the  (including path, params, query, body, and cookies) to the Lambda function via the `event` argument so all the http context is available to behave as an http server.

A simple example should help illustrate the design. Let's say we have a widget microservice that we want to invoke via AJAX. The 4front app's package.json config would look something like so:

~~~json
{
  "path": "/widget-api/:widgetId?",
  "module": "lambda-invoker",
  "options": {
    "functionName": "widget-microservice"
  }
}
~~~

Here we're specifying an optional `widgetId` parameter that will be present for calls to fetch a widget but not on the POST request to create a new widget. There is no `method` specified so all requests to `/widget-api` will be routed to the Lambda function regardless of verb.

The microservice would be invoked from client JavaScript like so (jQuery is for illustration purposes only):

~~~js
$.ajax({
  url: '/widget-api/' + widgetId,
  method: 'get',
  success: function(widget) {
    console.log(widget);
  }
});

$.ajax({
  url: '/widget-api',
  method: 'post',
  contentType: 'application/json',
  data: {
    name: 'first widget'
  },
  success: function(widget) {
    console.log(widget);
  }
});
~~~

The code skeleton below illustrates how the [router npm module](https://www.npmjs.com/package/router) can be used to implement lightweight [Express](http://expressjs.com/starter/basic-routing.html) style routing outside the context of an http server. This router module is in fact the same code from Express extracted out for general use by the ever-prolific lead maintainer of Express, [@dougwilson](https://github.com/dougwilson).

~~~js
var Router = require('router');

exports.handler = function(event, context) {
  // This is a reference to the Express request generated for the AJAX
  // call from your client app to the 4front server. It has many of the
  // same familiar properties as the Express request object including:
  // body, params, cookies, query, path, and originalUrl. It also has the
  // 4front specific env property. More on that in the next section.
  var proxiedRequest = event;

  var router = Router();

  router.get('/widgets/:widgetId', function(req, res, next) {
    getWidget(req.params.widgetId, function(err, data) {
      if (err) return next(err);
      res.json(data);
    });
  });

  router.post('/widgets', function(req, res, next) {
    // The body is already parsed as JSON by the plugin before
    // it's passed to the Lambda.
    createWidget(req.body, function(err, data) {
      if (err) return next(err);
      res.json(data);
    });
  });

  router.use(function(err, req, res, next) {
    // Error handler
    res.status(500).json({code: err.code});
  });

  // Create a wrapper response object that invokes
  // the lambda context.done event. This allows routes to
  // act just like they would in Express without being aware
  // of Lambda semantics.
  var res = {
    json: function(obj) {
      context.done(null, obj);
    }
  };

  // Run the proxied request through the router
  router(proxiedRequest, res, function() {
    if (arguments.length && arguments[0] instanceof Error) {
      return context.done(arguments[0]);
    }

    // We already returned from the json function
    return null;
  });
};

~~~

### Accessing 4front Context
In the Lambda function it may be necessary to access contextual information about the calling 4front app. The `event` parameter representing the proxied Express request object includes an `ext` property which is an object with a bunch of properties, the three most important of which are:

* `virtualApp` - reference to the current 4front virtual application
* `user` - the current logged in user, if there is one (for example single sign-in with the [ldap-auth plugin](http://4front.io/docs/plugins/ldap-auth))
* `env` - an object containing the environment variables specific to the current environment.

### Deploying Lambda 
A good way to automate the deployment of the Lambda function is with a [Gulp](http://gulpjs.com/) and the [node-aws-lambda](https://www.npmjs.com/package/node-aws-lambda) module. This [gist](https://gist.github.com/dvonlehman/2386826351e85fb96163) provides a fully functional gulpfile for deploying a Lambda.
