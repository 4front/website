---
layout: docs
title: Installing on AWS
menu: docs
submenu: install
lang: en
---

## TODO

## Provisioning AWS Resources
The platform leverages multiple AWS services including: ElasticBeanstalk, ElastiCache, S3, and DynamoDB. For most of these resources there are CloudFormation templates to automate the provisioning. S3 is not included because buckets are created manually by the cloud team.

In the `environments` directory are environment specific sub-directories corresponding to each 4front instance, i.e. non-prod-external, non-prod-internal, etc. In this context the terms "internal" and "external" refer to web apps that are employee facing within the Nordstrom network vs. ones that are public customer facing. Each resource provisioned via CloudFormation has a template script which does not change from environment to environment, and a parameters file which includes the environment specific values.

A shell script is provided to run the CloudFormation template for a specified environment that invokes the [AWS CLI](http://aws.amazon.com/cli/). These commands assume that a profile is setup in the [AWS credentials file](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) called "nordstrom-federated".

For example you can run commands like the following from within the `cloudformation/install` directory:

~~~sh
$ bash elastic-beanstalk-env.sh nonprod-internal
$ bash elasticache.sh prod-external
~~~

## Deploying Platform
Visit http://4front-platform-versions.s3-website-us-west-2.amazonaws.com/aws/ and download the latest version.

Open the AWS Elastic Beanstalk console and click the "Upload and Deploy" button. Upload the zip file you just downloaded (the Version label should auto-populate).
