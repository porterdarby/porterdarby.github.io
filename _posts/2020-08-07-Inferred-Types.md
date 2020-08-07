---
layout: post
title: Being careful with Inferred types
tags: Scala, static typing
---

I was recently working on a small hobby project of mine which was attempting to implement a niave version of [instant-runoff voting](https://en.wikipedia.org/wiki/Ranked_voting) calculator. The goal was to produce a small piece of code which could take in a collection of candidates and then, based on instant-runoff ballots, determine the winner of the election.

I eventually came up with the following:

<script src="https://gist.github.com/porterdarby/9cc97f4ec510107c5ce05869bdde2a7d.js"></script>

It's not much, but it works.

The interesting thing I found while doing this was how much inferred types are useful, and how much they actually can mess you up. Let's take for instance this line:

```scala
val topCandidate = scores.maxBy(_._1)
```

Can you see the problem? The attempt here is to get the candidate with the most votes after a "cycle" of the runoff. This code compiles, this code works does what it says -- it goes ahead and gets the top candidate pair based on the first pair element's value. That's not a problem, right? Well, the first value in the pair is the candidate's name -- a String. I didn't realize it at first, but I was wondering why the process almostalways needed to go to the 3 reduction from 2 to 1 candidates. I realized that I had mad an error -- I wasn't sure if I was finding the pair based on the maximum value of the ballot count, I was getting the max value based on the name. So I made this change:

```scala
val topCandidate = scores.maxBy(_._2)
```

One character difference, but it fixes everything.

Now, what is in the code that I originally posted? It's

```scala
val topCandidate = scores.maxBy[Int](_._2)
```

After thinking about it, I had messed up originally because I didn't use the fact that there is type checking in Scala to my advantage. If I had started with the following:

```scala
val topCandidate = scores.maxBy[Int](_._1)
```

The compiler would've thrown me an error. But I didn't do that, so I didn't get any errors. Because you can find the "max" of a String or an Int, I unintentionally shot myself in the foot.

Now, there are different ways I could've solved this. The first is to create a case class for the pair and use that to my advantage -- can't really screw up the difference between `_._1` and `_._2` if they are instead `_.name` and `_.votes`. There might've been other data structures I could've used to calculate this. Honestly, there might be fully different ways to do the calculations. Regardless, I didn't do the type checking and let Scala do it -- it functionally worked and it wasn't incorrect, but didn't work the way that I intended.

Moral of the story -- you can do a lot of stuff with inferred types, but you still might just need to add in explicit type checks somewhere to make sure you don't shoot yourself in the foot.