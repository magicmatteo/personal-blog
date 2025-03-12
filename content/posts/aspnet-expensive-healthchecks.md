+++ 
draft = true
date = 2025-03-12
title = "Handling Expensive Health Checks"
description = "Learn how to improve health checks in .NET applications by offloading expensive dependency checks to a background service."
slug = ""
authors = ["Matthew Macdonald"]
tags = [".net", "reliability", "observability", "performance"]
series = ["HealthChecks"]
images = ["/images/expensive-healthchecks-thumbnail.png"]
+++

# Introduction

This is the 2nd part in my series about health checks in .NET web applications. If you haven't already, I'd recommend reading the first article which provides a good base to build upon and provide context in this post.

It can be found here: [Optimizing .NET Health Checks]({{< ref "/posts/aspnet-healthchecks" >}})

Something I've come across from time to time, which makes me feel uneasy, are health checks that take a decent amount of time. Im talking anywhere from 200ms up to multiple seconds. We must assume that our health check endpoint is going to be spammed quite often. In Kubernetes, your readiness and liveness probes could be running every 5-10 seconds. The more often these probes fire, the more responsive your eviction and load balancing policies can be. Add in some synthetics calling from multiple regions and you could end up with checks that run every other second or even invoked multiple times at once. So making them lightweight is imperative.

# Potential culprits

There's a number of reasons you could have a slow, or compute intensive health check. Lets explore some example culprits below.

1. **Slow downstream API's** - The downstream API's you are checking may be slow themselves for various reasons.
2. **Database integrity verification** - You may be running queries inside your database which could take a long time, depending on the amount of data.
3. **Authentication flows** - You might be performing an Oauth flow during your check which contains multiple REST calls
4. **Cloud storage verification** - You could be verifying that you can read & write to SharePoint or Blob storage, which could have slow IO.
5. **File integrity checking** - You may need to hash multiple files to verify their integrity. This is compute heavy and could take time.

Generally speaking - its going to be things that are compute intensive or contain a slower form of IO.

# Handling slow or expensive checks

Now that we know we need to keep our health endpoint nice and light, lets explore how we can tackle it.

The lightest our health endpoint can be, is to simply return a cached health report. To achieve this - we will run a background service, responsible for running the expensive health check on a cadence we are comfortable with. This background service will then cache the result of its health check, which our main health endpoint can retrieve with minimal overhead. It not only reduces the response time of our health endpoint, it also means that you only ever have one instance of the actual health check running. 

Lets go with the example of synthetics. Lets say you have a synthetic set up with your observability platform. These can be configured to hit your service - and potentially your health endpoint - from multiple PoPs (points of presence) around the world at once. Depending on the number of replicas your are running, this could mean your health endpoint is called multiple times at once. Depending on the nature of your health checks, this could be a dramatic waste of IO, threads and resources. By implementing this pattern, you will never be running your health check more than you specify. You gain control of how and when it is run. The new health endpoint will be simply returning the cached health info, which is essentially free, so they can spam it til the cows come home!

# Code time!



