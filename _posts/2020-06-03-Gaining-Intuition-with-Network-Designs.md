---
layout: post
comments: true
author: Justin Pietsch
title: Gaining Intuition with Network Designs
excerpt: In this post we look at an example of simulation in another field and think about how we can apply that to networking.
description: Why do we need better tools so that we can more easily gain intuition in networking?
---
I care a lot about network design. I think **it's too hard to design networks well**
and I think that all the tools necessary to do it well don't exist. 
At least I haven't seen them. The lack of tools affects the 
biggest networks, but it affects all networks. I am especially 
concerned about how to design networks that are easy and safe to operate, which requires 
a network that is understandable.

What do I mean by network design? I mean all the decisions that you need to make to 
create a network. Picking the topology, the routing protocols, the router OS, 
the hardware, etc. There are so many different decisions 
that you need to make, and they interact. How do can I get a better intuition 
about what decisions are important and how different decisions interact?


[![youtube video of Stephen Wolfram](https://img.youtube.com/vi/kC6LHAv_lx0/hqdefault.jpg)](https://www.youtube.com/watch?v=kC6LHAv_lx0).

I like to learn from other engineering disciplines. 
In this video, [Stephen Wolfram](https://en.wikipedia.org/wiki/Stephen_Wolfram)
is learning Epidemiology so that he can 
model Covid-19 infection rate. If you don't know who he is, he
got a PhD from Caltech in Physics at age 20. He created Wolfram Research which produces 
Mathematica and he has been the CEO for more than 30 years. 

You don't have to watch the whole thing :),
even in the first 5-10 minutes it's pretty interesting.
It's fascinating to watch him learn by trying out different mental models he already
has to see if he can apply them to this new problem. 
He starts with a very simple mental model of how to understand the problem. He then types
in a small equation, then he does another equation which models a small worlds network, etc. 
From these small experiments which don't take very long he gains some important intuition
about the problem, including that the initial conditions change the results significantly.
I have no idea if he understands how to model Covid-19 
infection well, that's not the point, the point is about **how he easily uses his tools to
try out different models** to learn how to think about the problem.

**I want this for network engineering and design.** I want engineers 
(including me) to be able to try out new design ideas so that they can get an 
intuition about how to design networks. I'm not sure we can ever get to the 
speed that Wolfram tries out ideas in this video.
However, I'd love it if we could get to one day from ideas to trial. 
And then one hour and then maybe 10 minutes? I don't know. Just always getting better.
Wolfram's problems lend themselves to 
math; he's been working on how to model physics and other science processes in 
languages most of his life, so he has some advantages, but that doesn't mean we
should give up before we start.


Here are some examples of experiments I'd like to be able to try: 
* I want to compare OSPF vs eBGP vs iBGP. How do they compare in convergence during various failures?
Does one take appreciably more resources than the other?  
* I'd like to compare a 2 tier Clos using chassis devices to a 3 tier Clos using pizza box routers. 
Again, how does it deal with convergence?
* How will eVPN effect my datacenter network convergence and resiliency if we 
have 100 different VNIs.

Let's image how this might look. I want to compare OSPF vs eBGP in a 2 tier Clos network
with 8 spines and 32 leaves. I type a command to create the topology. Then I add IP addresses rules.
Then I add OPSF to the topology. Finally I add a series of tests to stress the topology 
and tell me stats about how it converges. I then do all that again, but instead with eBGP.
I'd also like to be able to pick from a catalog of different design options so that I
don't have to define everything myself. Wolfram gets to easily pull different networks
and different models from a pre-existing catalog.


While the GUI is important, Wolfram is using a computer language to describe 
what he needs and then it is drawing picture. He writes about how important a 
[computation language](https://writings.stephenwolfram.com/2019/05/what-weve-built-is-a-computational-language-and-thats-very-important/)
to the problems he's trying to solve. We need a equivalent language or set of tools. 
It's not enough to just say do it in python, there at least needs to be a good set 
of abstractions and a framework to use them. 

## What can you do today

Getting to this sophistication to work is going to take a long time. 
Right now I would settle for being able to
create the two different sets of configuration easily and then compare them. 
I'd like to be able to describe
the topology, addressing, and routing easily, and then get out configs. Maybe not just 
a couple expressions on a nice GUI, but even writing YAML and running a command line.
Then if I could spin up network simulation and getting running instances of these topologies.
Finally, I'd need some kind of test suite to stress the topology and give me those stats.

There are some pieces to this puzzle that exist. Using vagrant or (GNS3 or Eve-NG) to emulate 
networks (https://elegantnetwork.github.io/posts/Network-Validation-with-Vagrant/)
is an important piece. Using Ansible (or salt or Normir) to push automation configuration. 
There is no way that I know of that will allow me to easily describe whole network configurations that I want
 
Tools like [Suzieq](https://github.com/netenglabs/suzieq) ([Suzieq](https://elegantnetwork.github.io/posts/10ish_ways_to_explore_your_network_with_Suzieq/)) 
allow you to understand what your simulation
is doing in a easy way. One of the problems with trying to 
different network designs is that networking is a distributed problem and it's 
hard to get your mind around what is going on. Understanding your network in 
a holistic way is the point of Suzieq. Try it out. 


While there's not directly a catalog of options like Wolfram, my partner Dinesh
has started with some useful topologies to try out. He wrote a book, 
[Cloud Native Data Center Networking](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608) 
and has a [github repository](https://github.com/ddutt/cloud-native-data-center-networking) 
in which you can download vagrant 
files to try out 18 different scenarios from the book. This is a great way 
to start getting and understanding of these concepts. Dinesh has been also 
consulting over the last 18 months. For most of his 
clients he builds them a vagrant network to simulate the ideas that he is 
presenting to them. These are successful, the clients like being able to understand
and try out the concepts that Dinesh is talking about.
 
Another approach to simulation and verification is to use 
tools like [Batfish](https://www.batfish.org/) simulate and check
could make this simulation go more quickly, and they allow analysis to help 
understand how the experiment works. 

I think the linchpin 
is a way to describe and generate networks. This seems to be completely missing. 
It's too hard to come up with different designs and try them out. There is also 
the piece of describing tests and other assertions that must be true to know
that a network is working. (I was calling this intent based networking,
but that's not how the industry is using that term, I need something else.)


We can start trying out new ideas and building intuition now. 
It's not nearly as effortless as Wolfram
using Mathematica in the video, but it is it's possible and it's a very important
way to navigate complex fields like networking. I think that every 
network engineer could gain from having easy ways to gain insight about networks. 


I hope readers try these tools out and then let us know what things what make 
it more useful. You might have an idea that we could bring about in hours, 
you might have an idea that might take years. 

In the next post, we plan on talking about wrestling some of the vendor NOSes to simulate
with vagrant. This is another piece of friction in our ability to understand our networks.

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.