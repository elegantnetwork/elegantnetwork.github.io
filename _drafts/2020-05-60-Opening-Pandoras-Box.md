---
layout: post
comments: true
author: Justin Pietsch
title: Opening Pandora's Box -- Why Network Engineers Should Write Code
excerpt: 
description: Network Engineers are told to write code, but why? How important is it?
---

The topic of writing code can be a controversial topic for network engineers. 
Network Engineers already have to learn and know so many technologies, and 
it’s not fun to find out that your ideas about 
your career that you’ve worked very hard for are threatened by something 
else. I want to talk about why writing code can help you be a better 
network engineer. And also why our 
industry needs us to write code to be better engineers.

The usual argument starts with "Automate all the Things!". This is obvious, 
and it's important, but not the most important.  If you start automating 
the work you already do, you gain super powers. You can write tools to help 
you debug, or look at data more helpfully. 
Maybe you automate Tcpdump collection, or you automate collecting data 
just as pings at certain intervals. Or maybe you take data from several 
devices and combine them together to debug a problem. All of these are 
useful. Good engineers make tools for themselves. There are a 
lot of paper cut issues in network engineering, pick one and fix it.

However, I want to talk about something that I think is more important. 
The main reason I want Network Engineers to write software is so that you 
think differently, more powerfully. When you write software and try to learn 
how to do it well you start thinking about algorithms, abstractions, clean 
code, and testing, among other things. You then learn why they are so 
important and how you take those ideas into networking. The goal is to 
learn how to be a better engineer, how to understand trade-offs, how to 
make better abstractions, how to steal ideas from other disciplines, and 
how to think better.

For instance, let's say you need to build five small datacenter networks, 
and then in a year, each of those will grow, but at different rates. If 
you don't think in software, these networks will be mostly the same, 
but not quite. Depending exactly on when you get what switch, or need 
what feature, you might not have them all cabled exactly the same. 
In datacenter 3, switch 7, interface is connected to switch 9, and in 
datacenter 2 it's connected to switch 8. Not quite the same. If there's 
just one thing different, no big deal, but there's never just one thing 
different. Each of those differences adds up, sometimes they even multiply, 
the network becomes less and less coherent and less understandable. 
When you make changes you have to remember more 
things. When you debug, there are more things to check.

When things are coherent, then you can think in abstractions. For instance, 
this layer of the the network always does these these three things, including 
most of our interesting policy. Then you know where to put policy and you 
don't put it all over the place. When you make these abstractions and they 
are followed, then again you have a more coherent design which is a more 
understandable design.

You also can step back and look at how you are approaching a problem. 
When you first start writing code to do your job, you will automate 
something you already did. But as you go along, maybe you'll realize 
that there are different ways to approach the problem, and because you now 
have software and computers at your command, you can approach a problem in 
a way that doesn't make sense for a human, but is actually a better way of 
solving something. Hopefully you will start thinking of your network as a system, 
rather than individual boxes with independent configuration.

If you think about testing code, then maybe you'll start thinking about 
how to build networks that are testable. This kind of thinking often leads 
to simpler code, it can also lead to simpler networks. 

Another important reason to write code is that as you write code, you can 
take the time to think about how you would change your network so that it is 
more automatable. Writing network management software is hard, making it for an 
arbitrary network is next to impossible. In my experience, networks that 
are easier to automate are easier to scale, operate, and change. This is a 
subtle, but I think key secret to some of my more successful designs. 
Because I was trying to figure out how to automate and operate from the beginning, 
those networks scale much better.

And the last piece I will add is that as you write software, you begin to think 
about what other software you want and you can get better at asking for better 
software. Tools for network engineers are generally terrible, and a big reason 
for that is that network engineers don't ask for good tools. We often ask for the 
wrong thing, or don't know how to communicate what we actually need. Also, 
understanding software will help you understand when you ask for things that 
are too complicated. Sometimes vendors give you exactly what you asked for, and 
it’s overly complex and makes your network more fragile. 

 

Where to get started? That's a very good question. I don't really know. Probably 
pick up a python book. For whatever reason the networking community has picked 
Python. As you gain your footing, be sure to not focus only on how to automate 
what you already know how to do, though that is the place to start.  Learn how 
to think in abstractions and algorithms.

https://nrelabs.io/ looks like a great place to learn automating networking.


Languages do change the way we think. Try learning Python and think about 
how to change the way you do your job. Even when not writing code, it can 
make you a better network engineer.