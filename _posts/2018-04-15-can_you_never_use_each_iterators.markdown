---
layout: post
title:      "Can you NEVER use "each" iterators"
date:       2018-04-15 21:34:44 -0400
permalink:  can_you_never_use_each_iterators
---


# Can you NEVER use "each" iterators

### and will my code shrink in size if I do this?

As an experiment, I'm going to take my CLI Data Gem Project and in a new branch, I will replace every each iterator with a higher level iterator then compare the amount of code from both branches to see if I'm gonna get a shorter and simpler program or maybe I'll hit a roadblock. I'll get back to you with the results.

This was inspired by recent study group where I watched someone use 'each' way too often it made me think what could happen if I removed it completely.

So I'm starting off with 10 each iterators total. Here I go.

Ok, so I'm done✨😇. I removed 5 each iterators, made 42 additions and 48 deletions, fixed 1 bug and most importantly my code is more readable. check out the [merge commit](https://github.com/arye-dov-eidelman/google_experiments/commit/c4059da9502f330b789de4060d9f89ae949b805a).

With regards to the title, although it's technically possible to remove all each iterators, some things like printing a list of results don't have a built-in iterator and are best done manually also I don't know of a better way to do `data.each{ |k, v| self.send( "#{k}=", v ) }` if you do please email me or find me on the slack.
