---
layout: post
title: Working with Legacy code - DRY (Don't Repeat Yourself)
date: 2023-12-28
category: code
author: Mark Townsend
tags: [legacy code]
summary: 
---

Let's start this post with a couple of definitions.

**Legacy Code:** Any code that you either didn't write or code you wrote and can't remember ***why*** you wrote it in the first place.

**DRY (Don't Repeat Yourself):** This is one of the traditional software development principals that should be followed no matter what language or platform is being developed.  Don't write the same chunk of code in multiple places either in the same file or across the project.  This is for the following reasons:

* readability
* composability
* maintainability
* slow compilation time

If you've ever worked at a company that has been around for 5 or more years, you will undoubtedly will have code that is thousands of lines long and with thousands of source files. There are several ways to approach this project if you are new.  One way to make an impact is to look for places in the code where there is repeated code. Sometimes, developers just like to copy/paste code in different places because it "feels" easier. However, for the reasons mentioned above, it is not "easier" in any definition of the word.

One of the best tools that I've found is part of the [PMD - source code analyzer project](http://adangel.github.io/pmd/index.html). This tool has lots of ways to analyze source code in lots of different programming languages including Objective-C and Swift.  The support for it's various rule sets for Objective-C and Swift are limited, but you can still find some good ones to help see the current state of your code.

PMD has a sub-tool that can detect portions of code that are exactly the same.  This is what copy/paste detection is.  Why is it an important thing to understand in a large codebase? There are two that immediately come to mind.  Every line of code needs to be compiled and that compilation takes time. CPD is one way to optimize the code by removing duplicate code so it's not being compiled multiple times.  Another reason is the general design principle called [DRY - Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don't_repeat_yourself). Documentation for CPD can be found [here](http://adangel.github.io/pmd/pmd_userdocs_cpd.html).

Once you find areas in the code that are copy/pasted, this is a prime time to do some refactoring.

If the section of code is in the same file, the easiest thing to do is to create a new function with the repeated code and then call that function in all the places that it's used.

If the section of code is across several different files, then analyze how that code is being used. If it is being used as business logic for something, then perhaps, create a function and put it in new class that describes the business domain that the business logic belongs to.  Include this new class as a dependency to all the places the code was used and call the function.  The advantage to this is that when this business logic changes, then you only have to change it in one place.

These types of refactors shouldn't even need to be retested (although you of course should) because you are not changing the actual code, you are changing how or where it's being called from. And now you have easier to understand, maintainable, and probably even compiles a little bit faster because you've also deleted code in the process.
