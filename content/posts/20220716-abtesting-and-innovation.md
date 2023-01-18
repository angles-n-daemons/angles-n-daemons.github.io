---
title: "If not careful, A/B testing will kill innovation"
date: 2022-07-16T15:34:22-04:00
tags: ["software development", "experimentation"]
categories: ["software"]
---

**Note: I substitued the phrase A/B Testing for experimentation in this document**

I recently read an article on [hacker news](https://www.zumsteg.net/2022/07/05/unchecked-ab-testing-destroys-everything-it-touches/) highlighting some of the anti-patterns of experimentation and it got me thinking about an idea that I’ve had bouncing around in my head for some time. The article begins to unwind some of the common risks with experimentation, which generally include ignoring the long term effects of experiments and only measuring part of what matters.

While both are things I feel pretty strongly about, there’s another risk for relying on a/b testing that I feel could be expanded a little further and it’s this. An overreliance on experimentation kills innovation.

In using experimentation, we’ve turned our business into an optimization function. We modify these parameters in how our product works to optimize our loss function - which is informed by a set of organizational metrics. This process of turning the business into an optimization function runs the same risks that an ML algorithm or graph search algorithm have in that it’s capable of getting caught in global minima.

When you’re focused on moving the needle ever so slightly but incremental changes, you lose sight of the greater search space - and grow averse to large jumps that may in the short term tank metrics you care about, but in the long term expand the potential for your business.

This is what makes startups so successful, you could think of them as little random restarts in this broad and complex search space. There are merits to experimentation, but don’t allow the practice to keep you from breaking out of your organizational shell and shaking things up.

