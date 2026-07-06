---
title: "What's replacing RSA in a post-quantum world?"
date: "2026-03-30"
author: "Kenan Jasim"
tags: ["cryptography", "security"]
readTime: true
toc: true
summary: "A short tour of the post-quantum crypto already shipping in TLS, and the half that still isn't."
description: "A short tour of the post-quantum crypto already shipping in TLS, and the half that still isn't."
---

I took a big pile of notes on post-quantum cryptography a few years ago, mostly to convince myself the threat was real. Coming back to them now, the interesting part isn't the theory. It's how much of it actually shipped, and which bit everyone quietly skipped.

## The threat: harvest now, decrypt later

A big enough quantum computer breaks the maths behind almost all of today's encryption. The catch is that it doesn't need to exist yet to hurt you. Someone can record your encrypted traffic today and decrypt it the day the hardware arrives. So anything that needs to stay secret for a decade (health records, state secrets) is already at risk right now. That's why this is urgent and not a 2035 problem.

## What quantum actually breaks

Two jobs, really. Key exchange (RSA, elliptic curves) establishes a shared secret between strangers. Signatures (the certificate authorities) prove you're talking to who you think. Both rest on problems that are hard for us and easy for a quantum computer running Shor's algorithm. So both need replacing.

## What won

NIST ran an open competition from 2016 to find the replacements, with everyone trying to break everyone else's work in public. Most winners are lattice-based, built on grid problems that stay hard even for quantum machines. If you read older write-ups (like my notes) the names will confuse you, because the finals shipped in 2024 with grown-up names:

- Kyber became ML-KEM (key exchange)
- Dilithium became ML-DSA (signatures)
- SPHINCS+ became SLH-DSA (signatures)

Same algorithms, new badges. Just swap the names as you read.

## What's actually shipping: hybrids

The thing protecting your browser right now isn't pure post-quantum. It's a hybrid called X25519MLKEM768: the old, trusted X25519 elliptic-curve exchange bolted to the new ML-KEM-768. You run both and mix the secrets, so you're safe as long as at least one holds. If the new lattice maths has a flaw nobody's spotted, the classical half still covers you. That caution is why hybrid beat going all-in on post-quantum, and it's already everywhere. A 2026 survey of tens of thousands of domains found roughly half already support it.

## The half everyone skipped: signatures

Here's the number that stopped me. Of those same domains, the share using post-quantum *certificates* was zero.

We quantum-proofed the key exchange, so recorded traffic stays safe, and left the signatures (the part that proves who you're talking to) fully classical. A future quantum attacker couldn't read an old session, but could forge a certificate and impersonate the server live. We fixed the lock and left the ID badge a photocopy. The reason is boring and practical: post-quantum signatures are big and awkward. Some don't fit in a single network packet, some need fiddly floating-point maths, some are just slow. So everyone did the easy half first.

## Conclusion

If you run anything that matters, the moves are unglamorous:

1. Turn on hybrid key exchange where your stack supports it. Half the internet already has, and it costs almost nothing.
2. Find your long-lived secrets, because harvest-now-decrypt-later is aimed squarely at them.
3. Watch the signature side, because that's the unfinished half. "We did the key exchange" is not the same as "we're safe."

The migration is half done, and the publicly measured data proves it. The encryption got quantum-proofed. The authentication mostly didn't.
