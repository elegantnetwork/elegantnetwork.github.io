---
layout: post
comments: true
author: Justin Pietsch
title: Get the Network Out of the Way
excerpt: 
description: Getting the Network Out of the Way is an important goal for a well run network and has been an important goal for me in my career. How can the network better get out of the way to enable the business around it to succeed?
---

https://www.evernote.com/shard/s3/nl/221926/044e7304-0891-4b38-ab6b-c662659c6b47?title=lessons%20in%20moving%20to%20commodity%20networks%20in%20the%20datacenter

One of the things I learned at AWS Networking was the mindset that my most important goal was to get the network out of the way. This idea has been very important for me in my thinking about networks, and is an easy way to help talk about a bunch of very important concepts, decisions, and arguments. In this post I'll try to describe the concepts I'm talking about and illustrate with some of most important examples in my career.

**The more that the network is noticed the worse things are** for everyone. Often times, especially when the network is noticed, networking and network engineers are thought of negatively. Instead, if you think of it as a challenge it can help you focus on making a great network.

# How is the network in the way?

There are many ways that the network can be in the way of a business, several of which are especially important. The cost of the network can be prohibitive, the agility of changing the network to keep up with what the business needs, and the operational outages get in the way of what the business needs of the network. This leads to issues of trust: does the business that the network supports trust the network and the network team? Is it reliable, including reliable in keeping up with business requirements?

If it takes you multiple years to plan and build out a new network design, even before you implement it, that seems like the network is in the way. What if you could do that in half the time, or 20%, would that enable new things for your business? Would it give better service to  your customers? 

Do outages shake the trust that your customers have of the network? How would you like to demonstrate that you are in-control of your network and that you are minimizing outages?

## The network costs too much

At the end of the day, while cost is often the biggest conversation, it is not the most important. Much more important is if networking slows down or hampers the business. In 2009 when I was at AWS, James Hamilton was pushing to "Get the Network out of the way." What did he mean? James often starts investigations based on money. According to his calculations in https://perspectives.mvdirona.com/2010/09/overall-data-center-costs/, networking in a datacenter is about 8% of the total cost of the datacenter. The important part of that is if you are optimizing you networking hardware costs and because of that you don't get full usage of your servers, you are wasting money, potentially a lot of money. Spending money on networking saves money overall, assuming you use that money to get the network out of the way. It turns out, the capital cost is not something we need to get out of the way. Instead, we need to be able to *spend* more money on networking so that you can optimize operations and get the network out of the way. However, it's rarely clear that spending more money means that the business will get more agility and better uptime. This is one of the challenges in network, to demonstrate that spending more money gives necessary value.

This doesn't mean you have to spend more money. In our case because we were moving to white box switches with merchant silicon and merchant optics, we were saving so much money, the new network was much cheaper. However, we did not optimize that network for cost. We could have made it even cheaper, but that would have made it much harder to operate, especially at scale.

This helped us focus on how to think about networking. The network's job is to enable compute and storage in the datacenter, so if the network is in the way, *for any reason*, we should do something better. As mentioned above, if you've over-optimized your network capital spend so you are wasting compute resources, you are not making the right choice. Also, if your network is unreliable, then you are doing it wrong. If your network can't keep up with the changes that the business requires, then you have work to do.

Spending money, of course, isn't the only way to make the network better, and as mentioned above, there often isn't a correlation between spending money and getting a more effective network. It requires some deep thinking, better design and much better software.

Making the network "cheap enough" is important. If we had not made the network significantly cheaper by moving to whiteboxes, we would have had real problems at AWS. Our initial implementation reduced networking to 1/3 of the previous cost, and we never stopped making it cheaper. We could have made it even cheaper at the very beginning by trying to optimize the hardware even more carefully. However, I still believe the more important, much more important, outcome is that we came up with a network that was designed to deal with operational issues. We knew how we would add more capacity. We automated almost all of the configuration management and deployment. We had lots of redundancies so that we didn't have to worry about any one device failing, etc.


# so then how is the network in the way and what do we do?

it's all about operations and keeping up with the business. For AWS it was keeping up with unprecedented growth.


Another description of this is https://www.forbes.com/custom/2014/10/20/sdn-gets-the-network-out-of-the-way/ by Jeff Doyle. It's a marketing piece about an SDN controller and I'm not convinced SDN is required to get the network out of the way, but he shares a lot of the same motivation 
# The tension

As in every engineering task, there are trade-offs to be made. There's lots of ways to interpret getting the network out of the way. I think for many networks, that leads to over-design, and over-complexity because you want the network to take care of all the possible ways that the business needs. Even if I don’t need it now, the business will need to be able have vlans or whatever. I don't think that's right. I think the point is to make the network as simple as possible. This often the best way to keep the network agile. The trick is figuring out what is necessary and what is extra complexity.


Getting the network out of the way doesn’t mean that you do whatever somebody asks for. That almost never gets anybody what they need. You have to figure out what the business needs and figure out how to make things as simple as possible to get that right. I think if you aren’t always pushing for simplicity you will make something that can’t keep up with the business. 

When you think about it this way, you get less discouraged. When you first hear "get the network out of the way" you might feel discouraged that the network is important. But that's not it, the network is so important, but it's importance is in enabling other things and the less it's noticed the better it is. This is a grand challenge, to figure out how to design your network and your operations to meet these goals.

# Considerations

Another way of thinking about this is how do I design and build my network so that it is easy to operate the network? What is required? First think about what we are talking about. We've decided that we are willing to spend money to make the network better. We also want to make the network as simple as possible so that it can keep up with business changes, and that we want to optimize on operations. 

What does it mean to optimize operations? What are the day-to-day changes that will be necessary? Do you know how you will add more capacity to your network? Can you make it so that you don't have to do manual things like VLANs per customer segment, but instead can automate things.

## Design

I think the design phase is especially critical, it's where I focus my time and energy. The network needs to be designed to be automated and to operated well. It is extremely hard to write software to managed and operate an arbitrarily designed network. It's much easier if there are common patterns, 

One of the key important design ideas is that the focus of the design of the network must be on how to make it as operatable as possible. This includes how it can keep up with with the business, and as mentioned, not too complex that it gets even harder to operate.

It's worth it to spend time upfront to design networks that are easy to operate. One of the key pieces in the network is the software used to design, build, and operate the network. I think good network designs can't be done without software, and there is almost no software available to help with network design. Similarly, as you are designing your network, you need to be thinking about how to make it automatable. There are design tradeoffs which make networks easier to operate especially with software.

requires better software and better tools

Part of the design phase is iterating with what you have

design with operations in mind. Much better software.

An embarrassingly important lesson for me was the idea that 2 is the worst number in networks and distributed systems. If you have two of something, you think you are redundant, but are you really? If one router goes down, you lose 50% of traffic. If you want to do maintenance on one of the routers, everything else has to be working perfectly. It's much better to use a number bigger than two. I think this means four or greater, but even three is better. If four routers is too expensive, buy smaller routers and get four. At AWS we moved to white box single ASIC routers, and went from layers of two to layers of 24 or 16. If a single device goes down and you have just two in a layer, than means somebody needs to get woken up. If you have three, then you don't. It means that systematically operating the network is possible. 


systematic. regular. don't worry about optimizing hardware.

On the other hand, what do you do if you already have a network? What if it's not well designed, or just can't keep up with what's going on in the business now? What do you do if you aren't starting from scratch?

You can still assume that your job is to get the network out of the way, and the best way to do that is to figure out how to make it more operatable. I think it starts with understanding your network.

## Design Fallacies 

Similarly, you don't have to optimize every piece of network hardware. In fact, you should not if you want a network that is operatable. Remember, it might be okay to spend more money if it gives you simplicity that gets the network out of the way. Think about it like this, most of the software on the WWW isn't built with the most efficient languages and most software engineers don't spend a lot of time optimizing code to use their hardware.  There's a lot of  PHP, Ruby, Python, and even (gulp) Perl. This shows that in software people understand that he hardware costs are not nearly as important as being able to move more quickly and safely. **Why is it in network that we are still hand optimizing everything as if the hardware and hardware cost was the most important factor?**
## Capacity

Don't run out of capacity. The worst outages I remember are from running out of capacity. Don't almost run out of capacity and then figure out how to grow. That network will be complicated and the equivalent of spaghetti code. Running out of capacity is often when you make your network overly complicated which makes it harder to operate later. How will you grow in a way that is systematic. If you have five sites and they grow at different rates, you still want them all to look the same and be able to recognize the same patterns.

I think one of the worse operational mistakes is recabling a networking. Let's say you have 4 core routes and 18 aggregation routes (the aggregation routers are in pairs) and you realize you need more aggregation routers than you can support and you need more bandwidth than you can support. So you decide you want to have 8 core routers. Almost certainly you will have to recable between layers.


## Operations

**It's all about how easy it is to operate the network well**, including adding capacity, dealing with change, and competent dealing with failure. What do you do if you didn't do design right, or inherited a poorly fitting design. 
How

I think it comes down to two things: a better design mentality and much better tools. In fact, with much better design tools I can automatically get the better design mentality. I've never seen tools the pushed that except the tools I built that were specialized for Amazon/AWS.

When James Hamilton challenged us to get the network out of the way, he was  forcing us to look at commodity merchant silicon based networking. This made us re-examine our goal. This effort led to the model of networking that is still being used in AWS.  We started with merchant silicon, but we knew that the more important pieces were how to make this network easier to operate than our previous network, especially because we would be 10xing the number of routers in the network. We had to understand how we would add capacity, especially without recabling. By the way, if you are doing this, you are almost certainly wasting some hardware. We realized that because we were going to merchant silicon we were saving so much money that it was easy for us to hide the fact that we weren't using all available hardware so that we could make a network that was straightforward to add capacity to.

When you have a network as big as AWS, you really need all the deployments, in all the buildings, across all the countries to look the same. The third unit of capacity in every datacenter should look the same. I think this is true for any size network. It's so much easier to reason about things and get them right when everything is regular and you have common patterns

You need much better software to help you design your network. Other disciplines know how they will grow over time, why is it always a surprise in networking.

# Consequences

If every engineering task or decision is a tradeoff, what are the tradeoffs of getting the network out of the way? At least what are the consequences and requirements? I think it requires more up-front thinking. It requires better software to help with design and simulation.