---
title: "Your first rotation oncall as a Software Engineer"
date: 2021-12-24T21:37:13-04:00
tags: ["oncall", "pagerduty", "datadog", "prometheus"]
categories: ["software", "oncall"]
---

I think most engineers remember this first time oncall. If you're writing software at any larger organization, oncall is a way for the company to delegate critical issues and shield teams from unexpected interruptions. I found this helpful guide](https://brianjenney.medium.com/how-to-survive-your-first-on-call-rotation-as-a-software-engineer-5bc0d334566e) detailing a lot of the ideas I share, and hope that this document can serve as an affirmation and compliment to that one.

### What to do

I want to highlight the most important idea about oncall that I feel not everyone knows. Your job is not to resolve every issue. If looped into something, whether it be ping, page or partridge - your job is to find someone who can address it, not address it yourself.

As you get more rotations under your belt, you'll more and more become the person who knows how to address it - but in most cases when you start you should be making quick gut check assessments of "Do I understand why I'm being pinged / paged and what I can do about it" and if the answer is no then looping in a team lead / product manager or someone who knows how to delegate responsibilities.

This principle then leads into my next point ->

### Get a buddy

When you go oncall for the first time, ask the person adding you to the rotation if they can be your buddy or if they can find one for you. You SHOULD NOT be doing this on your own.

For any first rotation (and more realistically a second and a third) the fate of the system should not rest solely on your shoulders. Any team that does this without support has unrealistic expectations about how thin they can spread their workforce.

When you get looped into something, also loop your buddy in - so that they can help provide feedback and take over if the situation requires. Also ensure they're aware of this so that they're not caught off guard.

### Keep track of everything that you do

I like to personally keep a diary of everything I do oncall. I also recommend my team members keep track of the same. Generally this is a list of links to slack threads, and within these slack threads links to dashboards, pages, other relevant threads, direct platform changes (including manually executed database queries), etc.

This helps for two reasons:

1. It provides you an audit log of everything that was done. If you execute a faulty query - it'll show why you were instructed to. If oncall encounters the problem again, it provides them an existing pathway towards remediation.
2. It allows the people managing oncall to identify patterns and regular interruptions so that they can be prioritized  longer term.
