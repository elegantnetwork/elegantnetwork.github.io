---
layout: post
comments: true
author: Justin Pietsch
title: State of the Art in Network Monitoring
excerpt: 
description: There is a lot that makes up Network Monitoring
---


What is Network Monitoring and What do we need it to be?

if observability is better than monitoring, shouldn't we at least know what monitoring does for us?

People talk about Network Monitoring, but I don't think we all agree on what it means, we have a bunch of assumptions but aren't clear. I know that for me, I'm not even clear about my assumptions my self.

I want to start with [FCAPS](https://en.wikipedia.org/wiki/FCAPS.), which the ISO model for network monitoring and management. I think this is much like the ISO 7 layer model of the network: nobody actually does networking based actually on that model, but it's in the background of everybody's thinking. 


ok, what do I want from telemetry. What does it mean

what are you trying to do?
*  monitoring up/down and create alerts -- Fault
* trend and graph? Performance
* capacity planning?-- performance
* how do you aggregate?
* understand / troubleshoot / explore

What are we trying to talk about? We are talking about measuring network devices so that we can understand them. Usually this refers to data that is needed to understand the dataplane, but also management issues like CPU utilization and power.

The current state of the industry is very confusing. 
There are lots of tools, both closed and open source.

I don't think things are categorized very well. What are we trying to do?I think most people start with people want to know that there devices or interfaces are up or down. Many people use Nagios or a fork of it for that, as far as I can tell. telemetry in networking usually meansWhat it is and what it isn'tI don't want to talk about all the little windows based systems

what's the role of openconfig and yang?what's the role of streaming and so if SNMP causes so much trouble, why not use something else? the challenges in using not-SNMP
* with SNMP you have a normalized version of the data, you don't have to figure out what vendor A means and how that's different from vendor B

what do I do with the data?
*  create alerts 
* display per device status
* summarization of the network
* total problems and alerts
* network fault
* network availability
* capacity planning
* understanding performance like drops or other things
* is my network working?
* how hard is it to find information?
* predict issues
* let me explore my network

What to monitor
*  interfaces performance data
* drops
* any resource on the device that can fill up
* 

## Data sources

I wish vendors cared about monitoring and management as first class features of their products. I've never seen anybody take this seriously. Is there a product manager of data sources that has the power to beat up developers who only point counters in CLI?

And I wish us customers asked for these things and better evaluated them and vendors lost sales because of missing these features. The industry still had much to change.

It's also incredibly hard to know what you should be monitoring. Who can tell me the most important metrics I should be measuring on any given platform. Back to the idea of a product manager of data sources, tell me what are the best practices. 

For me, I always ask vendors if it's possible to count every way a packet can get dropped in a device. The hardware/ASIC engineers say "of course," and the software engineers give me a blank face. I can't figure out how to find that out for any or every platform

### Standardizing Data


### SNMP
the bests thing about SNMP is the standardized data, but that's insufficient, it doesn't cover enough of the data that we need these days. 

There is a lot of talk about how bad SNMP the protocol is and how it was designed in 1980s, but I don't care. It's easy access to important data that is standard


### CLI
vendors need to stop saying that they expose a counter if it's only in the CLI. On the other hand, with Suzieq we've taken them at their word and we are scraping CLI output. It's extremely frustrating from an architecture point of view, and lends to more bugs than structured output. 


### streaming telemetry, gNMI, etc
this is really confusing. Google says that we need streaming telemetry because of sharp edges and little inaccuracies. I guess, but that's not the biggest problem. If I had all the 

## Observability
Dinesh and I have written on observability, and it seems like nobody in network cares. I think that's because it's a confusing term, not because people don't need it.

One of the things we mean by observability is that you can ask any question you want.


When people usually think about monitoring, you monitor those things that you know that you need. With observability, you are more interested in getting all the data you can find, even if you don't know how you can use it. Then when you have a problem, you have more data to go through and ask questions of.

## products

I can't really capture all the products used, produced, or sold here. There are too many. I've tried to capture what I think are useful representative

### products for money

### cloud products


### open source


please use something really good here. I don't know if the commercial or cloud are good, but spend money (maybe in engineering time) and get a good telemetry systemTIG, TGP, SGP
zabbix
RRDtools
datadog
* <https://en.wikipedia.org/wiki/Comparison_of_network_monitoring_systems>
* <https://softwareportal.com/open-source-network-monitoring-software> where does Nagios fit
* <https://hackernoon.com/monitor-your-infrastructure-with-tig-stack-b63971a15ccf>
* <https://pc.nanog.org/static/published/meetings/NANOG77/2057/20191029_Garros_Getting_Started_With_v1.pdf>