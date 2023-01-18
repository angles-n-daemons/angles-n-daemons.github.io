---
title: "A curious claim about Advertising's effect on Google"
date: 2022-12-04T17:07:42-04:00
tags: ["google", "advertising", "experimentation"]
categories: ["business"]
---

## Google, is it worse now? [short form]

In a recent Freakanomics episode about the perceived quality degradation of Google Search, Marissa Mayer makes a claim that ads made Google a more enjoyable product to Google users. This claim came as the conclusion of a long running experiment in which some users were shown ads, and those users ended up using Google 3% more.

Mayer drew the conclusion that including ads in search generated better results, and therefore made people more likely to use Google.

Without additional evidence, I suspect that ads could have led to worse results, and required users to repeat searches - which would also reflect an increase in usage. This alternative cause is representative of my experience over time - as I find that nowadays I often have to tune my search terms to find the results I'm looking for, and that the results delivered by Google are getting worse over time.

![Google related content took up 1.5+ viewports in this search](/img/google-search-results.png)
**Google related content took up 1.5+ viewports in this search**

## Background
I actually remember one of the first, if not first time I used Google. It was at Vernon Middle School in Oyster Bay, NY and someone had mentioned it as a better alternative to the existing search engines that people used. At the time, I had transitioned from Yahoo to AskJeeves - and I was pretty content with both of them until I began using Google. At the onset, I loved the simplicity and straightforwardness of Google - the page was minimal, search worked fast and delivered great results. It quickly became my search engine of choice - and over time, my entrypoint to the web.

![Google as I first remembered it](/img/google-homepage-2000.png)
**Google as I first remembered it**


Lately it’s felt like Google has been getting more and more difficult to use. The minimalism that initially drew me to the product was being rapidly diminished with inline Google products. This topic has already been covered in length, so I won’t go too deeply into it - but [I found an article which does a decent job describing why this is](https://www.washingtonpost.com/technology/2020/10/19/google-search-results-monopoly/). What I wanted to discuss here was the claim made in Freakonomics that ads make a better experience for Google.

In the podcast, I learned that concern for advertising revenue's effect on search goes all the way back to the founding of Google [in a paper written by its founders](http://infolab.stanford.edu/~backrub/google.html) (see section titled "Advertising and Mixed Motives"). In it, there’s a commonly reference quote which is:

> “we expect that advertising funded search engines will be inherently biased towards the advertisers and away from the needs of the consumers”

The common belief and reasoning behind this performance degradation is gradual due to the incentives of advertisers, Google’s primary source of revenue.

## The Study
In discussing the idea of ads affecting search performance, Marissa Mayer, a well known executive and early engineer at Google, cited an experiment that ran between 2000 and 2008. In this experiment, the users at Google were divided into two groups: one who would receive ads and one who would not.

She then shared that the users who received ads ended up using Google 3% more, and concluded that given the increased usage - ads in search results made Google a more enjoyable experience.

I believed the claimed results of the experiment, but suspected there could be a different conclusion. Hoping to uncover this story, I searched the web for references of this study - but only found references to it in articles about the Freakonomics episode I had listened to.

## Interpreting the Results
It’s difficult to take the claim that ads make Google a more enjoyable experience given those numbers at face value. This isn’t to devalue the work of the bright engineers at Google and their analysis of the information they had at hand - but rather to propose my own interpretation given the limited information that I have.

If you take Mayer’s interpretation as truth, you might believe the story that your average user when using Google, gets better results when ads are included. Because Google is delivering this user the things they want, they use Google more often in their daily life; as a result of a great search experience.

An alternative story for increased search usage could be told using the same results. A person using Google searches for something, and when ads are included the search results are worse. Because of this, they have to repeat their search, changing their question slightly and hoping to get a different answer from the search engine. They’ve increased their search usage, but it hasn’t been a better experience.

Both stories are plausible, but the latter is more inline with my experience of Google lately.

## A Math-ier Description
Let’s say that each person in a given month experiences has `z` life experiences, of which `ρ` percent generate `m` unique questions that a person thinks could be answered by Google. Using Google, the person performs an average of `k` searches for each question. Given these numbers, I assume `m * k` (number of questions, number of searches per question) generates `n` searches per month. These numbers could be 100.000 life experiences, 0.25% leading to using Google, which would be 250 questions a user googles, and 1.8 searches per question which would culminate in 450 total Google searches in a given month.

```
Variable Legend
z: number of life experiences a person has in a month
ρ: percentage of life experiences that prompt a Google search
m: number of questions that a person has for Google in a month
k: number of searches that a person performs for each question
n: number of Google searches a person performs in a month

m = z * ρ
n = m * k
```

Using the proposition that Ads make Google better, it’s likely that the `ρ` modifier grows slightly - in that more experiences in the person’s life prompt them to use Google given their good experience with the product. This raises the `m` value, and therefore the `n` number of questions that a user might ask Google in a given month.

The alternative (and my theory) is that `k` increases when you use ad incentives, that users are being forced to search more to answer each question they have. Increasing the number searches that are performed for each question increases the `k` value as well.

## Conclusions
I didn’t write this hoping that I might change Google, or uncover the truth. It’s an engineer’s curiosity to see if a statement so strong as “ads make google better” actually holds up to the data - and perhaps this article is simply a manifestation of that curiosity.

I also feel that for a long time, despite running ads - Google provided a really usable and helpful product. I'm wary however of the effect that long running monopolies have on product quality, and with Google it could be no different.

I do hope that if anyone sees this, and feels the same way - that they pass the question forward. I wonder if the public will see an anonymized version of the referenced study, so that others can come to similar, or differing conclusions using the same information.

I remember the days where Google's experience didn’t feel as difficult or forced. I hope that with Google or another product we can experience those days again.

### Due Credit:
A buddy of mine named Kyle recommended this podcast to me after listening to it. A lot of the ideas I’m sharing were his, or were inspired by his perspective on this episode.

## Links
* [Freakonomics Podcast about Google](https://freakonomics.com/podcast/is-google-getting-worse/)
* [Google Paper on Search Engines](http://infolab.stanford.edu/~backrub/google.html) (see section titled "Advertising and Mixed Motives")
* [Article 1: Referencing Podcast and Study](https://www.dailymail.co.uk/sciencetech/article-11441689/Former-Google-engineer-blames-internets-failures-search-overall-decline.html)
* [Article 2: Referencing Podcast and Study](https://searchengineland.com/is-google-search-getting-worse-389658)
* [Washington Post Article on Google Experience as a Monopoly](https://www.washingtonpost.com/technology/2020/10/19/google-search-results-monopoly/)

Changelog:
 - 2023-01-18: edited for readability
 - 2022-12-04: published

