---
title: "musings on the take home technical interview"
date: 2023-08-23T12:41:00-04:00
tags: ["interviewing", "management"]
categories: ["management"]
---

Time spent
8/23 - 45m
8/24 - 40m
8/29 - 30m read wired article and study
8/30 - 1h15m start talking about take home project
9/2 - 1h
9/2 - 1h setting up old chat project to take pictures
9/4 - 1h trying to setup old project
9/11 - 20m write a little bit more about this

# What motivates the technical take home test?

The take home exercise is an interviewing practice that's existed for a while. It's been around for a while in the form of a case study, and I've seen it personally as an open ended development exercise. If completed well, it provides a great deal of insight into how the candidate works. Like any practice though, it isn't without its drawbacks. Aside from the issues companies face like cheating[1], take home interviews can artificially limit the prospective talent pool, in addition to self-select for dishonest candidates.

This treatise includes the story of two separate software engineers, both who are given a challenge project to complete as a part of their interview. Both challenge projects specify a 2 hour limit, and both engineers eventually receive offers. The only difference is in how they approach the project. Hopefully in reading their stories you can see or strengthen a perspective I have on take home interview assignments.

# The junior's approach

The first engineer is a 1.5 yoe journeyman software developer. His work is a bit rough, but he's motivated - and has the skills to learn. He's reached out to by a recruiter about an opportunity with an AI-focused startup and he's intrigued by the offer. He shares this interest and is asked to complete a challenge project for the company. His task is to build a chat application which demonstrates his skills as a full-stack developer.

On receiving the challenge project he immediately gets to work. He likes the company a lot and wants to stand out. The first day he starts with the backend and builds out the service and the configuration scheme. The configuration system itself is backed by sqlite and he writes a script to setup the configuration for the service. The next day he moves onto the login form and route development, employing a pbkdf2 password security scheme which should demonstrate his basic understanding of secure practices. He continues to build, adding to the back end and front end incrementally. He feels no pressure imposed by the artificial 2 hour limit, he only feels motivation to really deliver a knock out project. He gets stuck many times along the way, sometimes for hours at a time - but in lieu of a real deadline he just works through these issues as they come up.

```
Select an option...
0: Setup mysql
1: Setup password-hashing-iterations
2: Setup socket-key

t: Test Configuration
x: Exit program
```
_The project included a setup script which took all the user entered configuration values and wrote them to a sqlite database_

Another day passes and he's added a full DDL specification for his MySQL application store. Yet another one goes by and he's added functionality for joining and leaving rooms in irc like functionality. He writes tests, stylesheets, http and websocket handlers, another angular directive.

<picture of the completed application>

When he's preparing to zip up and submit the project, it's been more than two weeks. More than 1200 lines of code have been hand-tailored to fit the project's specification and he feels confident that his project will carry him far.

This project is far from runnable. It's got a few bugs in the setup process and doesn't work perfectly. But if one is ablew to run it, it's a fairly impressive piece of work.

The engineer makes it to the onsite, interviews with a number of members of the team and eventually joins the company. At this company he ends up working with some of the smartest people he's ever had the good luck of being around, and sees this experience as the primary growth point in his career.


# The senior's approach

The second engineer is a skilled artisan with 8.5 yoe under his belt. Over the years he's been able to fill in a lot of the gaps in his skills, and he's relatively comfortable building, teaching, and navigating the workspace. He receives the challenge project and sets some time aside to do it. He's struggled with burnout in his career, so he wants to make sure that the project is representative of 2 hours of work - he'd prefer to lose the offer than set unrealistic expectations for himself.

He gets to the project and begins reading it. It's a three page google doc which talks about building an application around a public sports dataset. It provides little direction, and encourages building whatever the engineer feels comfortable with. In two hours, he's pretty sure he'll only have time to build a front-end so he plans for that. 10 minutes has already passed, so before he decides what he wants to build he needs to go over the dataset.

```
last_name, first_name,player_id,year,ppg,blocks,rebounds
James, Lebron, 291, 2021, 30.8, 25, 458
Harden, James, 894, 2021, 24.1, 8, 201
Curry, Stephen, 776, 2021, 31.1, 10, 228
```
_The players dataset was a fairly simplistic into a year by year glimpse of the player's stats_

The dataset itself is fairly simple. He spends some time familiarizing himself with the rows, columns and the variance / cardinality of the data. It's got player records and statistics associated with each player. He figures he'll build a visualization for the normal distribution of each stat listed. In addition to this, he'll add a search bar where the selected player is highlighted along each curve. Another 7-8 minutes has passed so he quickly sets up a plan for work.

1. Setup a boilerplate react application
2. Load the dataset from the filesystem in the app
3. Create a search bar for the identities
4. Graph the curve of all datapoints for some statistic
5. Highlight where the selected player falls along that curve
6. Repeat for additional statistics in the dataset
7. Write a README describing the project and quickstart steps

Project setup goes well, create-react-app really makes this part of the project a breeze. Dataset loading was less simple. Without an API server, loading the dataset in the browser requires passing the file up from the filesystem then parsing the CSV on the front end. The first part he feels alright about, but the latter might require a little more thought. After a couple tries, his fetch call reads the file and logs i to the console. Preliminary searches on csv parsing on the front end lead him to this library named `papi`, but his project has no external dependencies outside of the ones included in create-react-app and he figures he'll see if he can go without it. He toys with the idea of implementing a simple parser on his own, but knows a lot better than that. He finds a [google library](https://code.google.com/archive/p/csv-to-array/) which modifies the String prototype; one which can be pulled from a CDN, just like the old days. It strikes him as the cleanest approach given the constraints of the project so he links to it in his index.html, and gives it a go.

> A little copying is better than a little dependency.
> - The Go Proverbs

First efforts are not good. The data isn't being parsed appropriately. He quickly weighs off the cost of debugging it himself. The problem could be the google library, or the dataset. Since google libraries are usually halfway decent, he assumes the dataset. If the public dataset isn't able to parse effectively there's likely no finished project at the end of 2 hours. 45 minutes have passed already and he's written less than 15 lines of significant code.

He attaches a debugger and steps through it, but he's making little headway. He tries a bit more and reweighs his options. Going against his preference, he installs the `papi` package to see if it works better than the google library. It works, and he's got the tabular data in memory. An hour has passed since he started.

He spends a little time reading the react-select documents, they seem fairly simple to setup. He spends some time filtering the dataset to just the identities and then adds a multiselect to the application. It works fine and he can confirm that selecting an player is firing off an event. Now he's off to the meat of the project.

He's worked with ChartJS before, but just in case he scans the internet to see if there's a more appropriate library for React. There's some stuff, but it seems like [ChartJS has bindings](https://react-chartjs-2.js.org/) for React. He thinks, he spends more time weighing his options having not plotted information in React to his memory. He's built a variety of things in React, but never a plot. He's only done that with Angular.

He thinks on it some more, he's past 1h20m on the project so he decides to go with ChartJS. He installs the react library and compares it to the documentation. The API is different. There's some registration mechanism which he's unfamiliar with. The documentation for the library doesn't seem to explain how it's used, and it seems tangentially related - but the examples from the ChartJS documentation seem to not require it. Confused, he decides to reverse engineer an example and go from there. He spends another few minutes deciding the best chart and then selects the Area Chart. He copies the code from the example and fiddles with it a little bit so it renders.

```
import React from 'react';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler,
  Legend,
} from 'chart.js';
import { Line } from 'react-chartjs-2';
import faker from 'faker';

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler,
  Legend
);
```
_The sample code from the react-chartjs-2 library for Area Charts._

He thinks about rendering the dataset, and realizes he'll have to come up with some bucketing mechanism. He's also still got to highlight where the selected player falls along the distribution so he gets started with that. He searches the internet for this and finds some promising posts that address this situation. He looks back at what he has and again tries to understand how the React registration maps to the underlying ChartJS library. If he misses a registration, would this specific change not work? What all are the registrations.

At around 2 hours, he steps away from his computer. He tells the hiring manager he's spent two hours on it but he'd like more time to work on it. The manager says not to worry about it, preferring a not-perfect 2 hour project to one which is worked on longer.

He thinks of it as less as a "not-perfect" solution and more like a "just-barely-better-than-handing-in-nothing" mess. He's dedicated to honesty with himself and others in this respect however, so he hands the project in.

At this point the sum of the work he's completed is:

 - Fetching a CSV from the filesystem
 - Creating a typeahead input from the data
 - Rendering a single graph, whose data and lables have been hardcoded and copied from the library's repository

![The finished project](/img/take-home-interview-senior-final.png)

The total of this effort is 81 lines of code outside of what create-react-app added, which is a paltry show of skill.

# Who do take-home assignments select for?

If it isn't obvious at this point, both engineers are me. The junior engineer interviewed for ASAPP, the senior for The Mets.

Now if I were to receive either of these projects, I likely wouldn't know much about the engineer than what they created. The first project was extensive, clean, creative and beyond what I would expect candidates to hand in. The second project is incomplete, sloppy, and confusing - the result of a person cut short in the middle of building something many times smaller. If I were to judge the two, the first project is far better than the latter.

On a level playing field, I'm more experienced and skilled now than I was at 1.5 years. A take-home project isn't a level playing field though. I'm not going to work 30 hours for a take home project, even though I see myself as many times more productive than I was then.

Herein lies the crux of the issue. While I feel more productive during 40 weekly hours than my younger 60, perhaps 70 hours a week counterpart, our projects tell quite a different story.

This selects for people who are willing to go "above and beyond" what they're ordinarily capable of.

1. https://www.holloway.com/g/technical-recruiting-hiring/sections/technical-interview-formats

https://github.com/angles-n-daemons/chat
