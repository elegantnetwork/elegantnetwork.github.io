---
layout: post
comments: true
author: Justin Pietsch
title: Starting a network tools open source company
excerpt: 
description: Can Networking be a good Market for an open source company, or any tools company?
---

Networking is critical to everything, yet it doesn't get the same amount of attention or understanding as other parts of IT or even IT infrastructure. Dinesh and I don't think the problem is that we are missing protocols or better devices. The problem is that there aren't the tools necessary to operate networks well. Because of that, the processes and procedures for networking are remedial and don't elevate networking to where it needs to be.

We need to [Get the Network Out of the Way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). Networks are about 10% of the cost of your IT infrastructure, but have an outsized effect on the whole company. If you have an outage in the network, it breaks everything. If the network can't keep up with the changes necessary for the rest of the business, it slows everything You can't operate your network well without better tools than you currently have. You just can't. Our tools are not good enough for how important networking is 

Ok, you understand why we exist. What are we trying to build? We've started with [Suzieq](https://www.stardustsystems.net/suzieq/), our open source multi-vendor [network observability](https://elegantnetwork.github.io/posts/observability/) tool. It's currently focused on physical networks, but we will add support for cloud networking including Kubernetes and VPCs from the big cloud providers. We've already built a useful tool that has so much protentional as we get to spend more hours building software.

To build out Suzieq to be really really great, and then to use that momentum to build more tools that are critical to a well operated network means that we need to turn our concern into a business. So far Stardust Systems is just Dinesh and I, and we are self funded. We did a little bit of consulting last year, but felt like the consulting was getting in the way of building the software we want to build. I, Justin, feel like even without consulting we aren't building nearly fast enough. We have so many good ideas that can help so many networks I want to go faster, much faster than we are currently going. I want to build great tools, they feel so important to networking.

Ok, so we want to build a company and build a product what is that going to look like? That's where we are struggling. So I thought we'd document some of our struggle.  VCs talk about picking a large market being the most important part of the problem. Ok that’s cool. But we aren’t picking this market because it’s particularly big. We are picking it because it’s particularly important and needs to be a lot better. 

In the wider space of developer tools, databases, big data, etc. there has been a set of big changes in the last 5-10 years. One is that the [software engineers drive the sales process and so are marketed to directly](https://a16z.com/2020/07/29/growthsales-the-new-era-of-enterprise-go-to-market/). In the past, all IT spend was directed at the CIO or Director of IT. You had to show ROI and you had to have checklists of features so that they could compare. Now you are marketing directly to the engineers and then from that figuring out how to get the people with the money to spend. Slack, Snowflake, etc. 

Relatedly, though not the same, [some very big companies are starting with Open Source communities and then building products from that](https://a16z.com/2019/10/04/commercializing-open-source/). Even though they will monetize only a small amount of the overall community, you have gotten a lot of attention for the problem and you know who has the problem, so you have a lot of qualified leads. Jfrog, Hashicorp, Redis, etc.

Then, of course, there is SaaS. More and more of what you wanted to have on-prem you are getting directly as a service.

Ok, but we are in networking. The market is smaller, it is not as far along as the compute, dev tools, database market is, and it's not clear how many people want to operate their networks remotely.

We have Suzieq as an open source project. It's hard to measure exactly how many people are using it. What we can measure is 294 github stars, 2800+ docker container downloads, 100+ people in our slack. We can track github stars and you can see how it's growing.



<iframe width="800" height="600" src="https://star-history.t9t.io/#netenglabs/suzieq&Icinga/icinga2&NagiosEnterprises/nagioscore&cilium/cilium&hashicorp/vagrant&batfish/batfish"></iframe>

If you compare Suzieq to other networking projects, the slope is steeper. If you compare those two compute projects to networking, networking slopes are shockingly smaller. So what do we do? We probably can never grow the community to be huge to drive sales. 

## Business Models and why they matter

At the end of the day we need to have a way to attract customers to even know we exist and think we can solve there problems. We need a product that solves problems. We need to be able to charge enough to build a company that can build enough features to keep growing the number of people willing to pay us more money than it costs us to build the product. 

How we figure this out will effect what is open source vs what is proprietary. It affects when we have to get sales people, and what kind of sales people, and when. It affects how people will enough know about us. 

There are various descriptions of what is called the sales funnel. For this post, I'll choose four steps: Awareness, Interest, Decision, Action. It's called a funnel, because the top of the funnel is the biggest, but as you go down the funnel, less and less people make it until you get to a sale. One of the important points is that to get to sales, you have to have a lot of awareness. In compute, databases, etc. there are many examples of open source projects being the top of that funnel and driving sales.


Also, networking is just different. I can't find examples of networking companies that are successful that have moved to these new business models. As far as I can see, they seem to be top down driven by large sales team with a legacy sales model. But I could be wrong, I just can't find an example. If you look at the websites of many of the modern IT enterprise companies they are directed towards the engineers. If you look at networking or networking software, it's more towards the CIO or Director of IT. There's usually lots of talk about ROI.

https://www.openlife.cc/blogs/2014/september/selling-open-source-101-sales-funnel-and-its-variables


### Gitlab / Hashicorp model

Gitlab and Hashicorp have similar business models. They market towards engineers, but don't charge for the features that engineers want. Basically everything their core customers need is free. Then they come along on the back and charge the people with money for things that they need. Or as gitlab says, features for engineers are free, and then teams cost some more, and then enterprises cost the most. It doesn't mean that if you have a big company you pay and a little company you don't, it means that features needed for managing a team cost some money and features needed when you manage many teams cost more. Some of the features are things like authentication and reporting. 



### Lots of proprietary features.

An obvious model is to have basic features open source and then the most important features for network engineers are proprietary. This seems to be the way to make the most people upset. There are quite a few examples of open source companies starting out with having important features proprietary and then over time they move them to open source and most if not all engineer features are open source. I can't find an example right now in infrastructure enterprise that has a small open core and lots of proprietary features.

### Slack model

Slack is, of course, all proprietary. What's interesting about Slack is that a small number of engineers could just start playing with it for cheap and just go around the IT staff. So while not open source, they market towards the engineers and make it really easy to get started. Then when you have 10 different teams each using their own slack and you'd like to combine everything, that's when the company gets charged. Sounds good. 

### SaaS

The coolest kids are all building SaaS. They are usually using the bottom up growth model like slack. 

## What are some options

We have a bunch of ideas on how to make Suzieq better in [Network Validation](https://elegantnetwork.github.io/posts/network-validation/),[ Network Health](https://elegantnetwork.github.io/posts/network-health/), and other things, but we don't know how best to get those to customers. Which features should be in Open Source, which features should be proprietary? How do we balance the needs of the Open Source community while also building a viable business?


The model of features for engineers are free and enterprises pay sounds great.. I'd love to build a lot of great features for network engineers that are all free and then features like authentication and reporting and maybe high availability are what we charge for. Except, I'm not really sure what team features or enterprise features mean for something like Suzieq. And even more importantly, it doesn't look like Open Source in Networking can drive a community big enough to push through the Sales Funnel. But I'm not sure. 

Or maybe we have lots of proprietary features. I can't find any examples in the compute space that is doing that right now, but because we have a different market

## Why are these people interesting?

In case you don't know who we are, [Dinesh Dutt](https://www.linkedin.com/in/ddutt/) and I ([Justin Pietsch](https://www.linkedin.com/in/justinpietsch/)) have been in networking for decades. Dinesh was at Cisco for a long while, eventually as a Fellow working on Catalyst 6500, Nexus 7000, and VXLAN, and then was Chief Scientist at Cumulus Networks. I was a Principal Network Engineer at Amazon for 17 years. We bring large vendor and large operator experience together. We actually have good experience with knowing what makes network operations better and we have experience building systems, software, and very large networks. We know how to solve problems. Getting to customers, getting people to pay, building a business,  we have a lot (a lot) less experience.

## Summary

We want to solve problems in the networking space, and there are many. It's a smaller market than compute, though not tiny. We'd like to use Open Source to help us build a company and we already have an Open Source project that we want to build into a product.

Networking Needs better software from better software tools companies.

## Conversation

1. Any good ideas on how to build a good network software tools company?