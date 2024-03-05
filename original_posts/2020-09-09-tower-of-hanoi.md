layout: post
title:  "Doing Challenging Work - Featuring Tower of Hanoi"
---

> "Work is not hard — the floor is hard — work is **challenging**"                        \- Montessori teacher

Sometimes we know what we want to do. We even believe that we know _how_ to do it. We cannot imagine all of the steps from beginning to end, but we know enough to start. We also feel confident that we can figure out the upcoming steps as needed. And yet we hesitate and stall.  Some work doesn't _feel_ easy though doable. The cognitive load is heavy or maybe the emotional labor is large. The time horizon is long.  Is it that our minds are trying to protect our brain of the effort? 

What does this have to do with Tower of Hanoi? It will help me illustrate some observations — that may even qualify as deep insights — about work. Also it's a game and it's fun. We will do some _[Gedankenexperiment](https://en.wikipedia.org/wiki/Thought_experiment)s _(aka__ thought experiments), where we imagine the results without having to physically carry out the experiment to its conclusion.

Tower of Hanoi Game
-------------------

The objective for Tower of Hanoi is to move all of the disks from Peg A to Peg B while using Peg C as a temporary storing place, keeping in mind two rules - you can only move one disk at a time and you can only place a smaller disk on top of a larger disk (not the other way around).

You can play the [Tower of Hanoi](https://www.mathsisfun.com/games/towerofhanoi.html) game to get a feel for the rules. You'll also experience why the game gets challenging quickly as you increase the number of disks.

‌

![Tower of Hanoi](https://theleafnode.com/content/images/2020/10/tower_of_hanoi_20200911_165111.jpg)

 Upload 

Tower of Hanoi

‌

Imagine a tower with 2 disks. You can imagine the steps to solve ahead of time. This _feels_ easy. Now imagine a tower with 3 disks. You can work out all 7 steps, with some effort. You can mentally carry out the physical action. Not too bad. 

The process of solving for 3 disks is all we need to know to be able to move 10 disks, or 25 disks or 50 or a 100! But beyond 3 or 4 disks, most of us cannot imagine the moves ahead of time, or hold them all in our head at the same time. The number of moves for 3 disks is 7, the number of moves for 4 disks is 15, and for 5 disks it's 31. So this makes sense that we can't easily imagine the steps beyond 3 disks. As most of us can only hold 4 to 7 distinct things in our short term memory at a time. In this case the number of moves increases exponentially to n, the number of disks.

### The Solution

However what is fascinating about the Tower of Hanoi is that you can get to a logical place in your mind where you're _confident_ that you can solve the game for any number n (given sufficient time of course). Here's exactly how we can _intuitively_ think about the solution: in order to move n disks from Peg A to Peg B, we have to know how to 

1\. Move n-1 disks from Peg A to Peg C, the temporary peg. 

2\. Then move the last disk (the largest disk) from Peg A to Peg B, the target peg. 

3\. Then use the _same process_ used in step 1 to move the n-1 disks, currently on Peg C over to Peg B, the target peg.

You end up with all n disks on Peg B, which was the goal. These are the only steps you need to know. But what about this 'process' for moving n-1 disks? How do we do that? 

Imagine n = 1, meaning the game is to move 1 disk. Well that's easy right, just pick it up and, you know, move it. What I am saying is that if we know how to move 1 disk, we know how to move 20 disks? Yes, you read that right. In order to move 20 disks, you need to know how to move 19 disks. But, to move 19 disks, we only need to know how to move 18 disks...and so on, down to...in order to move 3 disks we only need to know how to move 2 disks and for 2 disks we need only know how to move 1 disk. Aha! and we know how to do that.*

 This 'proof' although logically sound may not _feel_ satisfying. Why is that?

### Why the Solution feels Hard

Maybe it has to do with the effort involved? It doesn't feel easy. We cannot hold all of the moves needed to get to the solution in our head ahead of time. Even after we logically convince ourselves that a solution is possible, we have to exercise patience (and faith?) to sustain the effort required.

The number of moves increases exponentially. For n disks the number of moves for a solution is 2n - 1. Let's calculate this in terms of time. Assume it takes just two seconds to move each piece, and we work continuously without any thinking pauses.

For n=10, it would take 1023 moves. With two seconds per move, it would take 2046 seconds or about 34 minutes. This is 34 minutes of focused effort. It's not a 'mindless' effort, as we need to keep track of what we've already done and where we are in the The Plan. This can be aided by writing things down, crossing off moves as we carry them out, etc. But it still feels like a lot of effort

Enough with the Game
--------------------

So couldn't we apply this to real life hard problems - for example iterating on ideas to build a business? We know what to do. We are confident that we know how to start and figure out the upcoming steps. All we need to do is sustain effort for 34 weeks or maybe it can be more like 34 months — one move at a time. ⚡

‌

* * *

‌

\* If you're familiar with the concept of Recursion in computer science, you would recognize this as a classic example of a recursive solution with a base case, and a little proof by induction thrown in. Recursion is a powerful concept in programming. While we cannot hold all of the moves in our head, we can easily write a computer program to do this using recursion.