---
title: "Kubernetes Web Deployments, a love story"
date: 2021-03-30T21:37:13-04:00
tags: ["kubernetes", "incident"]
categories: ["software"]
---

_ Note: The names in this story are modified to as to not disclose the identity of those involved. _

### Background

In my working history I've seen some incidents that spark joy in the frightening edge cases that they illuminate. This short tale is about web deployments with kubernetes, and a lesser known evil of sending an orchestrator to do a CDN's job.

### Situation

I saw a ping one day in our engineering channel that our web deploy was stuck. Having come recently from a back end k8s shop, I quickly assured the group that although the stalled deploy had some unhealthy pods - kubernetes wouldn't forward traffic to a pod that didn't pass a healthcheck. In simpler terms - our deployment was healthy, and we didn't have anything to worry about because **Deployment A** (the previous deployment) would continue to serve most of the traffic until **Deployment B** (the new deployment) came up.

Crisis averted right? It would seem that way until around 10 minutes later, when my manager popped in rather frantically to say the site was down.

"The site is down?" I mused uncomfortably. Looking a little closer at how our services were deployed internally - it seemed like there were a few proxy layers between k8s ingress to our downstream pods and I wondered if any of these proxies could forward traffic to an unhealthy pod.

### Root Cause Analysis

By this point we had fixed the stuck deployment and the outage had ended, but I still wanted to figure out what the root of the issue was.

Soon after, I met with the platform team to iron out what had happened. They quickly debunked my strange proxy to a dead pod theory by showing me that the proxies beyond the ingress and deployment were a separate service, deployment and sidecar in a third deployment (seriously why do we do this to ourselves?) While we were speculating as to what could have gone wrong - Claire on the call pointed out a graph that caught my eye.

![Seeing an uptick in 404s](/img/k8s-web-graph.png)
**Figure 1. A graph of the outage showing that web requests dropped significantly**

> "Huh...we were getting about x 404s per second during this outage weren't we..."

...

> "Well I wonder what the errors were"

And like that we jumped to the logs.
```
...
1964-03-15 8:23pm [SERVICE] chasing a scooter
1964-03-15 8:23pm [SERVICE] 404 no path named /static/a.minified.js
1964-03-15 8:23pm [SERVICE] my cache way too full dog
...
```

And in that moment we had realized what had happened. For those who don't know how modern web app deployments work, there's generally a build step - where all the javascript gets compiled into a file (or set of files) with a unique name and is placed in some build directory. There's an index.html file that is also built, which references those unique build files.

Because these build names will be unique between deployments (and therefore containers) a request for index.html to Deployment A - will trigger a request for a.minified.js by the browser. If that request gets routed to a Deployment B pod the load will fail, because B will only have b.minified.js. In this way, kubernetes safe rollout behavior became an ironically fatal quirk in our deployment.

![The issue with two live deployments](/img/k8s-web-ascii.png)

**Figure 2. Because there was no guarantee requests would stick to pods by client, a user could request static assests from the wrong deployment**

### What you should know

Just because the stuck deployment caused an outage, it doesn't mean that smooth deployments escape this behavior. On most deployments there is some period where the new deployment is scaling up and the old one is scaling down where you are likely to experience this issue. The longer the scale up period, the longer the outage window - so if you're thinking about deploying a web application to kubernetes think long and hard about the availability requirements and deployment frequency.


### Epilogue

Katrina, a seasoned and intelligent engineeer on my team suggested two paths for mitigation in the future that we will investigate:
- .Rolling out via Blue/Green deployments so that we can have a 100% switch on.
- .Somehow enabling sticky traffic to pods.

We're not sure exactly what the fix is for now, but we do better understand the problem which is the first step towards any solution.
