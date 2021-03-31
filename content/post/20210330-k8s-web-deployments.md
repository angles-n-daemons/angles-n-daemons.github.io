---
title: "Kubernetes Web Deployments, a love story"
date: 2021-03-30T21:37:13-04:00
tags: ["kubernetes", "incident"]
categories: ["software"]
---

### Situation

Let’s say that you’re a full stack developer that works on an application that is deployed top to bottom on kubernetes. One day your team runs into an issue, your web deployment is stalled - while Deployment B was scaling up some pods stayed unhealthy! Nothing to fear though, your team knows that kubernetes wouldn’t forward traffic to a pod if it doesn’t pass a healthcheck. Healthy pods = healthy deployment right?

It would seem that way until around 10 minutes later, when your product manager pops into chat frantically to say the site is down.

“The site is down?” You muse uncomfortably. If your service deployments go through some indirection like a sidecar proxy your first instinct could be to wonder if it could forward traffic to an unhealthy pod.

### Root Cause Analysis

So you, the full stack developer hop on a call with the big guns - infrastructure, platform, networking, etc. You share your theory that maybe a proxy sidecar was forwarding traffic to an unhealthy pod and you nearly get laughed out of the room. Then one of the platform engineers mentions a graph that she had been looking at and the room falls silent:

![Seeing an uptick in 404s](/img/k8s-web-graph.png)
**Figure 1. A graph of the outage showing that web requests dropped significantly**

> "Huh...we were getting about x 404s per second during this outage weren't we..."
...
> "Well I wonder what the errors were"

And like that you jump to the logs.
```
...
1964-03-15 8:23pm [SERVICE] paying citibike overage charges
1964-03-15 8:23pm [SERVICE] 404 no path named /static/a.minified.js <---- heyo what's this??
1964-03-15 8:23pm [SERVICE] my cache way too i couldn't eat another bite
...
```

And in that moment you realized what had happened. For those who don’t know how modern web app deployments work, there’s generally a build step - where all the javascript gets compiled into a file (or set of files) with a unique name and is placed in some build directory. There’s an index.html file that is also built, which references those unique build files.

Because these build names will be unique between deployments (and therefore containers) a request for index.html to Deployment A - will trigger a request for a.minified.js by the browser. If that request gets routed to a Deployment B pod the load will fail, because B will only have b.minified.js. In this way, kubernetes safe rollout behavior became an ironically fatal quirk in your deployment.

![The issue with two live deployments](/img/k8s-web-ascii.png)
**Figure 2. Because is was no guarantee requests stick to pods by client, a user could request static assests from the wrong deployment**

### What you should know

Just because the stuck deployment caused an outage, it doesn’t mean that smooth deployments evade this problem. On most deployments there is some period where the new deployment is scaling up and the old one is scaling down and anytime you have two live deployments you can face this issue. The longer the scale up period; the longer the outage window - so if you’re considering deploying a web application to kubernetes, think long and hard about the availability requirements and deployment frequency.
