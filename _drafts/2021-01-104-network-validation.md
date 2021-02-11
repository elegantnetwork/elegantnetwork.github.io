---
layout: post
comments: true
author: Justin Pietsch
title: Network Validation
excerpt: Is my network is behaving as I think it should be?
description: How do you validate that your network is working correctly? What does that mean, how can we make it useful? How can we test our networks to know that they are correct?
---
What does Network Validation does mean practically? Of course, there's no official description of what it means, but we can talk about what we would like it to mean so that it can be useful. What we are trying to get to is a network that is trustworthy. If the business or organization using the network can't know if the network works correctly and that if it's failing the networking operations team knows it and will debug it quickly they can't trust the network. This gets in the way of the organizations ability to do it's primary goal. 

Many networks assume that they are working correctly if there aren't too many devices down and no customer is complaining. That's a reactive position and as important as networking is to businesses, not where we want to be to [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). For the most part, network monitoring focuses on faults. Just because a device or interface failed doesn't mean the network is not healthy or valid. **We want to raise the level of engineering.** Knowing faults does not give you enough information about if your network is functioning correctly.

When I've been on-call, my least favorite phone calls or tickets are along the lines of "my application isn't working, is the network working?" This is extremely hard to answer well. To answer this I think requires different ways of thinking about networks and how to validate that a network is working correctly.

What we care about is **"Is my network is behaving as I think it should be?"** including when changes and failures occur. We want to minimize surprises. **Network surprises are too be avoided, at almost any cost.** They can come up when a device fails and the network doesn't adapt correctly, or when you add capacity and it doesn't add as much capacity as you thought, or you make a change and it breaks the network. Of course, that requires that I have an idea of how my network should be behaving and that I have a way to check that. 

Lets examine some questions, what they mean to the network, and if we can answer them. Some of these are hard to answer, but that doesn't mean we should try. I'd bet most people want to implicitly know the answer to these questions, but can't. Some questions I ask to understand if my network is valid:

- Can I safely make a change?
- This change I just made, is it correct?
- The change that I'm proposing, do I have the configuration correct?
- Is my network working as expected?
- Is my network healthy?
- Do I understand my network?
- Do I have violations from best practices?

Just being able to ask and answer these isn't enough. **We need a way to systematically ask and answer these questions.** If you make it systematic then you can rely on those answers. If you are doing it ad-hock, then you can keep getting surprised. Especially as you make changes automatically, network automation means applying changes more quickly, then you really want to have systematic checking that your network is what you want.

One of the trickiest thing about this problem area is that we as network operators have a lot of assumptions in our heads about how the network is supposed to be working and how it is working. We need to get this assumptions out of our heads and into software that can confirm or deny your assumptions. You can think of network validation as ways to be testing your network to see if it is operating as expected. What are the kinds of tests you think you need to understand your network?

## It's not just about faults

As mentioned above, most network monitoring relies on fault monitoring: just knowing if a device or interfaces are down. This is insufficient, there are many other things that can go wrong. Also, it's often not detailed enough. You might know that you have a problem in your network, but where do you start? Why did a bunch of hosts lose connectivity? Did something change recently? You will have to figure out all the answers to these questions yourself; it would be better if you had a system that 

# Change Validation

So where do we start? Let's start when where I think most network engineers start: "When I make this change in my network, did the change work?" Actually, I think the better question is: **"When I make this change in my network, is my network still safe and is it operating the way that I intended?"**

## Post-Change Validation

In the most naïve case post-change validation is done after performing the change and by waiting for somebody to report that something is broken. You hope you have monitoring to show you that something is broken, but maybe it requires a customer to complain. I think we can safely say that is wildly inadequate: we want to be more proactive than that. The next step might be after the change to check traffic levels, or manually look to make sure that the configuration is applied correctly. This is no longer negligent, but we can do better. If checking it's manual, you might forget to completely check everything, and different engineers will check different things to make sure it's correct. Also, just because you check something doesn't mean you check correctly. We want automated checks of the network.

There's multiple ways to validate a change. You can check that it had the effect that you expected or you can check that the network is the way that you think it should be.  I think **it's more robust to check that the networks is operating as expected rather than doing something like making sure the config is correct**. If you think about the purpose of your change, what you are trying to accomplish, and then you can check for that, you have a better understanding at the end that the change worked correctly. Rather than checking that the configuration you typed has been applied, you check that the effect of the change is happening.

So what do I do if my post-change validation fails? The best answer is to roll back your change. You want to get the network back to valid, understood, state as soon as possible, and you don't understand why your validation failed; it's not what you expected.

## Pre-Change Validation

If I have post-change validation, then **I also want pre-change validation.** Changes fail because the network wasn't the way the operator expected it to be when they made a change. For instance, if you are upgrading OSes, and have two devices in a pair, but one of them has some interfaces down, or BGP sessions down, then it won't have as much capacity as expected.

I'm unsure if many people do pre-change checking, but it seems as important as post-change checking, and as automatable. If you don't have a system to do that automatically, then it's harder. Again, you can use ansible or equivalent, but it's better to separate out the collection and normalization from the checking.

You want to know that you are safe to make a change before you make the change. Worst case scenario is that the network wasn't in the state you expected, you make a change, and something breaks. If the change was fine, but the network was broken before you made the change, then you could have prevented that outages.

## Automating Pre/Post Change Validation

We know we want a way to write automated checking before and after a change. This mostly likely requires getting data from devices. If the data is metrics, especially interface metrics than you probably can get that data from a monitoring system. But usually when you want to verify a change you need other data.

Let's go through a simple example: I need to upgrade all the devices in my network. This is the first time and I'm unsure of my process, so I'm going to go device by device. As part of the change I upload the latest software, shift traffic away from this device, then reboot. When it reboots I want to make sure that it's working correctly. What should I check? Some good indicators are if my protocol neighbors are up and exchanging routes. I can check to see if all the BGP neighbors that are configured to be up are up, all the OSPF or ISIS neighbors configured are up, and nothing is stuck. I probably do also want to check to make sure that there is traffic flowing through the box, maybe comparing before and after.

**How do I automate all that?** I can do all this with ansible. As part of the OS upgrade process it does the checks. I think it's hard to make these checks reusable, and it's really hard to make it generalizable for many different checks. In [Getting Network Device Operational Data from Ansible](https://blog.ipspace.net/2020/12/updated-ansible-parsing-content.html), Ivan Pepelnjak talks about how to write your own collection using Ansible, and this is a good idea, but there are much better ways of doing this systematically

What I've seen is that when you start creating a set of automated checks that collects data and does checking, it's hard to write generalized code that is easy to maintain over time. You mix in the collection with the check. Then you realize you need to deal with multiple OSes or a new version changes the output or something ugly like that. And your code gets messy.

What you want is a system that collects the data you might want, and puts that in a standard format. The checks are totally independent from the collection systems, making the checks much clearer and just focused on the problem at hand. The checks can be much easier to understand since they aren't deal with the format but just focused on the logic that you are trying to understand.

I would never want to do automated changes of the network without automated checks of the network.

## Validating the Change is Safe

We've been talking about how to verify that before you make and change and after that your network is safe. But what about checking to make sure that the change is a correct and safe ever before you actually apply it? I think there are two important ways to do this: simulation or verification. 

[Batfish](http://www.batfish.org) is **software for checking configuration**, and if I still had a network I'd be using it. It's a great system for making sure that the change you are proposing has the correct syntax and will do what you think it will do in the network. [The what, when, and how of network validation](https://www.intentionet.com/blog/the-what-when-and-how-of-network-validation/)

Another way is to run your vendor NOS in a virtual machine, such as vagrant or GNS3. This gives you greater fidelity because it's running the actually NOS code, but it's much more complicated and requires more compute resources. There are pro and cons to each and I don't want to dive too deep into that here; just know that you need to do something to verify that your change is correct.

Batfish (or simulation) is complementary to pre and post change checks. In fact, I think all three are required if you really want to be safe in your network.

# Is my Network Behaving as I Expect

As mentioned above we want to know that the network is behaving as expected. If it isn't, then we are going to get surprised, possibly when an operator changes something, a device fails, or cosmic rays hit a router. 

**How do we confirm or deny what we think our networks are doing?** We talked about how to write health checks that can be automated. That is a great place to start. But we want to add more. It's worth it to add a lot of testing that most of the time finds nothing, but every so often the testing finds something surprising.

What is a valid network? I think to answer depends on the context that you are in.

## Continuous Checking

Starting with the automated pre/post change story about, if you have a systematic way of collecting data and writing tests that are easy to understand, you can **run these tests all the time**. Wait, why would I want to run these all the time? Let's step aside and talk about [unit tests](https://en.wikipedia.org/wiki/Unit_testing) and automated testing done by software engineers. Whenever a change is made to code, a bunch of tests run to make sure that nothing broke, and there are no surprises.. Most of the time most of the tests don't find anything wrong with the code. But sometimes you find that your change broke something that you didn't expect, and that one time is worth all the other effort. It gives you confidence that you can make changes in your code.

Networks are different in that they are changing all the time, either from things failing, or from people making changes throughout the network. Which means it's a good idea to always be running tests to make sure that the network is working the way that you expect. If you think of these pre/post change checks as health checks, rather than just making sure your change worked, you can run them all the time. This can give you confidence that the network is working as expected.

Thinking this was also will make your pre/post change checks be more robust. Rather than just running the tests that you assume you need, you can run all of them because we don't always understand the changes that we make and the network that we have.

## Validate Operational State

You can think of those pre/post change health checks, that you run all the time as validating operational state. When you start thinking this way, then you can think about what other checks you might want to run to validate operational state all the time. 

**How do you want to be automatically testing** your network all the time? Many network operators have felt the pain of OSPF or ISIS neighbors not coming up because MTUs were incorrect. So when not test all the time that your MTUs are what you expect? There are many things like this.

You can also include traffic levels and device uptime as validation. In other words, one of your assumptions is that more than 85% traffic on any interfaces is bad, or a device (or interface) down is bad. You can write think of these as validations against your operating network. These are just more assumptions you have about your network.

## Describe the Network

The second tricky thing about network validation (after getting our assumptions our of our heads), is that **it's actually hard to describe the network so that we can write good tests**. As a network gets more complicated, the assumptions get more complicated. For instance, you might assume that host to leaf switch has an MTU of 9000, but leaf to spine is MTU of 9100. It's harder to right topology based assumptions.

The problem is when we are designing networks, we are making topology, device, protocol, etc. based assumptions, but we don't have any way of writing those down. Without having that metadata about the network, you can't really express and test for your assumptions thoroughly.

If I have four devices in a layer, do I wake somebody up if one device goes down? How can I describe that? If I add two more devices to that layer, I have to remember what I was assuming about how much capacity I can afford to lose, rather than writing it out. For instance, maybe I'm planning that I can afford one device down, as long as there are at least four devices in a layer. But what if I'm really thinking I can afford 1/4 of the devices down and I have expanded to eight? I *do not* want to get woken up if a single device is down, but I do want to know if two have failed.

As far as I can tell, there are no good ways of describing my assumptions about a network.

## Intent based networking

Isn't this Intent Based Networking? It might be a part of it, as usual with networking marketing terms, it changes depending on who you are talking to. That's why I'm trying to dive into what we really need it to be. It's definitely not all of IBN. IBN focused on being able to describe what it calls intent and then be able to configure the network and check as appropriate. It wants to focus on the what, not the how. IBN does have a significant piece about checking the network, but I'm not sure exactly how that works and it is product specific.

Maybe what I'm talking about is assumption driven networking. **It's critical to get our assumptions of what the network will do out of our heads** o that we can have tests to make sure those assumptions are correct (or not.) I think capturing intent is critical, but I also think capturing assumptions is even more critical. There are so many different assumptions we all make and it's important to get them out and understand what they are. That does include the high level intent: it's a very good idea to know what you are trying to accomplish with your change and with your network. 

# Configuration Auditing and Validation

One thing I've heard proposed often is an auditing mechanisms to make sure that your configuration on devices is what is expected. I generally hate this idea. **I'm much more interested in understanding that the effect of the change is correct**, rather than the configuration. First off, if you just validate configuration you are only making sure that the device applied what you wanted, not that the effect that you were actually trying to produce is correct. Checking the configuration doesn't make you think about what it is you are really trying to do, and making sure that it what is correct. I also don't like things that are based on config checking, especially diffs. It's not based on a model, so you are doing syntax checking (is the text the same) not semantic (does it mean the same thing.)

# Solutions

Okay, we've identified a bunch of questions to ask, and thought about how we'd like to be able to answer those questions. Is there any software to help me do that? A lot of this is actually hard without good tools.

I already mentioned [Batfish](https://batfish.org) which you should check out if you have not already done so. It's really powerful and can quickly simulate a network so that you can test out and validate that your network will behave as expected.

[pyATS](https://developer.cisco.com/docs/pyats/) is an interesting open source package from Cisco that helps abstract the interaction with NOSes to get operational state. It's multivendor, though the Cisco support is much more robust than any other. I have no experience with pyATS, though I do know that others do use it for checking to make sure changes worked as expected.

Another plug for [Suzieq](https://github.com/netenglabs/suzieq), which is our open source multivendor tool we've written to help with these kind of problems. Suzieq continuously gathers operational state from the network and puts it in a vender-neutral normalized form. This makes it easy to then be able to write checks that we've been talking about. Suzieq also makes it easy to search for things like OS version. Suzieq has checks called asserts. It has the right architecture to make a great health check service. As an example of something cool, we already have a check (an assert) to see that MTUs on a link are the same. So it's not just checking for one MTU across the network, but can automatically take topology into account.

Ansible and related tools can be used as the workflow engine for network validation, but they aren't very good at it. Ansible doesn't separate data gathering from checking, isn't vendor neutral, etc. But it's better than nothing.

If you are going to do it manually, especially pre/post change validation, at least have a checklist so that you don't forget and everybody on the team does the same thing. I don't think just diffing configs is enough, make sure that the effect that you were expecting has occurred.
# Conclusions

1. We want to know our network is operating the way we think it is.
1. The best way to do this is to come up with ways to be continuously testing the network. 
1. Do pre/post change checking.
1. Automate pre/post change checking. Don't do automated changes without this.
1. I think it's worth it to have a separate service/system to do these health checks. Separate from Ansible.
1. Run these checks all the time to know your network is healthy.
1. Make sure your change is correct with something like Batfish or simulation.
1. Add even more correctness checks and run them all the time.
1. To have really great validation, you must have network metadata.
1. We need a way of describing topology and network holistically so that we can get more complete validation.
1. It's hard to know that your network is working correctly and the way that you expect, but that just means it's a place to innovate.

Great software engineering teams take testing very seriously. In networking, this is especially hard, and we haven't been trained that way. But we can get started.

One of my main career goals is to figure out how to get more assumptions out of network engineers' heads and into systems that can instantiate them and validate their correctness. I think it's crucial. Too many operating parameters are locked inside people's heads, where  software can do no good.

# Suzieq
Try out [Suzieq](https://github.com/netenglabs/suzieq), our open source, multivendor tool for network understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.
# Conversation / Talk-Back

- What of these ways of validating do you think are important?
- What do you already have solutions for?
- What would you like solutions for?
- If this is too complex, let me know, ask me questions, and I'll see if I can have another go at it. I think these ideas are really important and it's likely I'm not telling the story well enough yet.


