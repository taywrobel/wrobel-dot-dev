---
title: "The Tyranny of the CAP Theorem"
date: 2019-03-02T01:36:56-08:00
draft: true
toc: true
images:
tags:
  - development
  - distributed systems
  - theory
---

NASA Flight Engineer and Astronaut Don Pettit wrote a great article in 2012 presenting what he calls [The Tyranny of
the Rocket Equation](https://www.nasa.gov/mission_pages/station/expeditions/expedition30/tryanny.html).

You really should read that article, but the TL;DR is that there are compromises we're forced to make when designing
rockets that puts an absolute upper bound on what that technology is capable of. Specifically for rockets, the problem
we face is one of recursion.

In order to get more mass into orbit, you need to use more fuel.  However, that fuel also adds mass.  Luckily the fuel
produces enough thrust to push it's own weight and a little bit more, but these returns diminish as the rocket gets
larger.

There isn't much that can be done to avoid this.  It's a direct result of physical constraints we have;  The amount of
thrust per unit weight we can get from known fuels, and the acceleration of gravity.  However there are some clever
engineering tricks that can be used to seemingly cheat the laws of nature and do better, which we'll get into later on.

In distributed systems, we have our own tyrannical laws of nature to contend with.  One of the biggest issues we face is
that the speed of light, the fastest possible speed in the universe, bar none, is, well...  pretty slow.  And that
forces us to make compromises.

The CAP Theorem, one of the seminal theories in distributed systems engineering, describes one such compromise.

# CAP Theorem Refresher

I'm not normally a fan of spelling out the definitions of things as part of
blogs, but in this case I think it's important for explaining part of the CAP
theorem that's often not well covered.

The theorem, as proposed by Eric Brewer, is summarized [on Wikipedia](https://en.wikipedia.org/wiki/CAP_theorem) as:

> _It is impossible for a distributed data store to simultaneously provide more_
> _than two out of the following three guarantees:_
>
> - _\(C)onsistency: Every read receives the most recent write or an error_
> - _(A)vailability: Every request receives a (non-error) response – without the guarantee that it contains the most_
> _recent write_
> - _(P)artition tolerance: The system continues to operate despite an arbitrary number of messages being dropped (or_
> _delayed) by the network between nodes_

And it's around this time that the nearest distributed systems engineer is contractually obligated to chime in and
remind you "And don't forget, you're forced to pick partition tolerance!"

## Why not the CA Theorem?

When I first learned about the CAP theorem, it really bothered me that it presents 3 choices, and forces one of the
three onto you.  This is in large part due to the differences of theoretical and practical computer science.

The [actual paper](https://dl.acm.org/citation.cfm?id=564601) describing the CAP theorem uses much more verbose and
formal definitions for Consistency, Availability, and Partition tolerance than what we are using here.  It's written
from an entirely theoretical perspective, which is necessary for making provable assertions, but often is at odds with
practical considerations of a real world system.

To illustrate this, let's look at that definition of Partition tolerance again:

> _The system continues to operate despite an arbitrary number of messages being dropped <b>(or delayed)</b> by the_
> _network between nodes_

I'm sure you can think of plenty of situations where messages may be delayed in an actual computer network.  Even
completely non-technical people probably can. We've all had websites get slower or hang as your device gets too far
from the wifi router, or when we go through a tunnel while connected to LTE.

A more difficult question is actually "Can you think of any situation in a real world network where messages are **not**
getting delayed?"

The messages aren't being sent between devices instantaneously.  There is always a delay due to, at a minimum, the
latency of the network.  And this is really at the heart of why the "P" in CAP theorem is required:

> **Latency is a network partition**.

The reality is that any network is in a constant state of partition.  The delay in messages under normal operation is
normally fairly small and consistent, so is easier to reason about, but from the point of view of the computers
messaging one another, they have no guarantee of how quickly they'll get a response, if at all.

Any system that does not provide partition tolerance will simply fail to operate in a real world network.  Data will
almost immediately become corrupted, or the system will crash.

# The Wrobel Global Deli

Now that we understand why we're forced to support partition tolerance in our distributed systems, let's look at the
choice that we get to make; the tradeoff between consistency and availability.

Let's say that I want to make the world's first globally distributed deli.  I have no idea how we're going to transport
food around the world, but that sounds like a good problem for someone else.  I do know one thing however, which is that
our Deli needs one of those classic take a number systems:

{{< figure src="/take-a-ticket.jpeg" alt="Take a ticket machine" position="center" style="border-radius: 8px;"
caption="Holds society together" captionPosition="center" >}}

## Consistency vs. Availability

We all know that at the

## Light is Just Too Slow

The speed of light in a vacuum is the universe's speed limit.  It is the absolute maximum speed that information can
travel and is approximately 3×10<sup>8</sup> (300,000,000) meters per second (or about 670 million miles per hour).

These speeds seem (fittingly) astronomical.  In a world where we generally stick to an area of 10's to 100's of miles
around where we live, the notion of something moving millions of miles in an hour is almost incomprehensible.

But in a globally connected network, we need to adjust our frame of view to account for how large the earth as a whole
is. So how large is it? Well, near as makes no difference for this discussion, the circumference of the earth is
[40,000km](https://www.space.com/17638-how-big-is-earth.html), or 4×10<sup>7</sup> meters.

Let's do some back-of-the-digital-envelope math (aka pop open a javascript interpreter) to determine the minimum amount
of time it takes a signal takes to get around the earth.

```js
const speedOfLight = 3e8;
const circumference = 4e7;

const latency = circumference / speedOfLight;
console.log(latency); // => 0.13333
```

So it takes ~133 milliseconds for a message to make a round trip around the earth. And to be clear, we're being very
nice with our assumptions here making this a good theoretical lower bound.

We are assuming that we can move light around the planet as quickly as if it were in a vacuum, not factoring in network
topology or latency, or the slower speed of light through fiber optic or copper cables.

In our human timescales, this again doesn't seem that bad.  It is actually over twice as fast as the blink of an eye
(300 - 400 milliseconds).  But let's flip the numbers to figure out the max throughput for the system described above:

```js
const speedOfLight = 3e8;
const circumference = 4e7;

const throughput = speedOfLight / circumference;
console.log(throughput); // => 7.5
```

7.5 messages per second. For reference, credit card processor Visa
[reports](https://usa.visa.com/run-your-business/small-business-tools/retail.html) that as of 2010, it was capable of
processing 24,000 transactions per second, and a transaction usually consists of multiple messages passing between nodes
in the system.

Houston, we have a problem.

# Cheating the System

There came a point when we were hitting the limits of what we could do with rockets.  What was preventing us from making
bigger rockets was all the extra weight you need to carry along all the way to orbit.  The vast majority of a rocket by
mass is fuel (about 85%), but building bigger rockets required more structural material, and thicker/stronger materials
to handle the larger size.

But engineers had a bit of a crazy idea to address that.  What if we designed the rocket to disassemble itself as it
takes off?  Rather put all the fuel in one tank, we could break it up into several sections, and burn the fuel one
section at a time.  Then when a given section runs out of fuel, we separate it from rest of the rocket in a nice safe
way for a vehicle that's 85% rocket propellant: [Explosives](https://ntrs.nasa.gov/archive/nasa/casi.ntrs.nasa.gov/20090015395.pdf)!

<div class="center" style="text-align: center">
  <video width="512" height="288" controls loop video controls autoplay>
      <source src="/booster-separation.mp4" type="video/mp4">
      Your browser does not support the video tag.
  </video>
</div>

Crazy as it may be, this actually worked.  By shedding weight in flight, each stage of the rocket acted effectively like
its own fresh rocket being launched, on top of the velocity it had already picked up.  There is still a maximum that
can be effectively achieved, but we were able to increase it beyond what was one the theoretical max with a bit of
clever engineering.

So what kinds of tricks do we have up our sleeves for distributed systems? It may surprise you that it **also** involves
explosives!  *(Okay, not really, but how awesome would that be?)*

## More Nodes (Quorum)


## Partial Ordering (Logical Clocks)
