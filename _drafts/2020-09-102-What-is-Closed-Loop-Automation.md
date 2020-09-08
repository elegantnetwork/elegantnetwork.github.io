---
layout: post
comments: true
author: Justin Pietsch
title: What is Closed Loop Network Automation
excerpt: 
description: 
---

and what do we really need?

I’ve been reading about network automation and I keep finding the term closed loop automation and want to dive into what it might mean, why I think what’s currently available is inadequate, what we might want and what we could do right now. The point is that you do the automation and then you check to make sure it did what you said. When something promises as much magic as closed loop automation seems to imply! I want to know how it actually works and the mechanisms it uses to get the magic done. 

Most of the systems I’ve seen that talk about closed loop automation are so unsophisticated that it doesn’t do me any good that they are closed loop because they can’t do what I need in the first place. Closing the loop on nothing is not that useful. 
. . 

So I want to explore what we might really want closed loop automation



So I wonder if there are other ways to get the same benefit. There are several problems that this might cover:
1. Describing topology in networking is hard.
2. Describing changes to network configuration is hard to get right. It's hard to define what you want and it's hard to make sure.
3. Validating network change is hard.

The problem is that network changes are risky and you want to make sure that they are applied safely and they do what you want

 As far as I understand, checking to make sure the network is working is he primary reason for IBN. Most seem to think that the way to do this is confirm that the change that was intended is what was performed. Often they check device confit to confirm

I think there’s a much better way to do that, and even much more useful way. If you can describe checks yo verify that your network is working correctly before and after the change that is much better than just checking g the config has changed

Suzieq can be this

I think network also, but I’ve never actually seen or used netq at all. 


Are there also ways to be able to describe the outcome?

Other reason I have a problem with current IBN systems is that they are too opaque and magical. For instance, just describing the outcome that you want is not nearly enough. The system needs to have a lot of metadata about the network to be able to do the promise of IBN. You need to be able to record all (or as many as possible) of the decisions and assumptions that the engineers have when they design a network

Naming schemes often demonstrate these decisions. For instance when you tell a person that those devices are spine boxes, that means something. But it means nothing to any of the software systems that need to understand this to truly understand hat the network is safe after (and before) an change.

Enginer still make many many decisions when creating a network which need to get recorded if we can ever hope to have truly great network control and automation

## What are the key ingredients of a closed loop system


## What could I do now?

## Some current examples
I haven't used any of these software systems. This is just my impression based on what I can find out about them. If I'm wrong, I'd love a better understanding. Please tell me, and then show me, how I am wrong.

My general bias is that none of these tools do any given part well enough to want them to do everything. So I don't want a closed loop systems based on one tool.

### Apstra
Apstra looks interesting, and they say a lot of the right things, but I feel like it's too closed of a system. They


https://www.blueplanet.com/resources/what-is-closed-loop-automation.html