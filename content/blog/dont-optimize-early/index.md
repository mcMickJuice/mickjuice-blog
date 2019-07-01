---
title: 'Dont Optimize Your Code*'
date: '2019-06-30T19:00:00Z'
---

![Blazing](./blazing.jpg)

<sub>Live look at a blazing fast app</sub>

***Wait until you notice performance degradation, first**

We all want our apps to run blazing fast. As a developer, owning a poorly performing app wears like a badge of shame.  However, I do believe that a developer can focus too much on performance enhancement and in so doing, miss out on great learning opportunities as well as introduce more complexity to a codebase than is necessary.

The following summarizes my opinion on performance enhancement:

> Do not introduce performance optimizations until you have a reason why.  Optimized code is generally harder to understand code.  Deferring these optimizations until they're needed affords the developer and their team an opportunity to learn about performance profiling

_Note that I'm primarily talking about performance on the frontend as it relates to the rendering of web apps.  I've also been coding web apps almost exclusively with React over the past 2+ years so YMMV if you use other frontend frameworks, though the principles should still hold._

Let me expand on why I hold these sentiments.

## Focus on delivering features

When writing code, there is only so much we can hold in our heads.  If you're coding a non-trivial feature, it's likely that you are holding a number of considerations in your head. Why introduce more overhead to your mental process when your primary focus should be on delivering the feature?

I often see handwringing in the programming community around the question of "am I doing this right?".  This is often from junior devs who are worried that every line of code they are writing is wrong but I also see it with newcomers to a framework.  This is especially true with React where there really isn't a convention or "right way" to get something done.  

This applies doubly when it comes to performance as React's rendering model might cause a developer to think that unnecessary renders are the root of all evil.  Should you avoid unnecessary renders? Maybe (it depends on what's being rendered).  Should you worry about this when you're building out a feature? Absolutely not.

Break your problems into little piles ([Be like the squirrel, girl](https://www.youtube.com/watch?v=oOS00ttAblQ)).  Focus on the problem at hand (your new sick feature) and focus on performance enhancements when they come up.

## Your performance optimizations should be intentional

When you add a line of code, you should know the purpose of that line of code:

- this function makes a call to our API and returns the data I need
- this component renders a form
- this variable is used to store the current state of this loop

Your approach to writing performance optimizing code should be the same.  If you're adding a `useMemo` call to your React component, why?  Is it addressing a perf issue you're seeing when running the app? Or is it because you want to ward off potential issues that you foresee coming in the future?  If the reason is the latter, I'd suggest you hold off until it's absolutely necessary.

The purpose of code that enhances an app's performance should actually result in better app performance.  The only way to ensure this is true is by following these steps:
- identify a performance problem in your application
- find the source of the problem
- introduce code in an attempt to fix the problem
- measure the app's performance before and after to ensure that your change fixed the issue

Any performance related code that is introduced that is not a result of following the above steps can be unnecessary and/or problematic.

This approach should result in easier to understand code which will make your team and your future self (_"Why the hell did I do this?"_ - you, 6 months from now) happier.  It's also possible that adding these kinds of optimizations when they aren't necessary could result in a [_less_ performant application](https://kentcdodds.com/blog/usememo-and-usecallback) or worse, very confusing bugs (e.g. an improperly configured `shouldComponentUpdate` resulting a component not re-rendering when it should).

## Use this as a learning experience

While the adage ["Premature Optimization is the Root of All Evil"](https://en.wikipedia.org/wiki/Program_optimization#When_to_optimize) mostly relates to the prevention of complicated and unnecessary code, I contend that blindly adding performance optimizations without an evident performance issue can rob you and your team of a learning opportunity.

When teaching others, I always stress on understanding the "Why".  Why are you adding this code here? What problem is it solving? Why use React? What value does it have in comparison to other UI libraries?  Why are you using a `map` function here? What are the alternatives?

While "What" and "How" are surface level questions, being able to answer the "Why" demonstrates a full understanding of a given concept and generally ensures repeatability.

If you take an intentional approach to adding performance enhancing code, you will know _why_ that code is being added and _why_ your code was performing poorly in the first place.  Moreover, you can share this information with your team members who will also understand the _why_ around the code you're adding.  Simply adding `useMemo` or `PureComponent` all over your code because you think it's needed robs you of the opportunity to seek the "Why" and might also prevent others on your team of doing so as well.

## Conclusion

Over the past 2 years in React-land, I've noticed only a small set of issues that affect performance:

- jank related to putting too many elements on a page
- slow loading times due to fetching a ton of data from a server
- hot loop code (i.e. a `map`) on a large array of items where each operation is relatively slow (e.g. converting a string to a `Moment` object)

Each issue took me a few hours to identify and fix, though along the way, I learned a lot more about Chrome Dev Tools and strategies to utilize when debugging performance issues.  Futhermore, none of the issues above were related to my React code, rather how my app was architected.

While it might be tempting to use `useMemo` and `useCallback` throughout your app when you initially code a feature, ask yourself: 

- Why am I adding this?
- Is this solving a current problem?
- What is the worst case scenario if I didn't add these performance optimizations?

I don't believe that you should completely ignore performance optimizations, I believe that you should focus on functionality first and only introduce optimizations when you have good reason to do so.  Doing so will keep your code clean and intentional and will allow you to grow as a developer.