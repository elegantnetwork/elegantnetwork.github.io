---
layout: post
comments: true
author: David Sinn, Dinesh Dutt
title: How to Not Cut Off The Branch You're Sitting On
excerpt: making changes to the network safely
description: How do you make changes to the network safely without losing access to the device(s) you're making changes on?
---

_This is a joint post with David Sinn, who's worked on some of the biggest networks in the world_

This past week saw a major outage of Facebook. In this post, we tackle the question of updating a fleet of devices without running into the problems that Facebook ran into, specifically of making an update that doesn’t cut you off from accessing the device to fix mistakes in the update.

![](/assets/images/2021-10-11-safe-changes.jpeg)

## What Happened?

The entire Facebook family of applications went offline. Facebook, Instagram, and WhatsApp were completely inaccessible across the world. To make matters worse, Facebook’s operations team lost all access to the devices which needed to be fixed. Things kept getting worse. They needed physical access to the machines, but the people who could had their access cutoff by the mistake.

## What Caused It?


A seemingly routine update to the backbone router configuration took out all access to the Facebook network from outside the company network. According to [Facebook’s description of the problem](https://engineering.fb.com/2021/10/05/networking-traffic/outage-details/):

“_This was the source of yesterday’s outage. During one of these routine maintenance jobs, a command was issued with the intention to assess the availability of global backbone capacity, which unintentionally took down all the connections in our backbone network, effectively disconnecting Facebook data centers globally._”

The internet was rife with speculations on the cause and on what could have been done to avoid the problem. Some blamed automation, others lack of testing, some speculated on the folly of using the data plane to run the control plane (in reality the management plane). 

Facebook's detailed update clarified a few more facts:

“Our systems are designed to audit commands like these to prevent mistakes like this, but a bug in that audit tool prevented it from properly stopping the command.”

So, this was in some sense an accident. It wasn’t that they didn’t test. Programming is fundamentally hard. Writing infallible tests is hard.

However, a fundamental problem was that with a single change resulted in cascading failures, and they lost complete remote access to the devices. This prevented them from being able to fix their mistake. What could they have done to prevent this from happening? 

## The World Before Automation

In the times before network automation took hold, vendors like Juniper provided what is called a config-commit-rollback model for making changes. You edit the configuration, type _commit confirmed_ <timeout> to allow the configuration to take effect. If the configuration doesn’t cut the operator’s connectivity to the device off after this, the operator has to type “commit” within the timeout period specified. If the operator doesn’t manage to type _commit_ within the specified time window, the OS automatically rolls the configuration back to the pre-change state. 

Not every device offered it, and many still do not today, but when Juniper introduced it, it was life changing. This is one of the reasons so many network operators swear by Juniper. 

Now, if you merely committed, without using the “confirm” option to “commit”, its possible that you could’ve cut yourself off because the command took effect immediately. In the case of Facebook (no, what they did was not manual, but presenting a what if to make sense of the issue), they say they issued a command to assess the availability of global backbone capacity. Maybe they didn't do a dry run of this command since it isn't the change itself; maybe they intended to, but that was what the audit didn't catch. We may never know the details of what happened.

## With Network Automation

With automation, this manual model of config, commit confirm, commit or rollback model gets tricky. You’ll need to use something like expect scripts to accomplish this task. Ansible, a popular network configuration push tool, supports it if the underlying device supports it. But not everybody uses Ansible, and certainly not the hyperscalers and their ilk. They have their own internal tools for this, and their tools have to support this model.

But even this model is flawed. In the case of this FB outage, the error happened after a brief period following the change, taking effect due to the delay in propagating updates that is typical in the backbone. Adding artificial timeouts isn't the solution, it only provides the illusion of one. 

## Network Redundancy To The Rescue

In networking, we’re taught to always have redundant, unconnected paths to avoid single points of failure. Thus, having redundant points of connectivity to a device is essential. If we can always ensure that there’s a redundant path that’s not also modified when we’re effecting changes, the problem may be easier to tame. 

How do network operators provide redundant connectivity? One classical answer is to provide a separate __out-of-band (OOB) network__. A lot of chatter revolved around how an OOB network would have either avoided or shortened the duration of the event. But OOB networks are not a panacea upon closer scrutiny.

## OOB Networks

An OOB network usually means a network that involves connecting all the devices via an ethernet port. This port acts like a NIC connected to the control plane, and does not use the packet forwarding ASICs that power the router/bridge. This is how many enterprises deploy their data center network. 

Facebook did say they have an OOB network, but it was out of service when the change occurred. In the case of the Facebook outage, their OOB network is most likely used in the data-centers which the backbone disruption functionally isolated, potentially rendering the ability to leverage the OOB network difficult at best and impossible at worst. 

But an OOB network is just another network. How do you know the OOB network is working, i.e. not just that the link is up, but that you can reach the device via the OOB network. A lot of OOB networks are just cobbled together using the previous generation's left-over production network equipment. And they are often treated as less important to fix than the production network, with far fewer eyes on their operational status. This leads to the OOB connection only being tested when you need to make a change, as everyone opens up a backup connection to a device before they make their change, right?. So if your OOB path is broken, you may not have the ability to easily fix it without disrupting the change you were intending to make.

When you have a few hundred devices, such an OOB network can be quite simple. It is usually built using old school layer 2 (L2) packet forwarding. Routing or layer 3 (L3) is rarely if ever used in these networks. However, as the number of devices start to get very large, a large L2 network becomes fragile. You can add routing to fix this, and now this network starts to look more like the inband network in its complexity. The hyperscalers mostly run a simple, pure routing inband network, without EVPN or any such complexity thrown in. Some do complicate it by throwing in traffic engineering, but thats probably just one of them.

Furthermore, with rare exception, the control-processors only have a single OOB ethernet port. This implies that we will lose connectivity to the device due to maintenance on the OOB network or just due to a natural failure. And the more devices you manage, the more you will see the natural failures and must react to them.

Thus, making the OOB network your primary path to manage your network devices comes with several negative trade-offs. For example, if the OOB connection is lost and your network management system can no longer monitor a production node, is that a P1 network down emergency because you are running blind? Of course, you can infer the state of the device based on the state of its neighbors, but any performance monitoring data (resource utilization, drops, routing state, etc.) will be opaque to you until you restore the connection. This leaves you with the questions of running blind or shifting the traffic around the node until the failure is resolved and at what priority do you restore the connection? The alternative that many pursue is to use the more robust and redundant inband network to also monitor the network. Which leads us back to the inability to verify that there is access to a specific device via a redundant path. Furthermore, most monitoring systems either run inband or OOB. Few run with a leg on both networks. This lack of monitoring both inband and OOB in a seamless, holistic manner does not provide the vaunted redundancy when it comes to making non-catastrophic changes. 

Many networks deploy a different kind of an OOB network called __console networks__. Partially this is also to focus on the least common denominator for networks with remote locations where traditionally you would have a dial-up modem connected to your remote office router or via cellular access on newer gear. Another reason it is also commonly deployed is that many network operating systems have certain functions that can only be done on the console due to a "security" decision to enforce a physical connection to the device before they will let you do the sensitive function. Facebook maybe using console networks to access their backbone devices.

Console networks also suffer from having a single points of failure, and handling the case in reaching a device farther than serial likes to run. This leads to deploying a whole terminal-server for just a few nodes, or getting serial extenders. Serial extenders introduce additional types of devices that network operators need to understand and support. And, just as with OOB networks, there is also the question of how to know the console network is working. A common approach is to have a process to connect to every device in the network on its console port at some regularity and then informing people when any of them fail.

So do we have three redundant networks now: the inband or the dataplane, the OOB and the console network? Often, there are only two, with the larger network operators relying on just the console network as the OOB network, while enterprises tend to not deploy console networks.

Back to the Facebook incident: a possible compounding factor to what happened is that Facebook doesn’t have many remote sites, and so they probably didn’t plan on an cellular access. Most modern terminal servers come with LTE or cellular access option. It would be very interesting to know if a cellular modem connection to a terminal-server would have allowed remote access in this case.

Creating entirely redundant paths can be hard, especially in the presence of cascading failures as this one was. A mistake in the routing update cut off all external communication to FB, which caused DNS to fail, which caused any redundant path to also vanish as a consequence. 

## Conclusion

Managing networks is incredibly hard, even for relatively small networks, but especially at scale. People who haven’t walked a mile in the shoes of a network engineer don’t fully understand the realities that a network engineer faces, whether it be the lack of robust change support from the devices, the ability to automate effectively, or just the quality of their monitoring and visibility tools. 

To maintain redundant paths of connectivity to access a device, an OOB network is a common practice in most enterprise networks. Console networks are a form of OOB network that satisfy remote access requirements more simply, and are used in some of the more sophisticated networks. Independent of the type of OOB network you have, a few best practices of having an OOB network include:

* Ensure that the OOB network is working properly by treating it like the production network
* Test the multiple paths to managing a device periodically
* Ensure that the absence of a working redundant path to a device stops deployment of any change
