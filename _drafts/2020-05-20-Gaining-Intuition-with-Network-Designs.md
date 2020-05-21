---
layout: post
comments: true
author: Justin Pietsch
title: Gaining Intuition with Network Designs
excerpt: In this post we look at an example of simulation in another field and think about how we can apply that to networking.
description: Why do we need better tools so that we can more easily gain intuition in networking?
---
I care a lot about network design. **I think it's too hard to design networks well**
and I think the tools to do it well don't exit. At least I haven't seen them. 
The lack of tools affects the 
biggest networks, but I think it affects all networks. I am especially 
concerned about how to design networks that are operatable, which requires 
a network that is understandable.

I saw a [youtube video of Stephen Wolfram](https://www.youtube.com/watch?v=kC6LHAv_lx0) 
learning Epidemiology so that he can 
model Covid-19 infection rate. If you don't know who he is,
[Stephen Wolfram](https://en.wikipedia.org/wiki/Stephen_Wolfram)
got a PhD from Caltech in Physics at age 20. He created Wolfram Research which produces 
Mathematica and he has been the CEO the whole time. 

You don't have to watch the whole thing :),
even in the first 5-10 minutes it's pretty interesting.
It's fascinating to watch him learn by try out different mental models he already
has to see if he can apply them to this problem. 
He starts with a very simple mental model of how to understand the problem. He then types
in a small equation, then he does another equation which models a network, etc. From these 
small experiments which don't take very long he gains some important intuition
about the problem, including that the initial conditions change the results significantly.These problems lend themselves to 
math' he's been working on how to model physics and other science processes in 
languages most of his life. I have no idea if he understands how to model Covid-19 
infection well, that's not the point, the point is about **how he easily uses his tools to
try out different models** to learn how to think about the problem.

**I want this for network engineering and design.** I want engineers 
(including me) to be able to try out new design ideas so that they can get an 
intuition about how to design networks. I'm not sure we can ever get to the 
speed that Wolfram tries out ideas in this video.
However, I'd love it if we could get to one day from ideas to trial. 
And then one hour and then maybe 10 minutes? I don't know. Just always getting better.

While the GUI is important, Wolfram is using a computer language to describe 
what he needs and then it is drawing picture. He writes about how important a 
[computation language](https://writings.stephenwolfram.com/2019/05/what-weve-built-is-a-computational-language-and-thats-very-important/)
to the problems he's trying to solve. We need a equivalent language or set of tools. 
It's not enough to just say do it in python, there at least needs to be a good set 
of abstractions and a framework to use them. 


There are some pieces to this puzzle that exist. Using vagrant to emulate 
network (https://elegantnetwork.github.io/posts/Network-Validation-with-Vagrant/)
is an important piece. Tools like [Batfish](https://www.batfish.org/) 
could make this simulation go more quickly, and they allow analysis to help 
understand how the experiment works. I think the linchpin 
is a way to describe and generate networks. This seems to be completely missing. 
It's too hard to come up with different designs and try them out. There is also 
the piece of describing tests and other assertions that must be true to know
that a network is working. (I was calling this intent based networking,
but that's not how the industry is using that term, I need something else.)

Here are some examples I'd like to be able to try: 
* I want to compare OSPF vs eBGP vs iBGP. How do they compare in convergence during various failures?  
* I'd like to compare a 2 tier Clos using chassis devices to a 3 tier Clos using pizza box routers. 
* How will eVPN effect my datacenternetwork convergence and resiliency if we 
have 100 different VNIs.

Let's image how this might look. I want to compare OSPF vs eBGP in a 2 tier Clos network
with 8 spines and 32 leaves. I type a command to create the topology. Then I add IP addresses rules.
Then I add OPSF to the topology. Finally I add a series of tests to stress the topology and tell me stats
about how it converges. I then do all that again, but instead with eBGP.

Getting all that is going to take a long time. Right now I would settle for being able to
create the two different sets of configuration easily. I'd like to be able to describe
the topology, addressing, and routing easily, and then get out configs. Maybe not just 
a couple expressions on a nice GUI, but even writing YAML and running a command line.
Then if I could spin up network simulation and getting running instances of these topologies.
Finally, I'd need some kind of test suite to stress the topology and give me those stats.

This seems like something that could be created now. It's not nearly as effortless as Wolfram
using Mathematica in the video, but it is a lot easier than we have now.


I think that every network engineer could gain from having 
easy ways to gain insight about networks. My partner Dinesh wrote a book 
[Cloud Native Data Center Networking](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608) 
and has a [github repository](https://github.com/ddutt/cloud-native-data-center-networking) 
in which you can download vagrant 
files to try out 18 different scenarios from the book. This is a great way 
to start getting and understanding of these concepts. Dinesh has been also 
consulting over the last 18 months. For most of his 
clients he builds them a vagrant network to simulate the ideas that he is 
presenting to them. These are successful, the clients like being able to understand
and try out the concepts that Dinesh is talking about.

I do think the software we've built, [Suzieq](https://github.com/netenglabs/suzieq), 
can help with this intuition because it can give you a summary of all that's 
going on in these scenarios.  

I hope readers try these tools out and then let us know what things what make 
it more useful. You might have an idea that we could bring about in hours, 
you might have an idea that might take years. 