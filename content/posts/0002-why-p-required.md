---
title: "The Tyranny of the CAP Theorem"
date: 2019-03-02T01:36:56-08:00
draft: true
toc: false
images:
tags:
  - development
  - distributed systems
  - theory
---

NASA Flight Engineer and Astronaut Don Pettit wrote a great article about what
he calls [The Tyranny of the Rocket Equation]
(https://www.nasa.gov/mission_pages/station/expeditions/expedition30/tryanny.html).

The TL;DR is that there are tradeoffs you're forced to make when designing
rockets that puts an absolute upper bound on what that technology is capable of.
Specifically for rockets, the problem we face is one of recursion.

In order to get more mass into orbit, you need to use more fuel.  However, that
fuel also adds mass.  Luckily the fuel produces enough thrust to push it's own
weight and a little bit more, but these returns diminish as the rocket gets
larger.

There isn't much that can be done to avoid this.  It's a direct result
of physical constraints we have;  The amount of thrust per unit weight we can
get from known fuels, and the acceleration of gravity.  However there are some
clever engineering tricks that can be used to seemingly cheat the laws of nature
and do better, which we'll get into later on.

In distributed systems, we have our own tyrannical laws of nature to contend
with.  One of the biggest issues we face is that the speed of light, the fastest
possible speed in the universe, bar none, is, well...  really slow.  And that
forces us to make compromises.

The CAP Theorem, one of the seminal theories in distributed systems engineering
describes one of these compromises.

## CAP Theorem Refresher

I'm not normally a fan of spelling out the definitions of things as part of
blogs, but in this case I think it's important for explaining part of the CAP
theorem that's often blindly accepted.

The theorem, as proposed by Eric Brewer, is summarized
[on Wikipedia](https://en.wikipedia.org/wiki/CAP_theorem) as:

> _It is impossible for a distributed data store to simultaneously provide more_
> _than two out of the following three guarantees:_
>
> - _\(C)onsistency: Every read receives the most recent write or an error_
> - _(A)vailability: Every request receives a (non-error) response – without the_
> _guarantee that it contains the most recent write_
> - _(P)artition tolerance: The system continues to operate despite an arbitrary_
> _number of messages being dropped (or delayed) by the network between nodes_

And it's around this time that the nearest distributed systems engineer is
contractually obligated to chime in and remind you "And don't forget, you're
forced to pick partition tolerance!"

### Why not the CA Theorem?

When I first learned about the CAP theorem, it really bothered me that it presents
3 choices, and forces one of the three onto you.  This is in large part due to
the differences of theoretical and practical computer science.

The [actual paper](https://dl.acm.org/citation.cfm?id=564601) describing the CAP
theorem uses much more verbose and formal definitions for Consistency,
Availability, and Partition tolerance than what we are using here.  Focusing
entirely on theoretical system behavior, and using precise definitions, 

Let's look at that definition of Partition tolerance again:

> _The system continues to operate despite an arbitrary number of messages_
> _being dropped <b>(or delayed)</b> by the network between nodes_

Can you think of any situation in an actual computer network where messages may
be delayed?  I'm sure you can think of plenty, even if you've never worked with
computers before.  You know that websites get slower or hang as your phone gets
too far from a wifi router, or go through a tunnel while connected to LTE.

A more difficult question may actually be "Can you think of any situation where messages
are **not** getting delayed?"

The messages aren't being sent between devices instantaneously.  There is always
a delay due to the latency of the network.  And this is really at the heart of
why the "P" in CAP theorem
