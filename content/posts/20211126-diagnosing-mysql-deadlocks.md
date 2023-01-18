---
title: "Diagnosing MySQL deadlocks"
date: 2021-11-26T19:02:13-04:00
tags: ["mysql", "innodb"]
categories: ["software", "databases"]
---

# An unpleasant exception

```
Deadlock found when trying to get lock; try restarting transaction
```

If you're reading this document, you're either here out of curiosity or necessity. I'll be brief for those in the latter camp. When you encounter the above error in production, you only need the stack trace to figure out which query got blocked. What I have found is difficult to ascertain is which query is doing the actual blocking.

A precursory search for mysql deadlock errors will show that entering the command `show engine innodb status` "gives you information about the last known deadlock". What you probably haven't been told is that unless you've scanned the output of this command before you are in for a bit of heavy reading.

To find the offending transaction blocking the above, search the output for the section containing the phrase `HOLDS THE LOCK(S):`, and the offending query should be right above it. Example output below:

```
MySQL thread id 2, OS thread handle 140555289204480, query id 22 172.17.0.1 root updating           
DELETE FROM t WHERE i = 1                                                                           
*** (2) HOLDS THE LOCK(S): 
```
