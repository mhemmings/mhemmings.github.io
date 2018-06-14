---
layout: post
title: Moving to Serverless Go in Minutes
description: Move your current Go HTTP application to run on Lambda and API Gateway with ease using Up
---

The upsides of serverless architecture for many workloads are huge. Moving architectures is usually a far from simple task, but with [Up](https://up.docs.apex.sh/) you can be launching the Go applications you already know how to build onto serverless with ease!

## Prerequisites

You'll need an AWS account with your CLI credentials [set up](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

## Example Go Server

Here we have the awesome Go application you've lovingly built and deployed on expensive bare metal.

```go
package main

import (
  "fmt"
  "log"
  "net/http"
  "os"
)

func main() {
  http.HandleFunc("/hello", helloHandler)
  log.Fatal(http.ListenAndServe(":8080", nil))
}

func helloHandler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, %s!", os.Getenv("APP_HELLO"))
}

```

To keep things simple with the Up tooling, you'll want to ensure your file is named `main.go`.

## Install Up

Firstly, we need to install Up. TJ Holloway provides a simple script you can download and run to do this:

`curl -sf https://up.apex.sh/install | sh`

To verify you're _up_ and running, `up version` will tell you what version you've just installed.

## Listen on PORT

For many simple applications, the only change you'll need to make is support the `PORT` environment variable from the Up proxy.

Changing main to support this, while still supporting your hardcoded port is easy:

```go
func main() {
  http.HandleFunc("/hello", helloHandler)

  port := os.Getenv("PORT")
  if port == "" {
    port = "8080"
  }

  addr := fmt.Sprintf(":%s", port)
  log.Fatal(http.ListenAndServe(addr, nil))
}
```

## Prepare Config

If we run

`up`

in our main package directory, we will be taken through a wizard to set us up for deployment, and your application will shortly (first run will take a few seconds longer to provision everything) be live and ready to use! For the simplest of applications, this may be all you need to do!


`up url` will show you where your application now lives. Visiting this URL by running `curl $(up url)/hello`, shows us that something is missing however...

## Add environment variables

Normally the first run of anything more complex than our example will come crashing down. You'll have all sorts of environment variables defined that are critical for your application to run. In this case however, we're just missing `APP_HELLO`.

Up makes it super easy to add and manage the environment, and variables can be added to `up.json` as simple string key/values. Our `up.json` now needs to look something like this:

```json
{
  "name": "serverless-in-minutes",
  "environment": {
    "APP_HELLO": "world"
  }
}
```

Running `up` again will redeploy our application with the new environment variable, and hey presto `curl $(up url)/hello` shows us that we're up and running as we should be, but with zero servers!

You now have access to all of Up's awesome features such as logging and metrics. Just run `up help` to see more, and check out [the docs](https://up.docs.apex.sh/)!

## But I have a real app...

Our sample application is clearly not as complex as one in the real world. However, you'll be surprised how many applications the above steps will work for. If your app can be run with `go run`, it's likely you'll now be live on serverless.

There are however common pitfalls that you need to bear in mind.

### Encrypted Environment Variables

Many of your variables are likely to be secrets you don't want to check-in to source control with `up.json`. Up Pro offers the ability to add encrypted environment variables using the `up env` command.

### Database connections

We are cheating a little bit with Up. This instance of our application can only deal with one HTTP request at a time. If more come, Lambda will horizontally scale with more instances of our application, each dealing with 1 request.

Firstly, that means things like connection pools will likely be out of the question. Secondly, you need to be _very_ careful with connections, and a small number of database drivers require you to close each one every time, rather than hold one waiting for your next request.

### Go routines

Do you have something like `go track(request)` after you've sent an API response, to track user behaviour? Not anymore. As soon as you send a response back from your app, Lambda will freeze it until the next invocation. You'll need to refactor to do all tasks during invocation, or offload background tasks to something outside of Lambda.

### Cold starts

If your application sees uneven traffic patterns, you may suffer from slow "cold starts". Luckily, Up Pro combats this with active warming.

## Summary

Up is an incredible tool to get going with serverless architecture, allowing you to get started with little change to you current application and knowledge of Go HTTP. However, we're not harnessing all the advantages of serverless, and as you start to scale you may want to check out tools at a lower level of abstraction, such as [AWS SAM](https://github.com/awslabs/serverless-application-model).
