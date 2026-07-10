---
title: 'Satellite handover in non-terrestrial networks'
date: "2026-05-25"
author: "Kenan Jasim"
tags: ["ntn", "3gpp", "networking"]
readTime: true
toc: true
summary: "Notes on why satellite handover is harder than the ground version, and how 3GPP's conditional handover deals with it."
description: "Notes on why satellite handover is harder than the ground version, and how 3GPP's conditional handover deals with it."
---

I have been working on non-terrestrial networks, and satellite handover is one of the parts I have spent a while trying to get my head around. This is roughly what I have understood so far, written up mostly to organise my own thinking, so it is more a set of notes than anything authoritative.

Phones hand over between towers all the time. You drive down the motorway and your phone moves from one tower to the next without you noticing. Doing the same thing with a satellite turns out to be much harder, to the point where there are whole sections of the 3GPP spec about it, and the reason is that the tower is moving, and moving quickly. A satellite in low Earth orbit travels across the sky at roughly 7.5 km per second. By the time the network notices your signal weakening, decides to move you, and sends the command, the satellite it was reacting to has already moved on. Normal handover assumes the network has time to react, and with satellites that assumption does not hold.

## Why the usual approach breaks down

Normally your phone reports signal measurements, the network decides that a neighbouring cell is better, and it commands the handover in real time. On the ground this works fine. In orbit it runs into a few problems: the signal difference across a cell is often too gradual to trigger cleanly, a large number of phones may need to move at once, and the latency means a "hand over now" command can arrive too late to be useful. So the fix is not to make the real-time decision faster. It is to stop making the decision in real time at all.

## Conditional handover: decide early, trigger later

Conditional Handover (CHO) was added in 3GPP Release 16. Instead of commanding the handover when it is needed, the network configures the candidate target cells in advance and attaches a condition to each one. It hands that package to the phone, and the phone watches for the condition and carries out the handover itself when it is met. No real-time command is needed.

It is a bit like leaving instructions in advance rather than directing each step: "when you get to the roundabout, take the third exit." You do not need to be on the phone at the roundabout.

For satellites, Release 17 added two conditions that matter here, one based on distance and one based on time. Which one you use depends on how the beam is pointed at the ground.

## Two kinds of cell

An **Earth Fixed Cell** keeps its coverage on the same area of ground by steering the beam as the satellite moves. The connection is more stable from the user's point of view, but it needs active beam steering to achieve.

An **Earth Moving Cell** does not do this. The beam points straight down and sweeps across the ground as the satellite passes. The hardware is simpler, but the coverage area is always moving, so handovers happen more often.

## D1: the distance trigger

D1 (`condEventD1`) is used with fixed cells. The cell is anchored to the ground and you move across it, so the natural trigger is distance. It fires when you have moved far enough past your serving cell's reference point *and* are close enough to the neighbour's. For fixed cells those reference points are the same for every phone in the cell, so the network can configure it once for everyone.

## T1: the time trigger

T1 (`condEventT1`) is the one I find most interesting, because it leans fully into deciding early. The network knows where its satellites will be, so it can work out the window during which you should switch and simply tell you. The phone watches its own clock and switches when the time falls in that window. There is no real-time round trip that might arrive late.

## SIB19: the data that makes it work

None of this works unless the phone knows where the satellites are and when things will happen. That is what SIB19 provides, a satellite-specific broadcast added in Release 17. It carries the ephemeris (the satellite's position and path), the epoch time, the time at which the serving cell stops covering you (which feeds T1), and the cell's reference location (which feeds D1). Without SIB19 the phone has nothing to check its conditions against.

## Conclusion

The thing that stuck with me while reading about this is that the underlying idea is more general than satellites. The usual response to "the network cannot react fast enough" would be to try to react faster. Conditional handover does the opposite. It does the expensive work early, while there is still time, and then gives the phone a simple rule, a distance or a clock time, to check on its own.

If you want the actual detail, it is in 3GPP TS 38.331, though it is not light reading.
