---
title: "Kubernetes Web Deployments, a love story"
date: 2021-03-30T21:37:13-04:00
tags: ["kubernetes", "incident"]
categories: ["software"]
---

### The wrong tool for the job?
Kubernetes is quickly becoming the default mechanism for engineers to manage remote deployments. Its feature-richness and ease of use make it an attractive solution for pushing work into the cloud. However, there still exist some quirks of its usage that any web developer should know before using it. This story tells of one of them.

### Situation

Suppose there was a web development team, who had decided to deploy their web application on kubernetes. One day this team is running a regular deployment, and notices that the deployment has stalled. While the new deployment (let’s call it Deployment B) was scaling up, half the pods failed to become healthy! Nothing to fear though, this team knows that kubernetes wouldn’t forward traffic to a pod if it didn’t pass a healthcheck. Healthy pods = healthy deployment right?

It would seem that way until around 10 minutes later, when their product manager shares with them that the site is down.

”The site is down?” one of the engineers muses audibly. Given the haphazard proxy nature of many organizations' proxy schemes, these developers’ first instinct might be to wonder if traffic could be forwarded to one of the unhealthy pods.

### Root Cause Analysis

So the web devs hop on a call with the resident experts - infrastructure, platform, networking, etc. They share this theory that maybe a proxy sidecar was forwarding traffic to an unhealthy pod and nearly get laughed out of the room. Then someone points out a graph that brings the stakeholders to a heavy hush.

![Seeing an uptick in 404s](/img/k8s-web-graph.png)
**Figure 1. You might not alert on 404s, because users tend to hit them all the time**

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

And in that moment they realize what has happened. For those who don’t know how modern web app deployments work, there’s generally a build step - where all the javascript gets compiled into a file (or set of files) with a unique name and is placed in some build directory. There’s an index.html file that is also built, which references those unique build files.

Because these build names will be unique between deployments (and therefore containers) a request for index.html to Deployment A - will trigger a request for a.minified.js by the browser. If that request gets routed to a Deployment B pod the load will fail, because B will only have b.minified.js. In this way, kubernetes safe rollout behavior became an ironically fatal quirk in their deployment.

![The issue with two live deployments](/img/k8s-web-ascii.png)
**Figure 2. Because is no guarantee requests stick to pods by client, a user could request static assests from the wrong deployment**

### What you should know

Just because the stuck deployment caused an outage, it doesn’t mean that smooth deployments evade this problem. On most deployments there is some period where the new deployment is scaling up and the old one is scaling down and anytime you have two live deployments you can face this issue. The longer the scale up period; the longer the outage window - so if you’re considering deploying a web application to kubernetes, think long and hard about the availability requirements and deployment frequency.

### What can I do?

Below are a couple of strategies for mitigating this issue. They're listed below in no particular order:

 - .serve your web application behind a CDN
 - .do blue green deployments to manage a 100% deployment cutover ([source](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/))
 - .implement traffic stickiness using your ingress deployment affinity ([source](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/))
 - .??? you're the engineer, figure it out!
