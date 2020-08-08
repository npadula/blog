+++
date = 2020-08-08
menu =  "main"
title = "Foundations & Intuition: Programming and Music"
tags = [
    "programming",
    "music",
    "theory",
    "thinking",
    "cognition"
]
+++

_Fair warning: Probably none of what I'm going to say below is new or original._

## Music and Missing Fundamentals

As a teenager, with the primary goal of _becoming cool_ in mind, I decided to learn how to play the bass. I bought a bass, downloaded For Whom the Bell Tolls' tabs, and started practicing. Fast forward a couple of years and I could sort-of-play more interesting songs, but, eventually, while learning tabs and practicing technique was fun, I started noticing limitations, which eventually (among other factors), led me to stop playing.

At the time,  I didn't really understand _what_ was preventing me from both playing more complex songs, or creating my own. Surrouding myself with several musician friends, and seeing them grow over the years, I now understand that what I was missing was both proper technique and, more importantly,  **music theory fundamentals**.

## Solving Programming Problems

I've been programming for ~10 years now, the last ~4 of them proffessionally. As any programmer or IT worker knows, there is a lot about programming that is related to intuition. While the work we do is often done by maintaining entire contexts in our head, many times, particularly when debugging, we end up working with limited information. In these situations, we typically end up executing an algorithm resembling the following:

- analyze problem
- formulate hypothesis
- test hypothesis
- repeat if necessary

The hypothesis formulation step relies on a couple of factors: our domain knowledge, our previous accumulated knowledge, and intuition. The less information we have to solve a given problem, the more important our intuition becomes. It's intuition what allow us to combine domain and accumulated knowledge to **infer** a new hypothesis, and by testing it, we are effectively gaining more information on the problem.

Now, if we fix our intuition to a given value, and knowing that our domain knowledge is limited (which is why we are relying on intuition in the first place), we can see that the efficiency of our information discovery algorithm depends on our previous accumulated knowledge.

What is the practical consequence of this? ___**the more we know, the faster we'll reach a solution**___ (although this is affected by _beginner vs expert minds_ and that kind of stuff). But, in the typical case, if we have a broader, deeper toolbox, we have more tools, and more ways to use those tools, and so we'll get to an answer in a more efficient manner. This phenomenon not only occurs when debugging, but also when building new features. The more knowledge we have, the bigger our possible solution space will be. 

A bigger solution space means that not only we have better chances of finding _a_ solution, but that we also are more likely to find a **better** solution. This is important in programming because, as we know, there are many ways of doing things, but some are better than others (in terms of technical debt).

## Foundations, expression and communication

The opposite of this is the exact problem I faced while playing bass: Since I had no music theory foundations, it was hard for me to understand more complex songs, and I had practically no musical constructs to combine into a new song of my own. While I may have had song ideas, I was uncapable of **expressing** them, both on the instrument and while communicating with fellow musicians.

This very same problem occurs in software development, and has implications of great importance when working within a team. Just like music theory is a framework for expressing ideas and communicating with other musicians, technical concepts and foundations are tools for expressing solutions and communicating those solutions with other developers.

 When designing software is a team effort, the whole team must understand and agree on the proposed design. If that doesn't occur, team members will act on whatever interpretation of the design they have constructed in their minds. The bigger the discrepancy between interpretations of a given design among team members, the bigger the probability of increasing technical debt.

This situation is an unavoidable epistemic constraint. It's a mannifestation of the communication barrier that we encounter in many other scenarios in daily life.

## Speaking the same language - _Shared concept surface_

With all that said, we can account for that constraint. 

When we learn about design patterns, we often hear that one of the benefits that they bring is that they facilitate communication about a given problem (or solution). Having a set of shared concepts helps us reduce the uncertainty in team communication and improve our signal-to-noise ratio. This not only applies to the classical design patterns but also to more general CS/programming concepts (architectural styles, data structures and algorithms, OO/Functional principles, and many others).

Knowing this, our best bet for reducing communication noise is to increase our shared concept surface. Since we cannot inject knowledge into brains like in _Matrix_, we have to settle for tools like documentation and code reviews. 

While I don't have much data to back it up, I feel like a way to make these tools more effective is by **focusing on the why**. If we provide a goal that can be evaluated (albeit in a fuzzy manner), team members that lack foundational concepts have a better chance at sensing that there may be a better way at hand. Knowing this, they can research better alternatives on their own or as a team.

Another good way of improving shared concept surface is to **explain non-solutions**. When we learn that a piece of technology is _not_ the best way to do something, we typically also learn why. Recognizing non-solutions can help identify similar mistakes, and perhaps come up with better solutions that address the root cause of whatever is wrong with the sub-optimal approach.

## Being aware of gaps

It's impossible to know it all, we'll always lack something that another developer considers essential. 

When we are in the role of someone who lacks certain fundamental skills or concepts, we should be aware of the heuristics that may allow us to bridge the gap with our team. When we occupy the role of someone who has to do design work, we need to always keep in mind the possible gaps in our team. 

While learning curves are not a hard constraint, the effort spent on them can affect iteration veocity and team throughput, and in certain business contexts, it may make sense to base our designs in a different set of foundational primitives that adapt more to our team.