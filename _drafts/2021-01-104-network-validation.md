---
layout: post
comments: true
author: Justin Pietsch
title: Network Validation
excerpt: 
description: How do you validate that your network is working correctly? What does that mean, how can we make it useful? How can we test our networks to know that they are correct?
---

Network Validation is an important idea in networking, but what does it mean practically? Of course, there's no official description of what it means, but we can talk about what we would like it to mean so that it can be useful. What we are trying to get to is a network that is trustworthy. If the business or organization using the network can't know if the network works correctly and that if it's failing the networking operations team knows it and will debug it quickly they can't trust the network. This gets in the way of the organizations ability to do it's primary goal. 

Many networks assume that they are working correctly if there aren't too many devices down and no customer is complaining. That's a very reactive position and as important as networking is to businesses, not where we want to be to [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). For the most part, network monitoring focuses on faults. Just because a device or interface failed doesn't mean the network is not healthy or valid. **We are using this to raise the level of engineering.** Knowing faults does not give you enough information about if your network is functioning correctly.

What we care about is **"Is my network is behaving as I think it should be?"** including when changes and failures occur. We want to minimize surprises. **Network surprises are too be avoided, at almost any cost.** They can come up when a device fails and the network doesn't adapt correctly, or when you add capacity and it doesn't add as much capacity as you thought, or you make a change and it breaks the network. Of course, that requires that I have an idea of how my network should be behaving and that I have a way to check that.

I don't think most people are asking these great questions about network validity . I'd bet most people want to implicitly know the answer to these questions, but can't. So lets examine some questions, what they mean to the network, and if we can answer them. Questions that I care about that I think are hard to answer. Some questions I ask to understand if my network is valid:

- Can I safely make a change?
- This change I just made, is it correct?
- The change that I'm proposing, do I have the configuration correct?
- Is my network working as expected?
- Do I understand my network?
- Do I have violations from vendor approved design or best practices?

Just being able to ask and answer these isn't enough. We need a way to systematically ask and answer these questions. If you make it systematic then you can rely on those answers. If you are doing it ad-hock, then you can keep getting surprised. Especially as you make changes automatically, network automation means applying changes more quickly, then you really want to have systematic checking that your network is what you want.

One of the trickiest thing about this problem area is that we as network operators have a lot of assumptions in our heads about how the network is supposed to be working and how it is working. We need to get this assumptions out of our heads and into software that can confirm or deny your assumptions. You can think of network validation as ways to be testing your network to see if it is operating as expected. What are the kinds of tests you think you need to understand your network?

# Change Validation

Network Validation is a really big topic and actually combines many different ideas and questions. So where do we start? Let's start when where I think most network engineers start: "hen I make this change in my network, did the change work?" Actually, I think the better question is: **"When I make this change in my network, is my network still safe and is it operating the way that I intended?"**

## Post-Change Validation

In the most naïve case post-change validation is done after performing the change and by waiting for somebody to report that something is broken. You hope you have monitoring to show you that something is broken, but maybe it requires a customer to complain. I think we can safely say that is wildly inadequate: we want to be more proactive than that. The next step might be after the change to check traffic levels, or manually look to make sure that the configuration is applied correctly. This is no longer negligent, but we can do better. If checking it's manual, you might forget to completely check everything, and different engineers will check different things to make sure it's correct. Also, just because you check something doesn't mean you check correctly. We want automated checks of the network.

There's multiple ways to validate a change. You can check that it had the effect that you expected or you can check that the network is the way that you think it should be.  I think it's more robust to check that the networks is operating as expected rather than doing something like making sure the config is correct. If you think about the purpose of your change, what you are trying to accomplish, and then you can check for that, you have a better understanding at the end that the change worked correctly. Rather than checking that the configuration you typed has been applied, you check that the effect of the change is happening.

So what do I do if my post-change validation fails? The best answer is to roll back your change. You want to get the network back to valid, understood, state as soon as possible, and you don't understand why your validation failed; it's not what you expected.

## Pre-Change Validation

If I have post-change validation, then I also want pre-change validation. Lot's of changes fail because the network wasn't the way the operator expected it to be when they made a change. For instance, if you are upgrading OSes, and have two devices in a pair, but one of them has some interfaces down, or BGP sessions down, then it won't have as much capacity as expected.

I don't have any idea how many people do pre-change checking, but it seems as important as post-change checking, and as automatable. If you don't have a system to do that automation, then it's harder. Again, you can use ansible or equivalent, but it's better to separate out the collection and normalization from the checking.

You want to know that you are safe to make a change before you make the change. Worst case scenario is that the network wasn't in the state you expected, you make a change, and something breaks. If the change was fine, but the network was broken before you made the change, then you could have prevented that outages.

## Automating Pre/Post Change Validation

So we want a way to write automated checking after a change. This mostly likely requires getting data from devices. If the data is metrics, especially interface metrics than you probably can get that data from a monitoring system. But usually when you want to verify a change you need other data.

Let's go through a simple example: I need to upgrade all the devices in my network. This is the first time, so I'm going to go device by device. As part of the change I upload the latest software, shift traffic away from this device, then reboot. When it reboots I want to make sure that it's working correctly. What should I check? Some good indicators are if my protocol neighbors are up and trading routes. I can check to see if all the BGP neighbors that are configured to be up are up, all the OSPF or ISIS neighbors configured are up, and nothing is stuck. I probably do also want to check to make sure that there is traffic flowing through the box, maybe comparing before and after.

OK, so how do I automate all that. I can do all this with ansible. As part of the OS upgrade process it does the checks. I think it's hard to make these checks reusable, and it's really hard to make it generalizable for many different checks. In [Getting Network Device Operational Data from Ansible](https://blog.ipspace.net/2020/12/updated-ansible-parsing-content.html), Ivan Pepelnjak talks about how to write your own collection using Ansible, and this is a good idea, but there are much better ways of doing this systematically

What I've seen is that when you start creating a set of automated checks that collects data and does checking, it's hard to write generalized code that is easy to maintain over time. You mix in the collection with the check. Then you realize you need to deal with multiple OSes or a new version changes the output or something ugly like that. And your code gets messy.

What you want is a system that collects the data you might want, and puts that in a standard format. The checks are totally independent from the collection systems, making the checks much clearer and just focused on the problem at hand. The checks can be much easier to understand since they aren't deal with the format but just focused on the logic that you are trying to understand.

I would never want to do automated changes of the network without automated checks of the network.

## Validating the Change is Safe

We've been talking about how to verify that before you make and change and after that your network is safe. But what about checking to make sure that the change is a correct and safe ever before you actually apply it? I think there are two important ways to do this: simulation or verification. 

[Batfish](http://www.batfish.org) is software for checking configuration, and if I still had a network I'd be using it. It's a great system for making sure that the change you are proposing has the correct syntax and will do what you think it will do in the network. [The what, when, and how of network validation](https://www.intentionet.com/blog/the-what-when-and-how-of-network-validation/)

Another way is to run your vendor NOS in a virtual machine, such as vagrant or GNS3. This gives you greater fidelity because it's running the actually NOS code, but it's much more complicated and requires more compute resources. There are pro and cons to each and I don't want to dive too deep into that here; just know that you need to do something to verify that your change is correct.

Batfish (or simulation) is complementary to pre and post change checks. In fact, I think all three are required if you really want to be safe in your network.


# Is my network behaving as I expect it to

As mentioned above we want to know that the network is behaving as expected. If it isn't, then we are going to get surprised, possibly when an operator changes something, or a device fails, or cosmic rays hit a router. 

How do we confirm or deny what we think our networks are doing. We talked about how to write health checks that can be automated. That is a great place to start. But we want to add more. It's worth it to add a lot of testing that most of the time finds nothing, but every so often the testing finds something

What is a valid network? I think to answer depends on the context that you are in.

## Continuous Checking

Starting with the automated pre/post change story about, if you have a systematic way of collecting data and writing tests that are easy to understand, you can run these tests all the time. Wait, why would I want to run these all the time? Let's step aside and talk about [unit tests](https://en.wikipedia.org/wiki/Unit_testing) and automated testing done by software engineers. Whenever they make a change to code, they will have a bunch of test that run to make sure that nothing broke. Most of the time most of the tests don't find anything wrong with the code. But sometimes you find that your change broke something that you didn't expect, and that one time is worth all the other effort. It gives you confidence that you can make changes in your code.

Networks are different in that they are changing all the time, either from things fail, or from people making changes throughout the network. Which means it's a good idea to always be running tests to make sure that the network is working the way that you expect.

If you think of these pre/post change checks as health checks, rather than just making sure your change worked, you can run them all the time. This can give you confidence that the network is working as expected.

Thinking this was also will make your pre/post change checks be more robust. Rather than just running the tests that you assume you need, you can run all of them because we don't always understand the changes that we make and the network that we have.

## Validate Operational State

You can think of those pre/post change health checks, that you run all the time as validating operational state. When you start thinking this way, then you can think about what other checks you might want to run to validate operational state all the time. 

How do you want to be automatically testing your network all the time? Many network operators have felt the pain of OSPF or ISIS neighbors not coming up because MTUs were incorrect. So when not test all the time that your MTUs are what you expect? There are many things like this


## Validation against vendor or other best practices

Another good idea is to get the list of recommended settings from your NOS vendor and see if you have those things correct. At a minimum, they can help you realize all the things that are in your head that you need to get out.

## Describe the Network

The second tricky thing about network validation (after getting our assumptions our of our heads), is that it's actually hard to describe the network so that we can write good tests.


## Intent based networking

Isn't this Intent Based Networking? It might be, as usual with networking marketing terms, it changes over time. That's why I'm trying to dive into what we really need it to be. It's definitely not all of IBN. IBN focused on being able to describe what it calls intent and then be able to configure the network and check as appropriate.

IBN most pushed by A and in RFC ..., I don't think it does all these things, though I think it is trying to.

I'm focusing on the validation piece (obvi), but I want to dive in deeper than I think IBN talks about.
what is intent based networking
different types of failures

# Configuration Auditing and Validation

One thing I've heard proposed a lot is auditing mechanisms to make sure that your configuration on devices is what is expected. I generally hate this idea. I'm much more interested in understanding that the effect of the change is correct, rather than the change.

# Diving into the details of in-validness

What's the right level of checking for validation? Is it low level, is it high level?
just because something is broken, doesn't mean you know how to fix it?


I'm not talking about formal verification, such as in batfish. That's another very interesting and useful field, but for another conversation.




Just knowing what has failed or is in fault isn't enough. Just knowing things that are over threshold or out of compliance isn't enough. Is it important that interface or device or session is down



# Solutions
Okay, we've identified a bunch of questions to ask, and thought about how we'd like to be able to answer those questions. Is there any software to help me do that? A lot of this is actually hard without good tools.

Another small plug for [Suzieq](https://github.com/netenglabs/suzieq), which we've written to help with these type of questions. Suzieq continuously gathers operational state from the network and puts it in a vender-neutral normalized form. This makes it easy to then be able to write checks that we've been talking about. Suzieq also makes it easy to search for things like OS version. Suzieq has checks called asserts. These are still a little too complicated for people to easily add the kind of checks that we've been talking about, but it has the right architecture to make a great health check service. As an example of something cool, we already have a check (an assert) to see that MTUs on a link are the same. So it's not just checking for one MTU across the network, but can automatically take topology into account.

I already mentioned [Batfish](https://batfish.org) which you should check out if you have not already done so.

# Conclusions

Points I want to make

1. We want to know our network is operating the way we think it is.
1. The best way to do this is to come up with ways to be continuously testing the network. 
1. Do pre/post change checking.
1. Automate pre/post change checking.
1. Run these checks all the time to know your network is healthy.
1. Make sure your change is correct with something like Batfish or simulation.
1. Add even more correctness checks and run them all the time.
1. To have really great validation, you must have network metadata.
1. It's hard to know that your network is working correctly and the way that you expect, but that just means it's a place to innovate.

Great software engineering teams take testing very seriously. In networking, this is especially hard, and we haven't been trained that way. But we can get started.


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

