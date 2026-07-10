---
title: "Post-quantum cryptography, and what's actually shipping"
date: "2026-03-30"
author: "Kenan Jasim"
tags: ["cryptography", "security"]
readTime: true
toc: true
summary: "Some notes on where post-quantum cryptography has actually landed in TLS, and where it hasn't."
description: "Some notes on where post-quantum cryptography has actually landed in TLS, and where it hasn't."
---

A few years ago I worked at a company building post-quantum cryptography, and it is an area I have kept half an eye on since. Going back through my notes recently, the part I find most interesting is not the theory so much as how much of it has actually shipped, and how much has not. This is really me writing that up to make sense of it, rather than any kind of authoritative guide.

## The threat: harvest now, decrypt later

A sufficiently large quantum computer would break the maths behind most of the encryption in use today. The awkward part is that it does not need to exist yet to be a problem. Someone can record your encrypted traffic now and decrypt it once the hardware exists. So anything that needs to stay secret for a long time, like health records or state secrets, is arguably already at risk. That is why this is treated as urgent rather than a problem for 2035.

## What quantum actually breaks

There are really two jobs involved. Key exchange (RSA, elliptic curves) establishes a shared secret between two parties who have not met. Signatures (from the certificate authorities) prove you are talking to who you think you are. Both rely on problems that are hard for classical computers but tractable for a quantum computer running Shor's algorithm, so both need replacing.

## What replaced them

NIST ran an open competition from 2016 to choose the replacements, with entrants trying to break each other's submissions in public. Most of the winners are lattice-based, built on grid problems that stay hard even for quantum machines. If you read older write-ups, including my notes, the names can be confusing, because the finalists were standardised in 2024 under different names:

- Kyber became ML-KEM (key exchange)
- Dilithium became ML-DSA (signatures)
- SPHINCS+ became SLH-DSA (signatures)

They are the same algorithms with new names, so it is mostly a matter of translating as you read.

## What is actually shipping: hybrids

The thing protecting your browser at the moment is not pure post-quantum. It is a hybrid called X25519MLKEM768, which combines the older X25519 elliptic-curve exchange with the new ML-KEM-768. You run both and combine the secrets, so you are safe as long as at least one of them holds. If the new lattice maths turns out to have a flaw nobody has found yet, the classical half still covers you. That caution is why hybrids were preferred over going straight to post-quantum, and they are now fairly widespread. A 2026 survey of tens of thousands of domains found that roughly half already support it.

## The part that has been left alone: signatures

The number that stood out to me from the same survey was the share of those domains using post-quantum *certificates*, which was zero.

So the key exchange has been made quantum-resistant, which protects recorded traffic, but the signatures, the part that proves who you are talking to, are still fully classical. A future quantum attacker could not read an old session, but could forge a certificate and impersonate a server in real time. The reason is fairly practical rather than dramatic: post-quantum signatures are large and awkward. Some do not fit in a single network packet, some need fiddly floating-point maths, and some are simply slow. So the easier half was done first.

## Conclusion

When I try to make sense of what all this means in practice, a few things stand out:

* Hybrid key exchange is the part that is ready today, and turning it on where a stack supports it costs very little.
* Long-lived secrets are the most exposed to harvest-now-decrypt-later, so they seem like the obvious thing to worry about first.
* The signature side is the part still largely unaddressed, and it is worth remembering that quantum-proofing the key exchange is not the same as being safe.

So the migration looks to be about half done. The encryption side has largely been handled, and the authentication side mostly has not.
