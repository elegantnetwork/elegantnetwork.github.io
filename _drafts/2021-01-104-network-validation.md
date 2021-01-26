---
layout: post
comments: true
author: Justin Pietsch
title: Network Validation
excerpt: 
description: How do you validate that your network is working correctly? What does that mean, how can we make it useful?
---

Network Validation is an important idea in networking, but what does it mean practically? Of course, there's no official description of what it means, but we can talk about what we would like it to mean so that it can be useful. What we are trying to get to is a network that is trustworthy. If the business or organization using the network can't know if the network works correctly and that if it's failing the networking operations team knows it and will debug it quickly they can't trust the network. This gets in the way of the organizations ability to do it's primary goal. 

Many networks assume that they are working correctly if there aren't too many devices down and no customer is complaining. That's a very reactive position and as important as networking is to businesses, not where we want to be to [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). For the most part, network monitoring focuses on faults. Just because a device or interface failed doesn't mean the network is not healthy or valid. **We are using this to raise the level of engineering.** Knowing faults does not give you enough information about if your network is functioning correctly.

What we care about is **"Is my network is behaving as I think it should be?"** including when changes and failures occur. We want to minimize surprises. **Network surprises are too be avoided, at almost any cost.** They can come up when a device fails and the network doesn't adapt correctly, or when you add capacity and it doesn't add as much capacity as you thought, or you make a change and it breaks the network. Of course, that requires that I have an idea of how my network should be behaving and that I have a way to check that.

I don't think most people are asking these great questions about network validity . I'd bet most people want to implicitly know the answer to these questions, but can't. So lets examine some questions, what they mean to the network, and if we can answer them. Questions that I care about that I think are hard to answer. Some questions I ask to understand if my network is valid:

- Can I safely make a change?
- This change I just made, is it correct?
- The change that I'm proposing, do I have the configuration correct?
- Is my configuration correct everywhere in my network?
- Is my network working as expected?
- Do I understand my network?
- Do I have violations from vendor approved design or best practices?

Just being able to ask and answer these isn't enough. We need a way to systematically ask and answer these questions. If you make it systematic then you can rely on those answers. If you are doing it ad-hock, then you can keep getting surprised. Especially as you make changes automatically, network automation means applying changes more quickly, then you really want to have systematic checking that your network is what you want.

# Change Validation

Network Validation is a really big topic and actually combines many different ideas and questions. So where do we start? Let's start when where I think most network engineers start: "hen I make this change in my network, did the change work?" Actually, I think the better question is: **"When I make this change in my network, is my network still safe and is it operating the way that I intended?"**

## Post-Change Validation

In the most naïve case post-change validation is done after performing the change and by waiting for somebody to report that something is broken. You hope you have monitoring to show you that something is broken, but maybe it requires a customer to complain. I think we can safely say that is wildly inadequate: we want to be more proactive than that. The next step might be after the change to check traffic levels, or manually look to make sure that the configuration is applied correctly. This is no longer negligent, but we can do better. If checking it's manual, you might forget to completely check everything, and different engineers will check different things to make sure it's correct. Also, just because you check something doesn't mean you check correctly. We want automated checks of the network.

There's multiple ways to validate a change. You can check that it had the effect that you expected or you can check that the network is the way that you think it should be.  I think it's more robust to check that the networks is operating as expected rather than doing something like making sure the config is correct. If you think about the purpose of your change, what you are trying to accomplish, and then you can check for that, you have a better understanding at the end that the change worked correctly. Rather than checking that the configuration you typed has been applied, you check that the effect of the change is happening.


## Pre-Change Validation

If I have post-change validation, then I also want pre-change validation. Lot's of changes fail because the network wasn't the way the operator expected it to be when they made a change. For instance, if you are upgrading OSes, and have two devices in a pair, but one of them has some interfaces down, or BGP sessions down, then it won't have as much capacity as expected.

I don't have any idea how many people do pre-change checking, but it seems as important as post-change checking, and as automatable. If you don't have a system to do that automation, then it's harder. Again, you can use ansible or equivalent, but it's better to separate out the collection and normalization from the checking.

## Automating Change Validation

So we want a way to write automated checking after a change. This mostly likely requires getting data from devices. If the data is metrics, especially interface metrics than you probably can get that data from a monitoring system. But usually when you want to verify a change you need other data.

Let's go through a simple example: I need to upgrade all the devices in my network. This is the first time, so I'm going to go device by device. As part of the change I upload the latest software, shift traffic away from this device, then reboot. When it reboots I want to make sure that it's working correctly. What should I check? Some good indicators are if my protocol neighbors are up and trading routes. I can check to see if all the BGP neighbors that are configured to be up are up, all the OSPF or ISIS neighbors configured are up, and nothing is stuck. I probably do also want to check to make sure that there is traffic flowing through the box, maybe comparing before and after.

OK, so how do I automate all that. I can do all this with ansible. As part of the OS upgrade process it does the checks. I think it's hard to make these checks reusable, and it's really hard to make it generalizable for many different checks. In [Getting Network Device Operational Data from Ansible](https://blog.ipspace.net/2020/12/updated-ansible-parsing-content.html), Ivan Pepelnjak talks about how to write your own collection using Ansible, and this is a good idea, but there are much better ways of doing this systematically

What I've seen is that when you start creating a set of automated checks that collects data and does comparisons, it's hard to write generalized code that is easy to maintain over time. You mix in the collection with the check. THen you realize you need to deal with multiple OSes or a new version changes the output or something ugly like that. And your code gets messy.

What you want is a system that collects the data you might want, and puts that in a standard format. The checks are totally independent from the collection systems, making the checks much clearer and just focused on the problem at hand. The checks can be much easier to understand since they aren't deal with the format but just focused on the logic that you are trying to understand.


I would never want to do automated changes of the network without automated checks of the network.

## Continuous checking

If you have a systematic way of collecting data and writing tests that are easy to understand, you can run these tests all the time. 

# Correctness checking


## Finding

Another place to start is often, "we just had a failure in the network because of x, do we have x anywhere else in the network?" X might be an OS version with a bug, a bad configuration, a wrong design, something in the network that isn't what you want it to be. This isn't quite validation, it's more finding. However, 





## Checking that the change is correct

We've been talking about how to verify that before you make and change and after that your network is safe. But what about checking to make sure that the change is a correct and safe ever before you actually apply it. I think there are two important ways to do this: simulation or verification. 

I think [Batfish](http://www.batfish.org) is really great, and if I still had a network I'd be using it. It's a great idea for trying out a change




# Is my network behaving as I expect it to

the biggest problem is that it's hard to describe how it should be behaving.


How is my network supposed to behave?

What is a valid network? I think to answer depends on the context that you are in.

## checking operational state


## Validation against vendor or other best practices

## Intent based networking

Isn't this Intent Based Networking? It might be, as usual with networking marketing terms, it changes over time. That's why I'm trying to dive into what we really need it to be. It's definitely not all of IBN. IBN focused on being able to describe what it calls intent and then be able to configure the network and check as appropriate.

IBN most pushed by A and in RFC ..., I don't think it does all these things, though I think it is trying to.

I'm focusing on the validation piece (obvi), but I want to dive in deeper than I think IBN talks about.
what is intent based networking
different types of failures

# Diving into the details of in-validness

What's the right level of checking for validation? Is it low level, is it high level?
just because something is broken, doesn't mean you know how to fix it?


I'm not talking about formal verification, such as in batfish. That's another very interesting and useful field, but for another conversation.




Just knowing what has failed or is in fault isn't enough. Just knowing things that are over threshold or out of compliance isn't enough. Is it important that interface or device or session is down



# Solutions
Okay, we've identified a bunch of questions to ask, and thought about how we'd like to be able to answer those questions. Is there any software to help me do that? 

Another small plug for [Suzieq](https://github.com/netenglabs/suzieq), which we've written to help with these type of questions. Suzieq continuously gathers operational state from the network and puts it in a vender-neutral normalized form. This makes it easy to then be able to write checks that we've been talking about. Suzieq also makes it easy to search for things like OS version. Suzieq has checks called asserts. These are still a little too complicated for people to easily add the kind of checks that we've been talking about, but it has the right architecture to make a great health check service.




# questions
- 

other ideas I want to talk
- https://mustelids.atlassian.net/wiki/spaces/NETWORKING/pages/746749977/Suzieq+Validation+Product+Page
- turning alerts into something more useful
	- aggregation, and suppressions, etc

# Conversation

- what of these ways of validating do you think are important?
- what do you already have solutions for?
- what would you like solutions for?
- If this is too complex, let me know, ask me questions, and I'll see if I can have another go at it. I think these ideas are really important and it's likely I'm not telling the story well enough yet.

what am I trying to do here
- explain validation and it's interconnected parts
- see if anybody cares about any given part in specific
- see if the ideas of solving these things are done or 


