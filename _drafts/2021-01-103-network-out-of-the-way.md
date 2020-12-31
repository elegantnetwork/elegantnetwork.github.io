---
layout: post
comments: true
author: Justin Pietsch
title: Get the Network Out of the Way
excerpt: 
description: Getting the Network out of the Way is an important goal for a well run network and has been an important goal for me in my career.
---

One of the things I learned at AWS was the mindset that my most important goal was to  get the network out of the way. This idea has been very important for me in my thinking about networks, and is an easy way to help talk about a bunch of very important concepts, decisions, and arguments. In this post I'll try to describe the concepts I'm talking about and illustrate with some of most important examples in my career.

The more that the network is noticed the worse things are. Often times, especially when the network is noticed, networking and network engineers are thought of negatively. Instead, if you think of it as a challenge it can help you focus on making a great network.
How is the network in the way?

There are many ways that the network can be in the way of a business, though there are several that are especially important.  The cost of the network can be prohibitive, the agility of changing the network to keep up with what the business needs, and the operational outages get in the way of what the business needs of the network. This leads to issues of trust: does the business that the network supports trust the network and the network team? Is it reliable, including reliable in keeping up with business requirements?

In 2009 when I was at AWS, James Hamilton was pushing to "Get the Network out of the way." What did James mean? James often starts with investigations based on money.  According to his calculations in https://perspectives.mvdirona.com/2010/09/overall-data-center-costs/ , networking in a datacenter is about 8% of the cost. The important part of that is if you are optimizing you networking hardware costs and because of that you don't get full usage of your servers, you are wasting money; potentially a lot of money. Spending money on networking saves money overall, assuming you use that money to get the network out of the way. It turns out, the capital cost is not something we need to get out of the way. Instead, we need to be able to *spend* more money on networking so that you can optimize operations and get the network out of the way. However, it's rarely clear that spending more money means that the business will get more agility and better uptime. This is one of the challenges in network, to demonstrate that spending more money gives necessary value.

This helped us focus on how to think about networking. Networking's job is to enable all the other things in the datacenter, so if the network is in the way, *for any reason*, we are doing things wrong. As mentioned above, if you've over-optimized your network capital spend so you are wasting compute resources, you've done it wrong. But also, if your network is unreliable, then you are doing it wrong. If your network can't keep up with the changes that the business requires, then you are doing it wrong.

Spending money, of course, isn't the only way to make the network better, and as mentioned above, there often isn't a correlation between spending money and getting a more effective network. It requires some deep thinking, better design and much better software.

Another description of this is https://www.forbes.com/custom/2014/10/20/sdn-gets-the-network-out-of-the-way/ by Jeff Doyle. It's a marketing piece about an SDN controller and I'm not convinced SDN is required to get the network out of the way, but he shares a lot of the same motivation 
The tension

As in every engineering task, there are trade-offs to be made.  There's lots of ways to interpret getting the network out of the way. I think for many networks, that leads to over-design, and over-complexity because you want the network to take care of all the possible ways that the business needs. Even if I don’t need it now, the business will need to be able have vlans or whatever. I don't think that's right. I think the point is to make the network as simple as possible. This often the best way to keep the network agile. The trick is figuring out what is necessary and what is extra complexity.

As a side note, one of the things that discourages me about our industry is that there doesn't seem to be great overlay technology for datacenters that is readily available. This is the only way that the cloud providers succeed, but as far as I can tell, what is available for the public doesn't scale very well and/or is very complicated. Getting rid of eVPN and VLANs in datacenters, while providing overlay networks for the separation requirements, makes the network simpler, by providing the right abstraction and separate layers more cleanly. This can keep the network focused on what is required, and the overlay is usually easier to automate and test.

Getting the network out of the way doesn’t mean that you do whatever somebody asks for. That almost never gets anybody what they want. You have to figure out what the business needs and figure out how to make things as simple as possible to get that right. I think if you aren’t always pushing for simplicity you will make something that can’t keep up with the business. 

When you think about it this way, you get less discouraged. When you first hear "get the network out of the way" you might feel discouraged that the network is important. But that's not it, the network is so important, but it's importance is in enabling other things and the less it's noticed the better it is. This is a grand challenge, to figure out how to design your network and your operations to meet these goals.
Considerations

Another way of thinking about this is how do I design and build my network so that it is easy to operate the network? What is required? First think about what we are talking about. We've decided that we are willing to spend money to make the network better. We also want to make the network as simple as possible so that it can keep up with business changes, and that we want to optimize on operations. 

What does it mean to optimize operations? What are the day-to-day changes that will be necessary? Do you know how you will add more capacity to your network? Can you make it so that you don't have to do manual things like VLANs per customer segment, but instead can automate things.

design with operations in mind. Much better software.

An embarrassingly important lesson for me was the idea that 2 is the worst number in networks and distributed systems. If you have two of something, you think you are redundant, but are you really? If you want to do maintenance on one of the routers, everything else has to be working perfectly. If one router goes down, you lose 50% of traffic. It's much better to use a number bigger than two. I think this means four or greater, but even three is better . If four routers is too expensive, buy smaller routers and get four. At AWS we moved to white box single ASIC switches, and went from layers of two to layers of 24 or 16. If a single device goes down and you have just two in a layer, than means somebody needs to get woken up. If you have three, then you don't. It means that systematically operating the network is possible. 


systematic. regular. don't worry about optimizing hardware.

Capacity

Don't run out of capacity and then figure out how to grow. That network will be complicated and the equivalent of spaghetti code. Running out of capacity is often when you make your network overly complicated which makes it harder to operate later. How will you grow in a way that is systematic. If you have five sites and they grow at different rates, you still want them all to look the same and be able to recognize the same patterns.

I think one of the worse operational mistakes is recabling a networking. Let's say you have 4 core routes and 18 aggregation routes (the aggreation routers are in pairs) and you realize you need more aggregation routers than you can support and you need more bandwidth than you can support. So you decide you want to have 8 core routers. Almost certainly you will have to recable between layers.


Design Fallacies 

Similarly, you don't have to optimize every piece of network hardware. In fact, you should not if you want a network that is operatable. Remember, it might be okay to spend more money if it gives you simplicity that gets the network out of the way. Think about it like this, most of the software on the WWW isn't built with the most efficient languages and most software engineers don't spend a lot of time optimizing code to use their hardware.  There's a lot of  PHP, Ruby, Python, and even (gulp) Perl. This shows that in software people understand that he hardware costs are not nearly as important as being able to move more quickly and safely. **Why is it in network that we are still hand optimizing everything as if the hardware and hardware cost was the most important factor?**


Design

I think the design phase is really critical.

requires better software and better tools

Part of the design phase is iterating with what you have

Operations

It's all about how easy it is to operate the network, including adding capacity, dealing with change, and competent dealing with failure. What do you do if you didn't do design right, or inherited a poorly fitting design. 
How

I think it comes down to two things: a better design mentality and much better tools. In fact, with much better design tools I can automatically get the better design mentality. I've never seen tools the pushed that except the tools I built that were specialized for Amazon/AWS.

When James Hamilton challenged us to get the network out of the way, he was also forcing us to look at commodity merchant silicon based networking. This made us re-examine our goal. This effort led to the model of networking that is still being used in AWS.  We started with merchant silicon, but we knew that the more important pieces were how to make this network easier to operate than our previous network. We had to understand how we would add capacity, especially without recabling. By-the-way, if you are doing this, you are almost certainly wasting some hardware. We realized that because we were going to merchant silicon we were saving so much money that it was easy for us to hide the fact that we weren't using all available hardware so that we could make a network that was straightforward to add capacity to.

When you have a network as big as AWS, you really need all the deployments, in all the buildings, across all the countries to look the same. The third unit of capacity in every datacenter should look the same. I think this is true for any size network. It's so much easier to reason about things and get them right when everything is regular and you have common patterns

You need much better software to help you design your network. Other disciplines know how they will grow over time, why is it always a surprise in networking.

Consequences

If every engineering task or decision is a tradeoff, what are the tradeoffs of getting the network out of the way? At least what are the consequences and requirements? I think it requires more up-front thinking. It requires better software to help with design and simulation.