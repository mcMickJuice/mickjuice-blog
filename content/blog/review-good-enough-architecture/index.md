---
title: 'Review: GOTO 2019 • "Good Enough" Architecture • Stefan Tilkov'
date: '2020-04-12T16:54:00Z'
---

[Link to YouTube Video](https://www.youtube.com/watch?v=PzEox3szeRc)

## Overview

Stefan Tilkov presents a number of different microservice architecture anti-patterns and approaches that he encountered as a consultant. These anti-patterns and foot-guns resulted in poor performance, difficult to manage codebases/systems and disappointed shareholders.

In each example, he cites why a certain decision was made and how it resulted in subpar results. He then provides solutions that are specific to the given projects characteristics. He repeats many times that "context is key" when designing a system and emphasizes that there is no such thing as a perfect architecture for a given system.

My main takeaway from this presentation is that one must take a fluid approach to system design and understand that there are tradeoffs when making architectural decisions. One cannot design a system to be infinitely flexible as ("If you try to satisfy everyone, you'll likely satisfy noone") but with "too much rigidity, a way around the rules will be found". One must understand the costs and benefits to a given approach, communicate said tradeoffs and plan the future around these tradeoffs.

## Key Takeaways

- Often, a specifically coded solution is preferable to highly customizable.
- Netflix has influenced the industry with their "microservice per entity" pattern. While everybody wants to be Netflix, nobody is Netflix.
- Refactoring within team boundaries is much easier than globally.
- There's a fine line between diversity (that adds value) and chaos (that doesn't) and unmanaged evolution will lead to complete chaos (don't be afraid of some architectural influence).
- Smart endpoints, dumb pipes - never put too much intelligence into your architecture.
- Create the simplest thing that will work vs configure for evolvable system (these two are at odds with each other).
- It's fine to change your architecture to evolve. Your system will likely suck so fix it as need be.
- Architecture is not imposing road blocks rather it's about adding guardrails and sign posts. Don't impose your opinions and viewpoints onto others.
