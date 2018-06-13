---
layout: post
title: Go Serverless Notes - Golang NE Talk
description: A brief writeup of my Go Serverless talk from Golang North East in February 2018
---

I've written up some notes from my talk at Golang NE. They're provided as reference and may not make much sense if you didn't see me stumble through it on the night! My slides can be found below and on [Slideshare](https://www.slideshare.net/mhemmings/go-serverless-golang-ne-february-2018), with demos on [Github](https://github.com/mhemmings/golangne-feb18).

The following is basically my speaker notes written out, so apologies for any brevity or lack of sentence structure!

<div class="iframe-slideshare-4x3">
    <iframe src="//www.slideshare.net/slideshow/embed_code/key/mhIEvvj2X6tMh6"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        allowfullscreen>
    </iframe>
</div>

## Serverless?

*Slides 2-4*

We're all familiar with the term "Cloud", and that it's "just somebody else's computer". Serverless is the same, but I guess we can see it as "just somebody else's container". This is a great thing however. We can now access huge amounts of compute power across the world with little difficulty managing it or having to think about servers.

There are lots of things that can be "serverless", such as databases, but here we're talking about functions as a service, or FaaS.

Common characteristics of FaaS are:
- Code focused, with little infrastructure management
- Asynchronous, triggered by an external event that often doesn't wait around for a reply
- Short lived with no persistent state
- Massively scalable, with the ability to run thousands simultaneously across the world

The _big 3_ FaaS providers are AWS Lambda, Google Cloud Functions, and Azure Functions. Today, only Lambda natively supports Go, and this is the service we'll be focusing on.

## Benefits

*Slides 5-8*

### Price

For most (but importantly not all) workloads it is cheap. This is because you do not pay for idle time or over-provisioning. Compute power is billed in milliseconds, alongside provisioned power (RAM/CPU) and is analogous to paying your electricity bill from a calculation of kWh (i.e. power x time). Big savings can be made especially in the provision of staging/dev environments, where the cost will usually round to 0.

### Availability

Highly available, in tens of locations across the world, with automatic failover and scaling.

### Secure

The security is very fine grained with the ability to control who or what can invoke the function, and a set of permission for what the runtime can do. The environment is read-only and ephemeral, and when used as a webservice there are many features built in such as throttling and authentication.

### Easy

Due to the lack of a traditional server and built to simply "run code", they have a low barrier to entry for existing developers. Functions are easy to write, short by nature, and easy to reason about. Out of the box, you get many great features such as logging and aggregation, and tooling for monitoring and even profiling.

## Lambda Lifecycle

*Slide 11*

One of the few nuances of Lambda that often needs to be taken into consideration when building serverless software is its lifecycle. When the function is first invoked Lambda will fetch your code, setup any resources needed (such as network) and launch your function passing the invocation event to the handler. This initial launch could take many hundreds of milliseconds, or even seconds for some environments. After invocation has finished, Lambda immediately "freezes" the function, preventing it from doing any additional work whatsoever. The function remains in a "frozen" state until another event invokes it again, at which point it is "thawed" in almost instant time. If the function is left frozen for too long, it will be discarded and a "cold start" will have to be performed again. These timings are not currently documented, but it is usually within a few hours.

This behavior means that you lose some of the power of Go concurrency; all your work must be complete before a response is sent from the function. However, as the function isn't necessarily recreated every time, things can be cached locally and external connections held open (though be careful with this!).

## Apex

*Slide 15*

As you can see from our demos, we need start to use tooling to manage functions. [Apex](http://apex.run/) is a great tool from the incredibly smart TJ Holloway that let's us do this as well as providing metrics, logging, and environment management. It simply provides a simple and easy to use environment defined as JSON that behind the scenes calls the APIs provided by AWS.

## Up

*Slides 18/20*

[Up](https://up.docs.apex.sh/) builds on Apex, but is focused towards webservices. One of the main benefits of Up is you simply create a HTTP service as you normally would. It can be ran and tested as normal with the Go toolchain.

Up let's you deploy this basic HTTP service to a serverless environment in seconds, and comes with a whole host of features such as active Lambda warming (no more cold starts!), built in middleware, structured logging, and alerting. This is done by running as a small proxy inside Lambda, proxying requests to your Go binary.

## Summary

*Slides 22-24*

Serverless architecture provides some huge advantages for many workloads, but it's important you identify if yours is suitable. If it's long lived, probably not. If it's event based, for example a web service, you may be in luck.

You will lose some cool concurrency and post-event processing, and you need to make sure you're releasing and caching your resources appropriately.

You'll want to use tooling to make this as pain free as possible. If you're doing none HTTP FaaS, take a look at Apex. If you're writing a webservice, Up is a fantastic place to start. There are also other things such as the [Serverless Application Model (AWS SAM)](https://docs.aws.amazon.com/lambda/latest/dg/serverless_app.html) which will give you more power with less abstraction.

