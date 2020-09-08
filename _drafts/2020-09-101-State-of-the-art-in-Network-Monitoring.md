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

FCAPS

ok, what do I want from telemetry. What does it mean

what are you trying to do?
	*  monitoring up/down and create alerts -- Fault
	* trend and graph? Performance
	* capacity planning?-- performance
	* how do you aggregate?

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

## products

I can't really capture all the products used, produced, or sold here. There are too many. I've tried to capture what I think are useful representative

### products for money

### cloud products


### open source


please use something really good here. I don't know if the commercial or cloud are good, but spend money (maybe in engineering time) and get a good telemetry systemTIG, TGP, SGPzabbixRRDtoolsdatadoghttps://en.wikipedia.org/wiki/Comparison_of_network_monitoring_systemshttps://softwareportal.com/open-source-network-monitoring-softwarewhere does nagios fithttps://hackernoon.com/monitor-your-infrastructure-with-tig-stack-b63971a15ccfhttps://pc.nanog.org/static/published/meetings/NANOG77/2057/20191029_Garros_Getting_Started_With_v1.pdfhttps://thwack.solarwinds.com/t5/Geek-Speak-Blogs/An-Overview-of-Network-Telemetry/ba-p/475601https://tools.ietf.org/id/draft-song-opsawg-ntf-02.html