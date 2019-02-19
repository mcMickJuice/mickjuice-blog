---
title: WILLW - February 11th, 2019
date: '2019-02-18T02:16:27.765Z'
---

WILLW (What I Learned Last Week) will (hopefully) be a weekly post that I write to chronicle what I worked on, what I learned, blog posts I read, videos I watched and other various sundry that I wanted to put on paper.

## Daylong Code Hacks Are Great

Last week, I participated in a work sponsored "CodeHack". There were really no restrictions on what you could work on, granted that it had to be technical in nature and it was gently suggested that it should be something that would benefit your team or the company.

I utilized this time to focus on a POC feature that we are looking to introduce into our app in the coming month. I really enjoyed the experience as I was able to focus 100% on coding and experimenting while avoiding distractions.

My takeway from this hack was that even if you work at a place that doesn't have such events, you can still replicate the experience on your team to derive the same benefits. Maybe once a month or once a quarter, your team could dedicate a full day to working on something, anything to improve your team's skills, introduce utilities/tools that can help productivity or do some exploratory work on a coming feature.

## Writing a Technical Blog Post Takes a Lot of Time

Last Sunday, [I wrote a post](/hooks-exclamation-point) about React Hooks. It's a fairly straightforward post which shows how you can cleanup your state management code with hooks. Despite it's relatively simple content, from start to finish the post probably sucked up 4 hours of my time.

Some of this is due to me being a novice blogger (writer?) but I think it's mostly due to the fact that in technical writing, it is HARD to convey ideas in a simple manner. The act of "dumbing down" content takes a lot of effort as you need to constantly review what you write to make sure that you're not glancing over concepts or ideas that some readers do not know.

## The React Hooks FAQ is Awesome

I've been using the [new Hooks feature in React](https://reactjs.org/docs/hooks-intro.html) for about 2 weeks and I am really digging it.

I did have a few questions/concerns about the code I was writing:

- Am I using hooks in a correct fashion?
- Are there performance concerns with defining functions within components?
- How do I achieve X?

Googling one of my questions brought up the [React Hooks FAQ](https://reactjs.org/docs/hooks-faq.html) which answered every one of my questions AND includes some really interesting tidbits about this feature.

## Immer Looks Like a Cool Library

While reading one of the blog posts listed below, they linked to a project called [Immer](https://github.com/mweststrate/immer), a JavaScript immutability library.

A few things stuck out to me that are appealing:

- You do not need to learn a new API to use it, IMO the main drawback of using [immutable-js](https://github.com/immutable-js/immutable-js)
- It's method of updating is [straightforward](https://hackernoon.com/introducing-immer-immutability-the-easy-way-9d73d8f71cb3#c0cd)
- The way it achieves [this is awesome](https://hackernoon.com/introducing-immer-immutability-the-easy-way-9d73d8f71cb3#3bff) cuz [proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) are awesome!

## VSCode - View: Toggle Panel Position

I use Visual Studio Code and their integrated terminal (and panel). At some point, they removed the button to toggle the positioning (vertical/horizontal) of the panel. As such, I had some projects where the panel was positioned at the bottom, horizontally and others where the panel was positioned on the side, vertically.

I put up with this for probably three months until last week, I googled "How to change the positioning of vs code terminal" and wallah, found the solution:

`CMD + P - View: Toggle Panel Position`

You're welcome, future Mike, when you forget this in two weeks.

## Blog Posts/Videos I consumed

- [React Hooks, The Rebirth of State Management](https://blog.usejournal.com/react-hooks-the-rebirth-of-state-management-and-beyond-7d84f6026d87) - shows the awesomeness and flexibiity that React Hooks provides us in terms of state management.
- [Migrating to GraphQL](https://moonhighway.com/migrating-to-graphql) - a high level, backend focused approach to slowly migrating to GraphQL.
- [Intro to Immer](https://hackernoon.com/introducing-immer-immutability-the-easy-way-9d73d8f71cb3) - introductory blog post on what Immer is, how to use it and how it differs from other solutions.
- [How to use `useEffect` Deps Properly](https://www.youtube.com/watch?v=dIPvgHEM-2s) - Tutorial on how the `useEffect` hook can be conditionally run based on the second argument you pass to it.
- [Using `void` to make arrow functions return nothing](https://www.youtube.com/watch?v=OCTFOsCvcNs) - How the `void` operator works and when/if you should use it.
