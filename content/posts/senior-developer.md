---
title: 'What actually makes a developer "senior"?'
date: "2026-01-26"
author: "Kenan Jasim"
tags: ["discussion", "software engineering"]
readTime: true
toc: true
summary: "Less about clever code, more about protecting the codebase from future pain."
description: "Less about clever code, more about protecting the codebase from future pain."
---

For a long time I assumed "senior" meant writing cleverer code. Denser, more impressive, the kind that makes a junior go "wait, how does that even work." I've come round to the opposite view. The senior code I most appreciate is the boring code I can read half-asleep and still understand instantly.

That's the whole reframe. Senior isn't about showing off. It's about protecting the codebase, and whoever touches it next, from pain you can see coming and they can't yet. Almost everything below is a version of that.

## Write boring code on purpose

The best code you'll write is the code someone else can read without you in the room. Not the tightest, the most obvious. Machines will run almost anything; people have to maintain it, so optimise for the people. When you feel the urge to write the clever one-liner, that's usually your ego asking for a favour that future-you pays for.

## Don't say you're done until you're actually done

"Basically done" is a lie a project tells about itself. It compiles, the happy path works, you're tired. Then someone builds on top of your not-quite-finished thing and the missing 10% gets buried under their work. I'd rather frustrate someone by saying "not yet" than hand over something that looks finished and isn't. Frustration fades in an hour. Hidden half-built work rots for months.

## Never turn refactoring into a ticket

This is the one I'll argue for hardest. The instinct when code is a mess is to be a good citizen: raise a ticket, "Refactor the payments module," give it an estimate. It feels responsible. It's a trap. The moment refactoring is a line item with a number next to it, it becomes the first thing cut when the sprint gets tight, because management quietly deprioritises anything that ships no visible feature.

Fold it into the work that does ship instead. Already touching the payments module for a feature? Pad the estimate and clean up while you're in there. The refactor rides along with something visible, so it actually happens. Not sneaky, just refusing to let good structural work get killed by being made too visible in the wrong way.

## Protect your commitments

You commit to something. Halfway through: "oh, while you're at it, can you also..." And now the thing you promised has grown a second head and you're being micromanaged for "missing" a deadline you never agreed to. Keeping quality high means guarding the edges of what you signed up for. Not being precious about it, just honest: new scope is new scope, let's treat it that way rather than pretend it was always the plan.

## Document the patterns, and talk before you add new ones

Write down the patterns your team uses. Not a novel, just: here's the pattern, the purpose, a link, where we use it. And don't quietly slip a new abstraction into a shared codebase hoping people absorb it by osmosis. Get the team in a room first. A clever new pattern nobody agreed to is a landmine with your commit hash on it.

## Conclusion

There's no rule that five years makes you senior. I've met people three years in who thought like architects, and people a decade in still writing code to impress. The switch flips when you stop asking "how do I write code that works" and start asking "how do I write code, on a team, that's still good news for everyone six months from now."

Boring code. Honest status. Refactoring that survives a deadline. Scope you defend. Patterns you wrote down. None of it flashy. That's the point.
