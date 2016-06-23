---
layout: post
title: "What is Serverless?"
description: ""
category: "Opinion"
tags: [serverless, architecture]
---
{% include JB/setup %}

There has been a lot of buzz lately in the online tech world about a new type of architecture coined “serverless”. There are likely many definitions of what this architecture really means and I have found it to be an exciting development.

Traditionally, when people sit down to draw out a web application, often the first decision is which web server to use. In the Java world, Apache Tomcat would be a likely choice. You would download it, install it, configure it, build your application, and then deploy to it. You would typically do all of this on your local system. Now say you want to have a staging environment or deploy to production instances, the same steps need to be done. Likely firewalls have to be configured and servers need to be purchased, and then maintained and upgraded. The full monty server infrastructure that we are familiar with. Additionally, say your application load now extends beyond a single server. You need a load balancer and you need to figure out how to spread the load across your servers and scale them out. Oh wait, let’s not forget we need a backend store! Where do I download MYSQL? How do I install it? How many database servers will I need? What about replication? Too many things to keep track of so I’m just going to throw these concerns over the fence to Infrastructure or Ops.

But say you were to build this application today. Would you do things differently? You probably would. 

For starters you would probably containerize your application, which abstracts away the infrastructure details and promotes virtualization and automated deployments. This is great. But you are still dealing with the abstraction of a server.  But as long as you are careful and keep them stateless, scaling them out shouldn’t be difficult. The infrastructure will likely support auto-scaling containerized applications based on load. But as you add features and functionality to your server code, it continues to grow. It becomes fat. The code base becomes large and complicated. Perhaps we now need to think of a way to break up the server into many servers.

What if you could build an application that abstracted away the server layer? I would give it some serious consideration.

Recently, cloud platform providers like Google, Microsoft, and Amazon have begun to provide ways to build applications that are not server-centric but instead “server-less”. Rather than composing your application as servers you compose your application as stateless functions. For example, you can have a function that services your HTTP requests, a function that is triggered by changes to your persisted data, a function that is triggered on a schedule, a function that is invoked by other functions, or a function that subscribes to a message topic. Each function is independent of other functions and can be deployed and updated in isolation. Each function can also scale independently. There is no function startup phase, once It’s deployed it becomes ready to receive requests. Startup doesn’t make sense because there is no server state to initialize. As you can see, this model is very different from a server based model. It is distributed, scalable, and stateless at the core. Combine these serverless functions with other “serverless” components like a serverless datastore, then you truly can develop an application without having to think about servers.

It is yet to be determined if serverless is just another fad or will it be something that will stick around. I can see it’s applicability when it comes to encapsulating short units of work like processing an image or http request logic, but it needs to be proven whether one can successfully compose a full enterprise scale application. Personally I think you can, because given that a function is a primitive construct then you can combine primitives to make more complex constructs. I think it’s just whether or not the tooling will mature and if the community will embrace it. 

To finish, my definition of serverless is the ability to write an application above the abstraction level of a server, deploy it and have it automatically scale without managing the underlying infrastructure. I find that idea very appealing. 

### Further reading
 
Mike Roberts - [Serverless Architectures](http://goo.gl/UHALNc)   
Werner Vogel - Amazon CTO - [Serverless Reference Architectures with AWS Lambda](http://goo.gl/ZAWDYd)

*Kevin Chan is a Senior Principal Engineer in Workday’s Product Architecture and Research team.*  
