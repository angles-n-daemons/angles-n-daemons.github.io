---
title: "musings on the state of technical interviews"
date: 2023-08-23T12:41:00-04:00
tags: ["interviewing", "management"]
categories: ["management"]
---

Time spent
8/23 - 45m
8/24 - 40m
8/29 - 30m read wired article and study
8/30 - 1h15m start talking about take home project

split this into two blogs, against take homes and other approaches

Research:
Read 5 articles on types of technical interviews

Case Interview (case study)

I've never seen this, but it sounds like you propose an engineeer with a business scenario and ask the interviewee to study the problem, perform an analysis and discuss the problem in person

Seems good for those who are able to break down business problems as well as communicate

Coding interview

Assessing the candidate's coding skills.

white board, pair programming, etc

tech quizzes which tell you very little about coding skills

https://www.holloway.com/g/technical-recruiting-hiring/sections/technical-interview-formats

Effort and skill enumerated for qualification / phone screen / remote conversation / remote coding

onsite whiteboard

onsite task / project

take home coding project

portfolio or prior work

Has a great set of descriptions on interview formats

phone screen / video call
take home / assignment
in-person meeting

Best interview format is contraversial

Read triplebyte
Read google on fermi questions

Engineers find whiteboarding intimidating, stressful and not predictive of job performance

Dropbox couldn't contain cheating on take homes

it's not fair because people are cheating - talk about my own experience

https://www.shecancode.io/blog/its-time-to-end-whiteboard-interviews-for-software-engineers

Boot camps are an entry point to software engineering to underrepresented groups

Lots of engineers talk negatively about whiteboarding interviews
- Bias towards CS degree graduates

Whiteboard interviews are intimidating, especially for those with imposter syndrome

Remote can compound this factor

https://artsy.github.io/blog/2019/01/23/artsy-engineering-hiring/

artsy on interviewing by open communication

Pair programming (still kind of whiteboarding)

Talking through past work / code

Leaning into references?

2023 - 08 - 24

What is this discussion

https://www.wired.com/2015/04/hire-like-google/

First five minutes of an interview matter the most

thin slices

early judgements are useless

confirmation bias, move from assessing to finding evidence to support our initial impression

how many golf balls fit inside of a 747

makes the interviewwer feel clever

SVP people operations at Google

Unstructured job interviews are bad at prediccting how someone would perform once hired

reference checks 7%
work sample 29%

general cognitive ability tests discriminate against non-white, non-male test takers, despite being a strong indicator for work performance

structured interviews are predictive even for unstructured jobs (interesting)

People who score high on conscientiousness "work to completion"

The goal of the interview process is to predict how a candidate will do once they join the team

Do what the science says, behavioral and situational structured interviews with cognitive ability, consientiousness and leadership.

Measure on a consistent rubrik

Don't let only the boss interview

Get a disinterested assessment




https://rework.withgoogle.com/guides/hiring-use-structured-interviewing/steps/avoid-brainteasers/

Google interviewing practices

https://home.ubalt.edu/tmitch/645/session%204/Schmidt%20&%20Oh%20validity%20and%20util%20100%20yrs%20of%20research%20Wk%20PPR%202016.pdf

1998, Frank Schmidt and John Hunter 

Individual performance assements are poor

general mental ability coupled with something else (integrity, structured interview, etc)
general cogniative ability coupled with a structured interview

This essay is changing drastically

General Cognitive ability seems to be the best predictor by far
Take home tests seem to do little

Keep in mind this depends how on the job performance is measured

The case against take-home tests

variability in hires matters a lot

Might be worth discussing take home assignments

Why not take home assignments, references, past work

Why programming exercises

# Another dash of salt surrounding technical interviewing

As a software engineer with some experience under my belt, I've seen a lot of interviews from both sides. I've seen the good and the bad from both sides of the line, and after a somewhat uncomfortable recent interview I wanted to flush some of my stronger thoughts out to disk. At first I wanted to bring to light some criticisms on take-home projects, but after exploring my thoughts around this practice I found that there thoughts outside of it which I also wanted to share. With that in mind, I've set out to talk about a variety of practices to screen someone's technical abilities. This doesn't take into account ways to make interivews approachable, or screening for non-technical traits that provide generous value.

# The take-home project

I'll start this treatise with the thought that motivated this post. It's a story of two software engineers, both of whom received a take-home project as part of their interview. Both interview projects specify a 2 hour time limit, and both engineers end up receiving offers. The capabilities of each engineer, and the path they take to complete the challenge project is what is unique to each of their stories. This story seeks to highlight a possible anti-pattern of employing take-home projects.

The first engineer is a 1.5 yoe journeyman software developer. His work is a bit rough, but he's motivated - and has the skills to learn. On receiving the challenge project he immediately gets to work. He loves the company he's working for but he really wants to stand out. The rough time estimate for the project is about 2 hours, and he discards that instruction and says he'll spend as long as he needs to to make it a good application.


---- Second engineer


The second engineer is a skilled artisan with 8.5 years of experience under his belt. Over the years he's been able to fill in a lot of the gaps in his skills, and he's relatively comfortable building, teaching, and navigating the workspace. He receives the challenge project and sets some time aside to do it. He's struggled with burnout in his career, so he wants to make sure that the project is representative of 2 hours of work - he'd prefer to lose the offer than set unrealistic expectations for himself.

He gets to the project and begins reading it. It's a three page google doc which talks about building an application around a public dataset. It provides little direction, and encourages building whatever the engineer feels comfortable with. In two hours, he's pretty sure he'll only have time to build a front-end so he plans for that. 10 minutes has already passed, so before he decides what he wants to build he needs to read into the dataset.

The dataset itself is fairly simple. He spends some time familiarizing himself with the rows, columns and the variance / cardinality of the data. It's got identity records and statistics associated with each identity. He figures he'll build a search widget, which visualizes a normal distribution for each stat listed as well as highlighting where the selected identify falls along the curve. Another 7-8 minutes has passed

1. Setup a boilerplate react application
2. Load the dataset from the filesystem in the app
3. Create a search bar for the identities
4. Graph the curve of all datapoints for some statistic
5. Highlight where the selected identity falls along that curve
6. Repeat for additional statistics in the dataset
7. Write a README describing the project and quickstart steps

Project setup went well, create-react-app really makes this part of the project a breeze. Dataset loading was less simple. Loading the dataset in the browser would require passing the file up from the filesystem, then parsing the CSV on the front end. The first part you felt alright about, but the latter might require a little more thought. After a couple tries, your fetch call reads the file and is logging it to the console. Preliminary searches on csv parsing on the front end lead you to this library named `papi`, but your project has no external dependencies outside of the ones included in create-react-app and he figures he'll see if he can go without it. He toys with the idea of implementing a simple parser on his own, but knows a lot better than that. He finds a [google library](https://code.google.com/archive/p/csv-to-array/) which modifies the String prototype; one which can be pulled from a CDN, just like the old days. It strikes him as the cleanest approach given the constraints of the project so he links to it in his index.html, and gives it a go.

First efforts are not good. The data isn't being parsed appropriately. He quickly weighs off the cost of debugging it himself. The problem could be the google library, or the dataset. Since google libraries are usually halfway decent, he assumes the dataset. If the public dataset isn't able to parse effectively there's likely no finished project at the end of 2 hours. 45 minutes have passed already and he's written less than 15 lines of significant code.

He attaches a debugger and steps through it, but he's making little headway. He tries a bit more and reweighs his options. Going against his preference, he installs the `papi` package to see if it works better than the google library. It works, and he's got the tabular data in memory. An hour has passed since he started.

He spends a little time reading the react-select documents, they seem fairly simple to setup. He spends some time filtering the dataset to just the identities and then adds a multiselect to the application. It works fine and he can confirm that selecting an identity is firing off an event. Now he's off to the meat of the project.

He's worked with ChartJS before, but just in case he scans the internet to see if there's a more appropriate library for React. There's some stuff, but it seems like [ChartJS has bindings](https://react-chartjs-2.js.org/) for React. He thinks, he spends more time weighing his options having not plotted information in React to his memory. He's built a variety of things in React, but never a plot. He's only done that with Angular.

He thinks on it some more, he's past 1h20m on the project so he decides to go with ChartJS. He installs the react library and compares it to the documentation. The API is different. There's some registration mechanism which he's unfamiliar with. The documentation for the library doesn't seem to explain how it's used, and it seems tangentially related - but not uniform with the ChartJS API. Confused, he decides to reverse engineer an example and go from there. He spends another few minutes deciding the best chart and then selects the Area Chart. He copies the code from the example and fiddles with it a little bit so it renders.

He thinks about rendering the dataset, and realizes he'll have to come up with some bucketing mechanism. He's also still got to highlight where the selected identity falls along the distribution so he gets started with that. He searches the internet for this and finds some promising posts that address this situation. He looks back at what he has and again tries to understand how the React registration maps to the underlying ChartJS library. If he misses a registration, would this specific change not work? What all are the registrations.

A few minutes from 2 hours, he steps away from his computer. He tells the hiring manager he's spent two hours on it but he'd like more time to work on it. The manager says not to worry about it, preferring a not-perfect 2 hour project to one which is worked on longer. At this point the sum of the work he's done has been:

 - Fetching a CSV from the filesystem
 - Creating a typeahead input from the data
 - Rendering a single graph, whose data and lables have been hardcoded and copied from the library's repository

He thinks of it as less like a "not-perfect" solution and more like a "just-barely-better-than-handing-in-nothing". He's dedicated to honesty with himself and others in this respect however, so he hands the project in.

He tries to finagle

Rob Pike (a little copying is better than a little dependency)

# programming exercises

Data Structures and Algorithms provide a shared foundation that can be learned in a relatively short period of time. Absent this shared foundation, the overlap of skills between the interviewer and interviewee could be nil, and no signal could be gained.

# The go to screening process
Process that most organizations include is some number of programming exercises, and a system design portion


The cost for a mismatch? find citation
 - The cursed programming exercise
 Perhaps the most devisive in the industry, link to examples
 I've been cursed at in the middle of an interview
   - Straightforward
   - Debugging
   - Algorithmic
     - For this actually, the industry at this point has a pretty standard set of information that benefits candidates in this step. Complexity theory, data structures like trees, graphs, algorithms like searching, sorting, and often the flexitility to use language built-ins 
     - Asking about fringe data structures is as unfair as asking about fringe language features, I wouldn't expect a candidate to know about bloom filters anymore than I would expect them to know the python VM instruction set.
 talk about merits and drawbacks of square style interview
 Can develop lots of problems, so that candidates cannot memorize all of them in preparation (prevents cheating)

 - System Design
   - Can introduce a bit of bias into the process. Where I've seen this done, there wasn't necessarily a standardized rubric.
   - There's also a lot of room for alternative answers, which requires the interviewers to understand the possibilities much more than the candidates which isn't always the case. A BE engineer interviewing a FE engineer for a position may not gather that much signal on the depth and breadth of the FE engineer's skills 
   - Programming exercises do generally have a small set of answers, and their optimiality is relatively clear. For system design, the answers can include lots of configurations of schema design, infrastructure components, data storage systems in a way that makes it difficult to gauge the optimality of the answer. Often how good the candidate is at convincing you the design is optimal is the measure rather than how it would handle a real world test

# The take-home project, signal on stilts

 - Take Home Project
start with the good
Tons of signal in terms of the person's if it's done
Lots to dig into in how they design, write for readability, etc

Lots of overhead, both for the company and the person taking the project home

I don't think many decent projects come back in two hours.

quote the artsy thing
quote the dropbox thing (stopped doing it because of lots of cheating)
tell story about my experience
asapp 2 weeks, python, javascript, sql schema files, bash scripts to get it running
mets 2 hours - debugging project setup, debugging csv library, reading the documentation on chartjs (which I've used but not in conjunction with react)
ended with the create-react-app boiler plate, a text input, a couple data transformation functions, and a graph which showed nothing

The project which I worked on for asapp was much better than the one I worked on for the mets, but the engineer who would be working today is a much better engineer than the one who was working 6 years ago

What does it bias towards? People without worklife balance. Good for startups. To a degree. I feel like me working at 40 hours a week, is going to develop much better software than the me who would work 60 hours a week. herein lies the crux, 

# The technical quiz, an aside on a slowly fading practice on interviewing

 - For senior engineers, can really highlight their depth and breadth of knowledge
 - For junior engineers, this can be really hit or miss. 

Rote, doesn't actually gage skill in applying
People can game for this by searching "top 20 sql interview questions" etc. Requires little skill to deliver the right answer

# The past-projects interview
   - Talk about squarespace Essay
   - Looking over code
 - Drawback of not being familiar with the solution, so takes more overhead to build context around project
 - Drawback of not knowing the problem statement, not knowing whether the solution fits

# The "day in the life" Interview, a platonic ideal I've never experienced
talk about circleci
technically illegal if not done correctly
if done correctly, requires a lot of overhead in filling out tax documents etc
assumes your systems are easy enough to run / test / deploy. biases for toolchains

# Probing alternatives
 - More training for interviewers
   - Open source resources for reviewing pair programming, system design, take home project review
 - reference calls between an engineer and reference
   - mention how artsy leans into this exercise
 - Technology discussion, usage, quirks, learnings, etc
