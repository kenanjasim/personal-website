---
title: 'What actually makes a developer "senior"?'
date: "2026-01-26"
author: "Kenan Jasim"
tags: ["discussion", "software engineering"]
readTime: true
toc: true
summary: "Why I don't think being a senior developer is mostly about writing cleverer code."
description: "Why I don't think being a senior developer is mostly about writing cleverer code."
---

For a long time I assumed that being a "senior" developer mostly meant writing better code. By better I think I meant cleverer, or denser, the sort of thing that makes someone stop and work out how it does what it does. The longer I have worked on teams, the less I believe this.

The senior code I appreciate most now is usually the boring code, the code I can read when I am tired and still understand. That is roughly the shift I want to talk about: seniority is less about the code itself, and more about keeping the codebase, and the people working in it, out of trouble later on. Most of the things below come back to that.

## Write code that is easy to read

Code that other people can read without you sitting next to them is usually worth more than code that is clever. Machines will run almost anything, but people have to maintain it, so it makes sense to write for the people. When I get the urge to write a clever one-liner, it is usually worth stopping and asking whether the next person will actually thank me for it.

## Don't say something is done until it is

"Basically done" is easy to say when the code compiles, the happy path works, and you are tired of looking at it. The problem is that someone will build on top of it, and the last ten percent you skipped ends up buried under their work. I would rather tell someone something is not finished yet than hand over something that looks finished but is not.

## Be careful about turning refactoring into a ticket

This is the one I feel most strongly about. When some code is a mess, the obvious "responsible" thing to do is raise a ticket to refactor it. In practice I think this often backfires. Once a refactor is a separate ticket with an estimate on it, it becomes one of the first things dropped when a sprint gets tight, because it does not ship anything a user can see.

I have had more success folding that work into a feature that does ship. If I am already in the payments code for a feature, I will clean it up while I am there. The refactor travels with something visible, so it tends to actually get done.

## Be clear about scope

You agree to do something, and then partway through it grows. "While you are in there, could you also...". Before long the thing you committed to is bigger than what you agreed to, and you can end up being held to a deadline you never really signed up for. Part of keeping the quality up, I think, is being honest about this when it happens, and saying plainly that the extra work is extra, rather than letting it quietly fold into the original estimate.

## Write down your patterns, and talk before adding new ones

It helps to write down the patterns your team uses. Not a big document, just enough: what the pattern is, why it exists, and where it is used. I also try not to quietly drop a new abstraction into a shared codebase and hope people pick it up on their own. It is usually better to talk it through with the team first, otherwise you have added something nobody really agreed to.

## Conclusion

When I think about what makes someone senior, it is usually these things:

* They write code that is easy to read, even when they could write something cleverer.
* They are honest about when something is and is not finished.
* They keep structural work like refactoring attached to work that ships.
* They are clear about what they have agreed to do.
* They write their patterns down, and talk about new ones before adding them.

None of this is really about years. I have worked with people a few years in who already think this way, and people much further along who do not. For me the change is when you stop asking "how do I write code that works" and start asking "how do I write code the team will still be glad of in six months".
