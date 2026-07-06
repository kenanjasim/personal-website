---
title: 'How do you hand over a call when the tower is moving at 7.5 km/s?'
date: "2026-05-25"
author: "Kenan Jasim"
tags: ["ntn", "3gpp", "networking"]
readTime: true
toc: true
summary: "Why satellite handover breaks normal cellular assumptions, and how 3GPP's conditional handover fixes it."
description: "Why satellite handover breaks normal cellular assumptions, and how 3GPP's conditional handover fixes it."
---

When I started working on non-terrestrial networks, I had a question that felt too dumb to ask. Phones hand over between towers constantly. You drive down the motorway and your phone switches towers without you noticing. So why is doing the same with a satellite a hard enough problem that whole chunks of the 3GPP spec exist for it?

Then it clicked: the tower is moving. Fast. A low-Earth-orbit satellite crosses the sky at about 7.5 km every second. By the time the network notices your signal fading, decides to move you, and sends the command down, the satellite it was reacting to has already gone. Normal handover assumes the network has time to think. Satellites take that assumption away.

## Why the old way breaks

Normally your phone reports signal measurements, the network decides a neighbour is better, and it commands the handover in real time. On the ground that's fine. In orbit it falls apart: the signal difference across a cell is often too soft to trigger cleanly, huge numbers of phones need to move at once, and the latency means a "hand over now" command can arrive too late to be useful. So the fix isn't a faster real-time decision. It's to stop making the decision in real time at all.

## Conditional handover: decide early, trigger later

Conditional Handover (CHO), added in 3GPP Release 16, pre-configures the candidate target cells in advance and attaches a condition to each. The network hands that package to your phone and steps back. Your phone watches for the condition and executes the handover itself when it's met. No real-time command needed.

Think of it as leaving instructions instead of micromanaging. "When you reach the roundabout, take the third exit." You don't need to call them at the roundabout.

For satellites, Release 17 added two conditions that matter: a distance one and a time one. Which you use depends on how the beam treats the ground.

## Two kinds of cell

An **Earth Fixed Cell** keeps its coverage pinned to the same patch of ground by steering the beam, like keeping a torch on one paving stone as you walk past. The connection feels stable, but the satellite needs active beam steering to pull it off.

An **Earth Moving Cell** doesn't bother. The beam points straight down and sweeps across the Earth like a spotlight. Simpler hardware, but the footprint is constantly moving over people, so you get frequent handovers.

## D1: the distance trigger

D1 (`condEventD1`) is for fixed cells. The cell is anchored to the ground and you're moving across it, so the trigger is distance. It fires when you've moved far enough past your serving cell's reference point *and* close enough to the neighbour's. For fixed cells those reference points are the same for every phone in the cell, so the network configures it once for everyone.

## T1: the time trigger

T1 (`condEventT1`) is my favourite, because it leans all the way into "decide early." The network knows where its satellites will be, so it computes the exact window when you should switch, and just tells you. Your phone watches its own clock and switches when the time lands in that window. It's basically a calendar invite: "connect to the next satellite at 14:32:05." No slow round trip that might miss.

## SIB19: the data that makes it work

None of this works unless your phone knows where the satellites are and what time things happen. That's SIB19, a satellite-specific broadcast added in Release 17. It carries the ephemeris (the satellite's position and path), the epoch time, when the serving cell stops covering you (feeds T1), and the cell's reference location (feeds D1). Pull SIB19 out and the whole thing collapses, because the phone has nothing to check its conditions against.

## Conclusion

The lesson is bigger than satellites. The old answer to "the network can't react fast enough" would have been "react faster." Conditional handover does the opposite: it does the expensive thinking early, when there's time, then hands the phone a dumb rule (a distance, a clock time) to check on its own.

When the world moves too fast to respond in the moment, don't respond faster. Decide in advance. And if you want the real detail, it's in 3GPP TS 38.331, which is not light reading.
