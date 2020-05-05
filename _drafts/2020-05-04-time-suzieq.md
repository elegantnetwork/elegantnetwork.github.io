---
layout: post
comments: true
author: Dinesh G Dutt
title: Time in Suzieq
excerpt: In this post we discuss how Suzieq handles time
description: Time, time, time, see what's become of me
---
One of the powerful features of Suzieq is the ability to observe the state of the network at a time in the past, or to play back changes in state in a specified time window. To understand how this is accomplished, you need to understand how Suzieq treats time both in data capture and in data analysis. This post is an explanation of these concepts.

## Time during Data Capture
Every record that is stored has a timestamp associated with it. Suzieq's poller pulls the data from the devices its configured to gather data from. So the time recorded is from the perspective of the poller. Specifically, the poller records the time at which the result of executing a command on the remote node is received by the poller. So, if the poller fires a command at time t0, and the reply data is received at time t0+7 seconds, the timestamp associated with the record is t0+7. The timestamp uses the UTC timezone.

Suzieq only saves a record if it has changed compared to what was received in the previous poll interval. If nothing has changed, the result of the current poll period is not saved. Thus, Suzieq saves the data the very first time data is received for a service (bgp, routes etc.). After that, it only saves the data if something does change from this previously received data. 

Suzieq takes specific care to ensure that the data is consistently saved in the face of valid changes. Consider the case where the data returns certain fields that can change across every poll. Examples of such fields include uptimes and protocol state changes when a session is not up (transitioning between Active, Connect and Idle in case of BGP, for example). Without handling these cases, Suzieq would end up saving data every poll interval because the uptime or the failed protocol state is constantly changing. Uptimes are converted into a timestamp so that it doesn't change every poll interval. Protocol state is stored as either up or down (Established or NotEstd in case of BGP), ignoring various states of down. Next, if a node reboots, its uptime changes. The poller detects this change and throws away the in-memory, previously saved record for all services associated with the node. This in turn causes the poller to save all the data again for that node. The last part of handling changing data is w.r.t counters. If Suzieq is saving counters of any sort, say interface counters, the schema associated with the service marks the service as a counter. For 'counter' services, data is saved every poll interval instead of trying to check if any change has occurred.

## Time during Data Analysis
When it comes to analyzing the data, Suzieq provides two different perspectives on using time. The first is what we call **snapshot**, and the second is what we call **changes**. A snapshot, as the name suggests, is the view of the resource **at** a particular time. By default, Suzieq assumes the *latest* snapshot. This is the info provided when the user runs any of the commands such as show (or any of the other verbs such as summarize). Consider the timeline shown in figure 1. 

|![](/assets/images/2020-05-04/time-Fig1.png)
|:--:|
| Figure 1: Sample Timeline to Illustrate Time in Suzieq Analysis |

Consider two nodes n1 and n2 as being polled for BGP. t0 is when the polling began. Just before time t1, node n2 had a change in the BGP routes advertised which resulted in creating a new record for n2 at the next polling interval, t1. At a time just before t2, node n1 had a change in the BGP routes received which caused a new record creation for n1 at the next polling interval, t2. So, if the user runs *bgp show* command at time t3, without specifying any time, the output shows the record for node n1 with a timestamp of t2 and for node n2, with a timestamp of t1. 

Alternately, the user can specify the time for the snapshot. This is done via the **end-time** option associated with each command. If the user specifis an end-time between t0 and t1, the records shown will be that of at time t0 for both n1 and n2. If the end-time specified is between t1 and t2, the records shown will be that of t1 for n2 and t0 for n1.

The user can request Suzieq to show all the changes i.e. records, not just the latest.  This is done by specifying **view=all**.  Thus, at time t3 with just *view=all* and no time specified, Suzieq will return all the records for a node from t0 till t3. In other words, *bgp show view=all* at time t3 with the example of the timeline in figure 1 will show the two records each for n1 and n2. If the user specifies *bgp show view=all end-time=t2'* (where t2' is just before t2), then the user gets two records for n2 and a single record for n1 (because t2' < t2 and n1's change was captured as having happened at t2). 

Additionally, the user can request to see only the changes between periods t1' and t2' where t1' is just before t1 and t2' is just before t2. This is done via the **start-time** option associated with all commands. For example, by specifying *bgp show start-time=t1' end-time=t2'*, the output shows just the single record for n2 which occurred at t1. Since no change occurred on node n1 between t1' and t2', no records for n1 are shown. 

To summarize:

- Suzieq can either display only the values as of a particular time (called snapshot), or all the records in a time window (called changes).
- By default, with no time specification, the output shows the latest snapshot
- By specifying an end-time with the command, the output shows the snapshot as of the time specified in end-time
- By speciyfing view=all with the command, the output shows all the changes up to the time specified (or current time without any end-time specification)
- By specifying both start-time and end-time, the output shows all the changes between the start and end times


## Examples of Using Time in Suzieq

Given the explanations, lets' look at the outputs with each of the time options. We use the suzieq-demo docker container for all the commands. You can run this with the command: `docker run -it ddutt/suzieq-demo`. Inside the container, type `suzieq-cli` to get into the CLI.

Lets start with a listing of the interfaces in the ibgp-evpn namespace. The output of `interface show namespace=ibgp-evpn hostname=leaf02` looks as shown in Figure 2. The output in this figure and the remaining figures have been snipped to focus on the fields that matter.

|![](/assets/images/2020-05-04/time-Fig2.png)
|:--:|
| Figure 2: Interface Listing |
	
Now, lets use view=all to see if there are any interfaces with additional records. `interface show namespace=ibgp-evpn hostname=leaf02 view=all` results in the display shown in Figure 3. Since no time specification was provided, it lists all records upto the present time.

|![](/assets/images/2020-05-04/time-Fig3.png)
|:--:|
| Figure 3: Interface Listing, View All Records |

As highlighted in the figure, interface swp1 shows three records, of a transition down and then up. Lets see if we can provide a snapshot time and reduce the number of records displayed. Specifying an end-time of '2020-04-29 11:48:00' results in the display shown in Figure 4. We see that the interface shows up as being down, because that was the final state of the interface as of time 11:48:00.

|![](/assets/images/2020-05-04/time-Fig4.png)
|:--:|
| Figure 4: Interface Listing, Snapshot with Specific Time |

For the same time window, using view=all produces the output in Fig 5. The two highlighted lines indicate that the final transition from down to up is not shown in the output as it falls outside the specified time window. 

|![](/assets/images/2020-05-04/time-Fig5.png)
|:--:|
| Figure 5: Interface Listing, View All Records Upto Snapshot Time |

Next, to only see the records that changed in a time window, use both start-time and end-time. For the same command, the command `interface show namespace=ibgp-evpn hostname=leaf02 start-time='2020-04-29 11:47:00' end-time='2020-04-29 11:48:00' produces the output shown in Figure 5.

|![](/assets/images/2020-05-04/time-Fig6.png)
|:--:|
| Figure 6: Interface Listing, View Changes in Time Window |

And the final picture is to increase the time window and see all the transitions associated with interface swp1.

|![](/assets/images/2020-05-04/time-Fig7.png)
|:--:|
| Figure 7: Interface Listing, View Chages in Longer Time Window |



