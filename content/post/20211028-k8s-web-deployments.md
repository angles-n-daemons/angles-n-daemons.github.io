---
title: "Kubernetes Web Deployments, a love story"
date: 2021-03-30T21:37:13-04:00
tags: ["kubernetes", "incident"]
categories: ["software"]
---

### The wrong tool for the job?
Kubernetes is quickly becoming the default mechanism for engineers to manage remote deployments. Its feature-richness and ease of use make it an attractive solution for pushing work into the cloud. However, there still exist some quirks of its usage that any web developer should know before using it. This story tells of one of them.

### What happens on Deployment

For anyone who knows kubernetes, you'll know that service deployments are rolled out in sets of pods. Each time you deploy a new version of your application, the old deployment slowly scales down while the new one scales up. In a deployment, each request is likely to be routed to any given pod so that means for some period of time in a rollout, your service is going to serve two versions of code simultaneously.

When serving a web application as a kubernetes deployment you might see a sharp uptick in 404s while the new version of code is rolling out. It should also be noted that the larger the replicaset, the longer the rollout takes (and therefore the longer the negative behavior).

These 404s will directly be correlated with some high percentage of users unable to load any web page in your app. In the worst case scenario where a rollout stalls, your app could experience an outage lasting the duration of the stalled deployment.

![Seeing an uptick in 404s](/img/k8s-web-graph.png)
**Figure 1. Some orgs don't alert on 404s, because users tend to hit them all the time**

Looking at the logs, you might see something like this:
```
...
1964-03-15 8:23pm [SERVICE] paying citibike overage charges
1964-03-15 8:23pm [SERVICE] 404 no path named /static/a.minified.js <---- note this log line
...
```

What's happening is fairly obvious once you think about how the web build packages an application up. For those who don’t know how modern web app deployments work, there’s generally a build step - where all the javascript gets compiled into a file (or set of files) with a unique name and is placed in some build directory. There’s an index.html file that is also built, which references those unique build files.

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
