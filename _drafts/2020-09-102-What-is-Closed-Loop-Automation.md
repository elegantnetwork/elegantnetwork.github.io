---
layout: post
comments: true
author: Justin Pietsch
title: What is Closed Loop Network Automation
excerpt: 
description: What is Closed Loop Network Automation, what do vendors think it is, and what do we need it to be?
---
I’ve been reading about network automation and I keep finding the term closed loop automation and want to dive into what it might mean, why I think what’s currently available is inadequate, what we might want and what we could do right now. **The point is that you do the automation and then you check to make sure it did what you said**. When something promises as much magic as closed loop automation seems to implement. I want to know how it actually works and the mechanisms it uses to get the magic done. 

Most of the systems I’ve seen that talk about closed loop automation are unsophisticated that it doesn’t do me any good that they are closed loop because they can’t do what I need in the first place. Closing the loop on nothing is not that useful. I have no actual experience running one of these closed loop automation systems, so let me know what I get wrong.

In networking, we have a barrage of marketing terms to try to get us to buy new things. I think that many times they are trying to describe a real problem, but usually the solution isn't what we actually need. **What do we need that could be represented in closed loop automation?** I want to explore what we might really want closed loop automation. Here's the thing, and I think about it a lot, there's a lot of hype in networking that confuses, stunts, and holds back the industry. I believe that networking needs a lot of innovation, but most of what we get isn't what we need. We need to work harder to figure out what we need.

The problem is that network changes are risky and you want to make sure that they are applied safely and they do what you want?

What I really want is to have unattended changes that are safe, have strong verification, and can automatically rollback and be safe. Safe, unattended, automatic, reliable.


So I wonder if there are other ways to get the same benefit. There are several problems that this might cover:
1. Describing topology in networking is hard.
2. Describing changes to network configuration is hard to get right. It's hard to define what you want and it's hard to make sure.
3. Deploying change is hard, and often times it requires a workflow.
4. We don't have sufficient systems that can really automate unattended changes in the network.
5. Validating network change is hard.


What do we want?
* a network that is automatable. This is actually hard and deserves a whole article. If you want truly hands off automatic config management and deployment, then you have to design a network in which that is safe proposition.
* all relevant metadata needs to be recorded so that automatic system can do it's work.
* I want to be able to describe my topology, describe my assumptions about my network, and describe changes. When a network engineer makes a change, he usually has a lot of assumptions about the current state of the network, and how the change being made will change the network. We need to get all that described.
* fast and safe deployments, with verification and rollback.
* unattended changes.
* pre-change checking and validation.

I have envisioned a weekly cycle sort of CI/CD, in which changes go into a pipeline that automatically runs in through a bunch of simulation and pre-checking. Assuming you have a big network, it's split into 7 (or less) regions, and each region is deployed to on a given day of the week. So US-east cost is always Tuesday, and no matter when you put a change into a the pipeline it's always the same.


 As far as I understand, checking to make sure the network is working is he primary reason for IBN. Most seem to think that the way to do this is confirm that the change that was intended is what was performed. Often they check device confit to confirm

I think there’s a much better way to do that, and even much more useful way. If you can describe checks yo verify that your network is working correctly before and after the change that is much better than just checking g the config has changed

Suzieq can be this



Are there also ways to be able to describe the outcome?

Other reason I have a problem with current IBN systems is that they are too opaque and magical. For instance, just describing the outcome that you want is not nearly enough. The system needs to have a lot of metadata about the network to be able to do the promise of IBN. You need to be able to record all (or as many as possible) of the decisions and assumptions that the engineers have when they design a network

Naming schemes often demonstrate these decisions. For instance when you tell a person that those devices are spine boxes, that means something. But it means nothing to any of the software systems that need to understand this to truly understand hat the network is safe after (and before) an change.

Engineer still make many many decisions when creating a network which need to get recorded if we can ever hope to have truly great network control and automation

## What are the key ingredients of a closed loop system


## What could I do now?


### pre-change validation
You have to know your network is safe.

I think batfish for pre-change validation could be really cool

Dashboards or automatic checks

## Some current examples
I haven't used any of these software systems. This is just my impression based on what I can find out about them. If I'm wrong, I'd love a better understanding. Please tell me, and then show me, how I am wrong.

My general bias is that none of these tools do any given part well enough to want them to do everything. So I don't want a closed loop systems based on one tool.

### Apstra
Apstra looks interesting, and they say a lot of the right things, but I feel like it's too closed of a system. They


https://www.blueplanet.com/resources/what-is-closed-loop-automation.html