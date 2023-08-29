---
title: "musings on the state of technical interviews"
date: 2023-08-23T12:41:00-04:00
tags: ["interviewing", "management"]
categories: ["management"]
---

Time spent
8/23 - 45m
8/24 - 40m
8/25 - 30m read wired article and study

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

# Another drop in the sea of opinions surrounding technical interviews

Intro about technical interviewing
 - Go into my experience
 - Talk about how I'll use my experience to show some perspective that may be new on certain types of interviews
 - Focus on technical screening, team fit, cultural, motivation etc aren't the topic of focus.
 - Does the person have the technical experience to perform their job function, write code, turn business problems into technical solutions

What makes interviews good? Structured, prepared questions, having a rubric for grading candidates
 - Also omtting the intro where the person talks about their experience
 - Also going to omit how to be a good interviewer, to look for reasons to hire, to make the candidate feel comfortable, to try to understand solutions that diverge from your biases, to encourage a person's best work

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
