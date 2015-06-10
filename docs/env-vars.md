---
layout: docs
title: Environment Variables
menu: docs
lang: en
---

Environment variables are used in conjunction with middleware plugins to control configuration that differs between virtual environments.

A common scenario is connecting to a test or production API endpoint from the corresponding 4front virtual environment. Using the [express-api-proxy plugin](docs/plugins/express-api-proxy), the configuration would look similiar to this:

~~~js
router: [
	{
		"module: "plugin:express-api-proxy",
		"options": {
			"endpoints": {
				"customerApi": {
					"url": "env:CUSTOMER_API_URL",
					"cacheMaxAge": 5000
				}
			}
		}	
	}
]
~~~

Environment variables can be set using the CLI or the portal. Here's how the `CUSTOMER_API_URL` would be assigned for the test and production virtual environments respectively:

~~~sh
$ 4front set-env --key CUSTOMER_API_URL --value \
 "https://test.customerapi.com" --virtual-env test
~~~

~~~sh
$ 4front set-env --key CUSTOMER_API_URL --value \
 "https://customerapi.com" --virtual-env production
~~~



