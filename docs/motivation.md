---
layout: docs
title: Motivation
menu: docs
lang: en
---

Single page web apps powered by JavaScript frameworks such as [AngularJS](http://angularjs.org), [Ember](http://emberjs.com), [React](http://facebook.github.io/react/), and [Backbone](http://backbonejs.org/) are quickly emerging as the preferred architecture for modern web app development.

Today these apps are often implemented as a monolithic web stack bundle consisting of both dedicated server code (Java, Ruby, C#, Python, Java, etc.) for communicating with databases, performing business logic, rendering views, etc. along with the static browser assets (JavaScript, CSS) that power the rich client component. While this design works, it limits opportunities for re-use of business logic across different front-ends and reduces shipping agility as pure front-end changes need to be deployed in conjunction with back-end server code, increasing the risk of introducing defects.

At the other end of the spectrum are pure static apps where static assets are delivered from a web server like nginx or Apache, or from an S3 bucket. This approach has the benefit of forcing a service-oriented design, performs well (by virtue of a CDN), and is very cheap to operate. However the constraints imposed by exclusively static files are often insufficient for apps with demanding integration and security requirements.

4front sits between these two worlds. It provides an active server for powering functionality that is not well-suited to the browser (including enterprise integration, security, authentication, blue/green deployments, and more) with the streamlined development style of static apps. It has been designed from scratch specifically to address the needs of today's demanding single page web apps.

## Multi-tenancy

Just as VMs allow packing multiple logical machines onto a single physical one; 4front acts a virtual app container, packing multiple distinct apps onto a single server process. This allows for even greater server utilization as memory, storage, and CPU are all shared across a pool of virtual apps. For apps with large memory and CPU requirements on the server, this could be problematic as any single app could starve the others of resources.

However in a rich browser client / API powered application, the vast majority of processing is pushed either back to an API or forward to the client. The web app server is primarily left with pure I/O work which Node.js, with its asynchronous event loop, is extremely well suited. Many simultaneous I/O operations can be in-flight at once without blocking the main thread from accepting new incoming requests.

As the number of virtual apps and traffic increases, the 4front instance scales out across a cluster of multiple nodes, each a mirror replica of the original. Every server is stateless and capable of serving a request for any given virtual app, so there is no need for "sticky sessions".

## Cost Savings

There is a potential for considerable cost-savings with a multi-tenant platform, both in terms of hardware and DevOps processes. In a well utilized multi-tenant cluster, far fewer virtual machines are required than if every app had one or more dedicated VMs.

The dev-ops work for non-functional app concerns such as auto-scaling, logging, monitoring, security policies, load balancer configuration, CDN configuration, etc. is a one time investment for the core platform. Once it's up and running, delivery automation of all subsequent virtual app is essentially free; no longer does a continuous delivery pipeline have to be re-imagined everytime a new app comes along.

## Speed to Market

Ability to iterate faster. No need to redeploy API code for pure UI changes such as updating look & feel or user interaction flows. Liberate teams to experiment and get proof of concepts out to customers in less time. Reduced cost of failure, if an app isn't working, toss it out and start over again.