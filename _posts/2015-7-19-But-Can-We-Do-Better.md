---
layout: post
title: testing in prod is bad but that is what is happening
---

When I first learned to program in high school and college, the emphasis was on correctness and not on performance. When I taught students programming in college as a teaching assistant, I would often point out a suboptimal code path to the student. I can't tell you how many times I heard the following phrase: "Well, processors are fast enough these days that it doesn't really matter. Even still, the compiler will optimize it away". 

*No! Stop that right now!* The tireless work of the industry to bring you faster processors and better compilers is not an excuse for the programmer to write subpar code! 
 
In fact, I probably said that one day, but then I grew up to be a systems programmer where saving a few extra clock cycles could be the difference between a laggy experience and a smooth one for a user. Optimization and performance is the name of the game. 

"But Michelle", you say, "My job isn't anything like that. It's not such a big deal if things aren't optimized like crazy". Fine, but what job are you in that you don't want to give your users the absolute smoothest experience in town? Even if you are only programming for practice and nobody will ever see your finished product, I urge you to think about performance even if it slows you down at first. 
Why? It becomes natural after a while to use performant patterns and you'll end up with faster code without even thinking about it. 

This article uses C for the examples since that's what's easiest for me to write, but I am avoiding using things specific to C so that any programmer can understand and apply them. 

---

*__Time for a game of "Can We Do Better"!__*

**Problem:** Write a simple algorithm that inserts a number into a table using a hash that takes a seed, such that inserting `x` into a table with seed 0 would look like: 
