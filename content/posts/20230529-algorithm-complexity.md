---
title: "A bit on complexity theory you should know for interviewing"
date: 2023-05-29T16:32:00-04:00
tags: ["computer science", "math", "algorithms", "optimization"]
categories: ["computer science"]
---

# Prerequisite knowledge
 - Basic Programming
 - Basic Math

# Overview

Understanding how your code scales is an important part of the programming interviewing process. While the merits of having it in interviews is debated, there's no question that having some understanding of complexity theory is generally benficial when conducting a job search. With that in mind, I wanted to describe what I think would be important about complexity theory for any technical candidate.

This document will go into:
 * The terminology used in complexity theory
 * A descriptive rundown of various complexity classes
 * Some additional information on how to use complexity classes
 * How to approach discussing complexity in an interview
 * A quick set of practice problems to familiarize yourself with complexity theory

# Terminology

### n, the size of the input

The first term we need to go over when talking about complexity theory is n, the size of the input. The reason for this is that complexity theory deals with scaling, and in scaling we need a placeholder for our input size. Consider the following program:

```typescript
function sum(nums: Number[]): Number {
    var result = 0;
    for (num in nums) {
        result += num;
    }
    return result;
}
```

n in this case represents the length of the array nums. When discussing complexity theory, we are curious about how the program scales when the size of the input grows.

If we were to call `sum([5])`, n would be 1. If we were to call `sum([1,2,3])` n would be 3. In generally complexity theory is useful when your n approaches very large numbers.

### Big-O Notation

Big-O Notation is a word that's used pretty frequently, but if you haven't been exposed to it may seem foreign and obscure. [Definition pulled from the internet].

What does Big O look like: O(class) where class is the complexity class, this can look like O(1), O(log(n)), O(n^2) and many others. Why 1, log(n) or n^2 isn't important for right now, all you need to know is that they describe how a program scales to the size of the input.

- Describe what n is
  - Use code to show this
- Talk about O notation
  - Commonly referred to as Big-O
  - Looks like O(<complexity class>)
  - It's worst case
- Complexity class (what the heck is this called)

# Complexity Classes
- Constant

A simple constant function.

```
function selectFirstElement(nums: Number[]) {
    if (nums.length == 0) {
        throw(Error("cannot select first from empty array"));
    }

    return nums[0];
}
```

This is a simple (and perhaps contrived) example of a function with constant time complexity, also referred to as O(1). The thing that should stick out when looking at this program is that it always performs the same number of operations, regardless of the input size. If nums had a length of 1, it would perform the same number of operations as if it were length 1,000,000.

- Linear



- Logrithmic

- nlog(n), a unique case

- Quadratic
- Cubic and Beyond

# Additional information
- Some more info
  - you don't need all termsThe least you should know about algorithmic complexity"
    - Example with linear and quadratic
  - If you multiply by a constant, you can remove the constant
- Multiple sizes for the input

# Graph

- Allows you to check different complexities
- Gives a description in the differences
- Shows divergence from some small number to some large number

# Quiz on complexities

- Example one, linear
- Finding duplicates
  - Quadratic
  - Linear
