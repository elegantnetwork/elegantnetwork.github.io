---
layout: post
comments: true
author: Dinesh G Dutt
title: Time in Suzieq
excerpt: In this post we discuss how Suzieq handles time
description: Time, time, time, see what's become of me
---
In this post, we discuss how Suzieq handles time.

Every record that is stored has a timestamp associated with it. Suzieq's poller pulls the data from the devices its configured to gather data from. So the time recorded is from the perspective of the poller. Specifically, the poller records the time at which the result of executing a command on the remote node is received by the poller. So, if the poller fires a command at time t0, and the reply data is received at time t0+7 seconds, the timestamp associated with the record is t0+7. The timestamp uses the UTC timezone.

Suzieq only saves a record if it has changed compared to what was received in the previous poll interval. If nothing has changed, the result of the current poll period is not saved. Thus, Suzieq saves the data the very first time data is received for a service (bgp, routes etc.). After that, it only saves the data if something does change from this previously received data. 

Suzieq takes specific care to ensure that the data saved is accurate and that the changes are valid. If the data returns certain fields that can change across every poll such as uptimes, or protocol state changes when a session is not up (transitioning between Active, Connect and Idle in case of BGP, for example), this data is normalized to avoid saving the data every poll period. Uptimes are converted into a bootup timestamp so that it doesn't change every poll interval. Protocol state is either up or down (Established or NotEstd in case of BGP). Next, if a node reboots, its uptime changes and this causes Suzieq to throw away the previously saved record for all services and save the data again for that node. Thus, the data is saved again for a node after a reboot. If Suzieq is saving counters of any sort, the type of such a service is marked as a counter and so data is saved every poll interval instead of trying to compare for changes.

When it comes to analyzing the data, Suzieq provides two different perspectives on using time. The first is what we call "snapshot", and the second is what we call "changes". A snapshot, as the name suggests, is the view of the resource *at* a particular time. By default, Suzieq assumes the *latest* snapshot. This is the info provided when the user runs any of the commands such as show (or any of the other verbs). Consider the timeline shown in figure 1. Consider two nodes n1 and n2 as being polled for BGP. t0 is when the polling began. Just before time t1, node n2 had a change in the BGP routes advertised which resulted in creating a new record for n2 at the next polling interval, t1. At a time just before t2, node n1 had a change in the BGP routes received which caused a new record creation for n1 at the next polling interval, t2. So, if the user runs *bgp show* command without specifying any time, the output shows the record for node n1 with a timestamp of t2 and for node n2, with a timestamp of t1. 

![Figure 1: sample timeline](/assets/images/2020-05-04/time-Fig1.png)

Alternately, the user can specify the time for the snapshot. This is done via the "end-time" option associated with each command. If the user specifis an end-time between t0 and t1, the records shown will be that of at time t0 for both n1 and n2. 

At time t3, the user can request that not just the latest, but all the records be shown as of time t3.  This is done by specifying view=all with an end-time of whenever you'd like the snapshot to be. With just view=all and no time specified, you'll see all the records for a node from t0 till t3. In other words, *bgp show view=all* at time t3 with the example of the timeline in figure 1 will show the two records each for n1 and n2. If the user specifies *bgp show view=all end-time=t2'* (where t2' is just before t2), then the user gets two records for n2 and a single record for n1. 

Additionally, the user can specify to see changes in the data between periods t1' and t2' where t1' is just before t1 and t2' is just before t2. This is done via the start-time option associated with all commands. For example, by specifying *bgp show start-time=t1' end-time=t2'*, the output shows just the single record for n2 which occurred at t1. 

To summarize:

- Output in Suzieq is either at a specified snapshot, or all changes in a time window
- By default, with no time specification, the output shows the latest snapshot
- By specifying an end-time with the command, the output shows the latest information as of the time specified in end-time
- By speciyfing view=all with the command, the output shows all the changes up to the time specified (or present without time)
- By specifying both start-time and end-time, the output shows all the changes between the start and end times
	
	
