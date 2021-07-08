---
layout: post
comments: true
author: Justin Pietsch
title: Some ideas on what network design tools might look like
excerpt: 
description: 
---

I've written about needing network design tools and needing better ways to understand networks. Let me try to describe a little more concretely where I'm coming from and what I mean.


I've been pretty abstract so let me describe where I'm coming from. In 2009, we started trying to figure out how to use commodity ASICs in pizza boxes for our aggregation network. The devices we had were based on Broadcom Scorpion, a 10G x 48 device. With a three tier Clos, that's 2880 devices. I knew that in the future we'd have 64 port devices, which is a max of 5120. The Network OS that we had only had OSPF. Do you know how to get OSPF to work across 5K devices? Can it be done? How? I certainly didn't know how. As we worked it out on a whiteboard, I needed to see 
