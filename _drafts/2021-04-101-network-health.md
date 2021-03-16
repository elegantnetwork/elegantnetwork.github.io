---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: 
description: 
---
Do you ever ask "Is my network healthy?" I think it's an important question, but a hard and confusing question. What does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility?

I think it is your responsibility [to decide the network is not healthy before your customers do](https://elegantnetwork.github.io/posts/Who-decides-if-the-network-is-not-working/). Which means we have to figure out what it means to know that your network is healthy. What are we trying to understand when we ask that question?

Sometimes it's when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." Sometimes it's when your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?"

I think healthy doesn't just mean that there are no outages or no current customer complaints, though those are very important. I think it includes "Is the network trustworthy to do the things that we need to do it in?" That might be "I need to make a change, is the network healthy enough to do this change?"


There are times in which all your systems are telling you things are running fine, but it turns out you aren't asking the right questions. Not exactly related, but in 2006 I was responsible for load balancers at Amazon, and I met our VP of Infrastructure in the hall and he asked me if we were doing fine for capacity. I said yes because all our indications were that it was correct. Then I started thinking about it more, and went home and started looking. I realized that we had been incorrectly modeling capacity, and that we were already out of capacity. In other words, you will get this wrong and you need to be able to ask and explore your network so that you can discover new things that you didn't realize you needed to care about.

