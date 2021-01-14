---
layout: post
comments: true
author: Justin Pietsch
title: Get the Network Out of the Way
excerpt: Getting the network out of the way was an important concept for me to think about in my AWS career. Spending money on networking saves money overall, assuming you use that money to get the network out of the way.
description: Getting the Network Out of the Way is an important goal for a well run network and has been an important goal for me in my career. How can the network better get out of the way to enable the business around it to succeed?
---

Getting the network out of the way has been very important for me in my thinking about networks, and is an easy way to help talk about a bunch of very important concepts, decisions, and arguments. In this post I'll try to describe the concepts I'm talking about and illustrate with some of most important examples in my career.

**The more that the network is noticed the worse things are** for everyone. Often times, especially when the network is noticed, networking and network engineers are thought of negatively. Instead, if you think of it as a challenge it can help you focus on making a great network.

# How is the network in the way?

There are many ways that the network can be in the way of a business, several of which are especially important. The cost of the network can be prohibitive,  the network might not be able to keep up with what the business needs and  operational outages get in the way of what the business. This leads to issues of trust: does the business that the network supports trust the network and the network team? Is it reliable, including reliable in keeping up with business requirements?

If it takes you multiple years to plan and build out a new network design, even before you implement it, that seems like the network is in the way. What if you could do that in half the time, or 20%, would that enable new things for your business? Would it give better service to  your customers? Do outages shake the trust that your customers have of the network? How would you like to demonstrate that you are in-control of your network and that you are minimizing outages?

For a network engineer, when you first hear "get the network out of the way" you might feel discouraged that the network is though of negatively. But that's not it, the network is so important, but it's importance is in enabling other things and the less it's noticed the better it is. This is a grand challenge, to figure out how to design your network and your operations to meet these goals. This actually requires innovation and some serious engineering.
## The network costs too much

At the end of the day, **while cost is often the biggest conversation, it is not the most important.** Much more important is if networking slows down or hampers the business. In 2009 when I was at AWS, James Hamilton, AWS VP and Distinguished Engineer, pushed us to "Get the Network out of the way." What did he mean? James often starts investigations based on money. According to his calculations in https://perspectives.mvdirona.com/2010/09/overall-data-center-costs/, networking in a datacenter is about 8% of the total cost of the datacenter. This means that if you are optimizing your networking hardware costs and because of that you don't get full usage of your servers, you are wasting money, potentially a lot of money. For instance, if you have to buy more servers because they don't have enough bandwidth, and so they have wasted CPU cycles, you are wasting money. Also, if the network is unreliable, so that you can't make use of your compute and storage resources, you are wasting money.

**Spending money on networking saves money overall, assuming you use that money to get the network out of the way.** It turns out, the capital cost is not something we need to get out of the way. Instead, we need to be able to *spend* more money on networking so that you can optimize operations and get the network out of the way. However, it's rarely clear that spending more money means that the business will get more agility and better uptime. This is one of the challenges in network, to demonstrate that spending more money gives necessary value.

This doesn't mean you have to spend more money. Making the network "cheap enough" is important. In our case because we were moving to white box switches with merchant silicon and merchant optics, we were saving so much money. The new network was much cheaper while also easier to operate. However, we could have made it even cheaper, but that would have made it much harder to operate, especially as AWS exploded in popularity. We knew how we would add more capacity. We automated almost all of the configuration management and deployment. We had lots of redundancies so that we didn't have to worry about any one device failing, etc.

This idea helped us focus on how to think about networking. The network's job is to enable compute and storage in the datacenter, so if the network is in the way, *for any reason*, we should do something better. As mentioned above, if you've over-optimized your network capital spend so you are wasting compute resources, you are not making the right choice. Also, if your network is unreliable, then you are doing it wrong. If your network can't keep up with the changes that the business requires, then you have work to do.

Spending money, of course, isn't the only way to make the network better, and as mentioned above, there often isn't a correlation between spending money and getting a more effective network. It requires some deep thinking, better design and much better software.

It's all about operations and keeping up with the business. For AWS it was keeping up with unprecedented growth while creating better availability for our customers.
## Capacity

Don't run out of capacity. The worst outages I remember are from running out of capacity. Don't almost run out of capacity and then figure out how to grow. That network will be complicated and the equivalent of spaghetti code. Running out of capacity is often when you make your network overly complicated which makes it harder to operate later. How will you grow in a way that is systematic. If you have five sites and they grow at different rates, you still want them all to look the same and be able to recognize the same patterns.

I think one of the worse operational mistakes is recabling a networking. Let's say you have 4 core routes and 18 aggregation routes (the aggregation routers are in pairs) and you realize you need more aggregation routers than you can support and you need more bandwidth than you can support. So you decide you want to have 8 core routers. Almost certainly you will have to recable between layers.

To not run out of capacity you need two things. You need a good way of knowing how your traffic is growing and what is running out of capacity. The second is that you need to know how you will scale. 

## Engineering Tension

As in every engineering task, there are trade-offs to be made. There's lots of ways to interpret getting the network out of the way. I think for many networks this thinking can lead to over-engineering, and over-complexity because you want the network to take care of all the possible ways that the business needs. Even if I don’t need it now, the business will need to be able have more vlans or whatever. I don't think that's right. I think the point is to make the network as simple as possible. This often the best way to keep the network agile. The trick is figuring out what is necessary and what is extra complexity.

Getting the network out of the way doesn’t mean that you do whatever somebody asks for. That almost never gets anybody what they need. You have to figure out what the business needs and figure out how to make things as simple as possible to get that right. I think if you aren’t always pushing for simplicity you will make something that can’t keep up with the business. 

Don't optimize every piece of network hardware. You should not if you want a network that is operatable. Remember, it is a good idea to spend more money if it gives you simplicity that gets the network out of the way. Think about it like this, most of the software on the Web isn't built with the most efficient languages and most software engineers don't spend a lot of time optimizing code to use their hardware. There's a lot of  PHP, Ruby, Python, and even (gulp) Perl. This shows that in software people understand that the hardware costs are not nearly as important as being able to move more quickly and safely. **In networking that we are still hand optimizing everything as if the hardware and hardware cost was the most important factor.**

# Getting the Network out of the Way

I think it comes down to two things: a better design mentality and much better tools. In fact, with much better design tools I can automatically get the better design mentality. I've never seen tools the pushed that except the tools I built that were specialized for Amazon/AWS.
## Design

Another way of thinking about this is how do I design and build my network so that it is easy to operate the network? Operations is key here. What is required? First think about what we are talking about. We've decided that we should be willing to spend money to make the network better. We also want to make the network as simple as possible so that it can keep up with business changes, and that we want to optimize for operations, change, and availability. 

What does it mean to optimize operations? What are the day-to-day changes that will be necessary? Do you know how you will add more capacity to your network? What will be common changes in the network? Can they be automated? If not, can they follow the exact same roadmap? The more things are consistent the easier it is to understand the network and be able to make safe changes. 

It's worth it to spend time upfront to design networks that are easy to operate. One of the key pieces in the network is the software used to design, build, and operate the network. I think good network designs can't be done without software, and there is almost no software available to help with network design and simulation. Similarly, as you are designing your network, you need to be thinking about how to make it automatable. There are design tradeoffs which make networks easier to operate especially with software.

Another key idea is that you need to think about the network as a whole, not just a single device. The goal is not to have any individual device available, it is to have the whole network available. For that reason I have always hated dual-supervisor routers. Their goal is to make the one device more available, by adding extra cost and complexity. If instead your network can handle devices failing, you do not nee the dual-supervisor. Also, the complexity of keeping both supervisors up-to-date leads to more outages than not having them. You have to be careful that the solution to one problem isn't a much worse problem, and I think dual supervisors is a much worse problem.

I think the design phase is especially critical; it's where I focus my time and energy. The network needs to be designed to be automated and to operated well. It is extremely hard to write software to managed and operate an arbitrarily designed network. It's much easier if there are common patterns, and everything is standard. If everything is different and an exception, then the software gets much more complex.

An embarrassingly important lesson for me was the idea that **2 is the worst number in networks** and distributed systems. If you have two of something, you think you are redundant, but are you really? If one router goes down, you lose 50% of traffic, or worse, you are active/backup and you just hope that the backup will behave correctly as a backup. If you want to do maintenance on one of the routers, everything else has to be working perfectly. It's much better to use a number bigger than two. I think this means four or greater, but even three is better. If four routers is too expensive, buy smaller routers and get four. At AWS we moved to white box single ASIC routers, and went from layers of two to layers of 24 or 16. If a single device goes down and you have just two in a layer, than means somebody needs to get woken up. If you have three, then you don't. It means that systematically operating the network is possible. 

On the other hand, what do you do if you already have a network? What if it's not well designed, or just can't keep up with what's going on in the business now? What do you do if you aren't starting from scratch? You can still assume that your job is to get the network out of the way, and the best way to do that is to figure out how to make it more operatable. I think it starts with understanding your network.
## Automating and planning for operations

When James Hamilton challenged us to get the network out of the way, he also required us to look at commodity merchant silicon based networking. This made us re-examine our goals about the network. This effort led to the model of networking that is still being used in AWS. We started with merchant silicon, but we knew that the more important pieces were how to make this network easier to operate than our previous network, especially because we would be 10xing the number of routers in the network. We had to understand how we would add capacity, especially without recabling. By the way, if you are trying to minimize cabling (or almost any operational task), you are almost certainly "wasting some" hardware. We realized that because we were going to merchant silicon we were saving so much money that it was easy for us to hide the fact that we weren't using all available hardware so that we could make a network that was straightforward to add capacity to.

When you have a network as big as AWS, you really need all the deployments, in all the buildings, across all the countries to look the same. The third unit of capacity in every datacenter should look the same. I think this is true for any size network. It's so much easier to reason about things and get them right when everything is regular and you have common patterns
## Network understanding

You have to be able to understand your network so that you can know how to make it better and keep up with the business.  One thing I've noticed from network engineers is that there are more of them that have things memorized than other people I'm around. They seem to think it's required of people to remember important pieces of information. I have a terrible memory, which means that I need to make things much simpler. I need a small set of rules so that I can understand how the network works. Being ruthless about adding any new capability to a network means that it is much easier to understand, which means it is easier to troubleshoot, operate and change. 

The tradeoff, is that you have to provide all the necessary features, which might be more than you can keep in your head.This doesn't always work, or work in every case. I think there then needs to be better software to help you understand your network. How do you find things in your network? Do you know what things expect your network to be, but it isn't any more, either intentionally or not?

Software to help you understand and find things in your network is critical. A small plug for [Suzieq](https://github.com/netenglabs/suzieq), which we are building to help understand and find things in your network. I wish I'd had Suzieq or something like it at AWS. Instead I had to rely on my spotty memory. When I first went to Amazon, the network was complicated and I couldn't fit it into my head, so as I designed networks, they got simpler.
## Network Software

I don't think you can really get the network out of the way without much better software. I think you have to design with software tools to help you understand the tradeoffs, and simulation so that you can try out your assumptions, including things like trying out how you will grow your network. You need software to automate standard changes. You need software to keep the network regular. I don't think good examples of these things exist. and I don't know why.

# Conclusion

Innovate and get the network out of the way. The best way to do this might be to spend more money on hardware, and almost certainly on software to manage the network. Getting the network out of the way will probably save your company money in other ways by optimizing the rest of hardware.

Another description of this is way of thinking is in https://www.forbes.com/custom/2014/10/20/sdn-gets-the-network-out-of-the-way/ by Jeff Doyle. It's a marketing piece about an SDN controller and I'm not convinced SDN is required to get the network out of the way, but he shares a lot of the same motivation 

# Conversation
I'd like to turn this into a conversation, one way is to answer some questions I have.
1. Any good examples of the network in the way?
1. Any good examples about getting the network out of the way?
1. Any good examples about tradeoffs, and choosing the wrong (or the right) answer, that became clear over time?
1. What software capabilities would you like to help you get the network out of the way?
1. What software do you like that helps you get the network out of the way?