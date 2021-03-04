
---
layout: post
comments: true
author: Justin Pietsch
title: Exploratory Network Analysis (XNA)
excerpt: 
description: 
---

In data science there is this idea of Exploratory Data Analysis (EDA). You have a big bunch of data, what’s in it? How do you make sense of it? Where do you even start? In data analysis, this led to the S language, which led to S-Plus and the current favorite R. In the network I think we have much of the same problem and we don't have ways to systematically approach it. We have to custom build tools and analysis for each question we want to ask, but usually we either do it all manually, relying on humans to remember and process things, including using whiteboards. This seems wildly insufficient, especially in the 2020s.

EDA was defined by John W. Tukey in 1961. In his book Exploratory Data Analysis  in 1977 he felt that there is too much emphasis on statistical testing, and not enough on understanding the data. I think we are in a similar situation in which we are not really looking at our network to understand how it operates.

Networks are complicated, have a lot of data, and are hard to understand.  How do you explore your network now? you look at configs, maybe you look at traffic graphs. But the configs are complicated and hard to get your head around. And the traffic graphs don't represent how traffic flows or the protocols that ensure a functioning network. You look at a diagram, if it's up to date, at least you hope it is. 

I want to talk about this idea in general, and then talk about how we've implemented it in Suzieq . I'd love a discussion about the ideas in general, and any ideas on how to make Suzieq even better at eXploratory Network Analysis. We are accepting PRs :). 


One of the things that's different is that we have a bunch of data that isn't metrics based, so requires understanding of the concepts to make anything useful

Suzieq is focused on operational state in routers. We will be collecting interface counters and other metrics, but that's not where we've started. The reason is that there really aren't good ways to  monitoring and investigate operational data in networks.

what are some uses cases:
- where is this IP address or MAC address?
- How often (and when) does this MAC move?
- what is my topology, L1, L2, L3, per protocol
	- do my L3 map to my L1/L2?
- How does my routing policy work
- why does my network work this way?
- what does this policy do this? what does it mean for my overall network?
- If I make this change, what will change?
- Where did this MAC go? How many MACs do I have? Do I have any outliers that move a lot
- How many OSes do I have? How many device types?
- How have things changed over time?
	- what changed in the last 24 hours

Most networks (AFAIK) have reasonable interface monitoring and graphs of these metrics. However, even with counters, there are not good ways to explore your data. You can usually point to a specific interface to see it's utilization, but can you ask more general questions like:
- how many interfaces are ever over 50% utilized?
- what percentage of interfaces 99th percentile is less then 20% utilization?
- If we change our TCP algorithm, will we have less drops in the network? 
	- how will the distribution of drops change?
- If we change our buffers, will we have less drops in the network?
- when will we run out of capacity and where will that be?
- what is the correlation between utilization and drops and is it the same across the network?


We are using regular data science tools in Suzieq. We’ve built network engineering and operations applications on top of that, but you can use a Jupyter notebook and do all the analysis yourself. 

As an example, Suzieq gathers the routing table from every device and allows you to look up that data. You can also ask it to give you a summary of the table at any given time, such as the size of the table. But what if you want to know how fast your routing table is growing. Suzieq doesn’t currently answer that question, but it has the data so that you could answer it yourself in sw real different ways
How is this different than observability



Conversation

1. how do you explore, find, and understand your network today?
2. Are there things you'd like to investigate that are too hard right now?
3. Try out Suzieq (ok, not a conversation, but a conversation starting point.)
4. When you've tried out Suzieq, what would you like to help you explore, find, and understanding your network?

https://www.itl.nist.gov/div898/handbook/eda/section1/eda11.htm