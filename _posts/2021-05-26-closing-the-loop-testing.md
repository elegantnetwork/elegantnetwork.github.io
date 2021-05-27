---
layout: post
comments: true
author: Dinesh Dutt, Ratul Mahajan
title: Closing the Loop on Testing Network Changes
excerpt: closed loop automated testing is key to making safe network changes
description: Rigorous testing via automated closed-loop testing makes the change process highly resilient and catches problems as early as possible
---
Network changes, such as adding a rack, adding a VLAN or a BGP peer or upgrading the OS,  can easily cause an outage and materially impact your business.  Rigorous testing is key to minimizing the chances of change-induced outages. A central tenet of such testing is test automation—a program should do the testing, not (error-prone) humans. Test automation should target all stages of the change process. Prior to deployment, it should test that the change is correct and that the network is ready for it. Post deployment, it should test that  change was correctly deployed and had the intended impact. This **closed-loop test automation** makes the change process highly resilient and catches problems as early as possible. 

But writing the code to automate network testing can be quite complicated. For instance, if you were adding a new leaf, prior to deploying your change, you may want to test that the IP addresses on the new leaf do not overlap with existing ones. So, you may write a script that mines addresses from configurations and then checks for uniqueness. Similarly, post deployment, you may want to test that all spines have the new prefix. So, you may write a script to fetch and process  “show” data from all spines. Writing such scripts is fairly complex because you have to know the right commands, know how to extract the data (via textfsm or json queries), and so on. Sometimes, you have to combine information from multiple commands. And of course, this is different for every vendor, and in many cases, across different versions of the vendor. Your ability to automate network testing increases dramatically if you have tools that can take out much of the complexity. In this article, we cover two such open-source tools, Batfish and Suzieq, that help you easily automate closed-loop testing.

## The three stages of closed-loop test automation

Closed-loop network testing has three stages:

1. **Pre-approval testing**: Before you schedule the planned change for a maintenance window, test that it is correct.
2. **Deployment pre-testing**: Before you deploy the change, test that the network is in a state where the change is safe to make. It is not uncommon that the current device state has drifted from what is assumed to have been configured.
3. **Deployment post-testing**: Right after you deploy the change to the network, test that the change produced the intended behavior.


![The figure](/assets/images/2021-05-batfish-suzieq/fig-1.png)The figure above places these stages within a typical change workflow. pre-approval testing is done when you are designing and reviewing the change, and deployment pre- and post-testing is done during the maintenance window. 

Experienced network engineers will recognize that the three stages map to how manual change validation happens today. Peer-reviews fill in for pre-approval testing, and running show commands before and after the change fill in for deployment pre- and post-tests. We show how you can automate such validation. To be clear, we are not claiming that automatic testing can or should completely supplant human judgement. Rather, automation and humans can work together to make network changes more efficient and robust. This hybrid mode is akin to software, where developers use both a battery of automatic tests and peer reviews, and information produced by automatic tests greatly assists human reviewers.

## Tools of the trade

Let us first introduce the tools of the test automation trade that we’ll use. 

### Batfish

[Batfish](https://batfish,org) analyzes network configuration to validate that the behavior of individual devices and the network as a whole matches the user’s intent. It constructs a model of the devices (Cisco router, Arista router, Palo Alto firewall etc.) and uses the device configuration files to put the model in the state specified by the configuration. It can then use formal verification analysis to comprehensively answer questions such as whether a BGP session is up, if an IP address can communicate with another, and so on. Because of its reliance on only device config files, it is a simple yet powerful tool that doesn’t require you to run slow and expensive emulation tools such as GNS3, EVE-NG or Vagrant to validate network behavior.

### Suzieq

[Suzieq](https://www.stardustsystems.net/suzieq/) is the first open source multi-vendor network observability platform that gathers operational data about the network to answer complex questions about the health of the network. It enables both deep data collection and analysis. It can fetch all kinds of data from the network,including routing protocol state, interface state, network overlay state and packet forwarding table state. You can then easily perform sophisticated analysis of the collected data via either a GUI or CLI or programmatically. Using either a REST API or the native python interface, you can also write tests that assert that the network is behaving as you intended (e.g., is this interface up? is this route present?). 

You may be wondering: why do I need two tools? The short answer is that their testing capabilities are complementary. **Batfish reasons about network behavior based on configuration** (which may not have been deployed yet) and provides comprehensive guarantees for large sets of packets. **Suzieq reasons about actual network behavior based on operational data**. You use Batfish before pushing the change to the network for comprehensive analysis that the change is correct. This analysis assumes that the network is in a certain state at the time of deployment. Suzieq first helps validate those assumptions, and then also helps ensure that the change had the intended impact after it is deployed.

Testing focus aside, there are many similarities between the two tools that make them work well together. Both Batfish and Suzieq are multi-vendor and normalize information across vendors, so your tests are independent of the specific vendors in your network. Both have Python libraries that make it easy to build end-to-end workflows. And they both use the popular Pandas data analysis framework to present and analyze information. Pandas represents data using a DataFrame, a tabular data structure with rows and columns. You can find out the names of the columns in a DataFrame using the columns() method and use its powerful data analysis methods to inspect and analyze network behavior. A particularly useful method is “query” which filters rows/columns per user specifications. Some examples of using query are [here](https://suzieq.readthedocs.io/en/latest/pandas-query-examples/).

## Implementing closed loop testing

We now illustrate how Batfish and Suzieq combine to enable closed-loop network test automation. We will consider a trivial 2-leaf, 2-spine Clos topology, though the testing workflow and the example tests below apply equally to bigger, real-world networks. For brevity, however, we describe only a handful of tests in this post. A real test suite should have more tests that cover additional concerns. Developing a good test suite is an art form by itself, and we’ll cover this topic in a future post. The configuration fragments and tests used in this post are available in this [GitHub repo](https://github.com/netenglabs/closed-loop-testing-blog).

### Installing the software

Download the Docker container for Batfish and launch the service as follows:

```bash
docker pull batfish/batfish
docker run --name batfish -v batfish-data:/data -p 9997:9997 -p 9996:9996 batfish/batfish
```

Install pybatfish (the Python client for Batfish) and Suzieq in a Python virtual environment. You need at least Python version 3.7.1 to use these two packages.

```bash
pip install pybatfish
pip install suzieq
```

A quick introduction to the Pythonic interfaces of Batfish and Suzieq is useful now. The Python APIs of Batfish are documented [here](https://pybatfish.readthedocs.io/en/latest/index.html). Batfish provides a set of questions that return information about your network such as the properties of nodes, interfaces, BGP sessions, and routing tables. For example, to get the status of all BGP sessions, you would use the bgpSessionStatus question as follows:

```python
bfq.bgpSessionStatus().answer().frame()
```

The _.answer().frame()_ part transforms the information returned by the question into a DataFrame that you can inspect and test using Pandas APIs.

Suzieq’s Python interface is defined [here](https://suzieq.readthedocs.io/en/latest/developer/pythonAPI/). Suzieq organizes information in tables. For example, you can get the BGP table via:

```python
bgp_tbl = get_sqobject(‘bgp’)
```

Every table contains a set of functions that return a Pandas DataFrame. Two common functions are _get()_ and _aver()_ (because _assert_ is a Python keyword, Suzieq uses _aver_, an old synonym). Because Suzieq analyzes the operational state of the network, you must first gather this state by running the Suzieq poller for the devices of interest. [These instructions](https://suzieq.readthedocs.io/en/latest/poller/) will help you start the poller on your network.

You are now ready to implement the first stage of closed-loop testing.

### Pre-Approval testing

The goal of pre-approval testing is to ensure that the change is correct. We show how a Python program using Batfish, the tool we’ll use in this stage of testing, helps you catch errors in your change.  What exactly you test during pre-approval testing depends on the change. Let's continue with the example that we’re adding a leaf to our network. We have a new config that we created using the existing config from one of the other leaves. Or maybe you have a template and you’re using Ansible to generate the config for this new device. But due to a cut-and-paste or a coding error, one of the interface IP addresses is that of another device, and not the one you’re deploying. A string of numbers is easy to miss even with a peer review. Batfish can easily catch such errors. 

To start pre-approval testing with Batfish, put the configuration files of each router in a directory as described [here](https://pybatfish.readthedocs.io/en/latest/notebooks/interacting.html#Uploading-configurations). You can add leaf03’s config along with the modified spine configs to the configs subdirectory. 

You can write Python programs that use the Batfish API to automate your pre-approval testing.  ![Here](/assets/images/2021-05-batfish-suzieq/fig-2.png) The screenshot above shows an example of such a program.

This program initializes the network snapshot (with planned config modifications) in _init\_bf()_ and defines two tests. _test\_bgp\_status()_ uses the _bgpSessionStatus_ question to validate that all BGP sessions will be established after the change. _test\_all\_svi\_prefixes\_are\_on\_all\_leafs()_ verifies that the SVI prefixes will be reachable on all nodes. It uses the _interfaceProperties_ question to retrieve all SVI prefixes and verifies that each is reachable on all nodes. You retrieve the list of nodes using the _nodeProperties_ question 

**TIP**: The first time you use Batfish on your network, take a look at the output of _bfq.initIssues().answer().frame()_ to confirm that Batfish understands it well. The output of this command is also a good thing to check when a test fails because problems such as syntax errors are also reported in it.

Hopefully, you now see the power of automated testing with tools like Batfish and Suzieq. A few lines of code can validate complex end-to-end behaviors across your entire network. When you add another leaf or spine, you can run this test suite as is. In fact, you can run the same test suite across different vendors. Our example network uses Arista EOS. You won’t have to change even a single line if you used Cisco or Juniper or Cumulus or a mix.

You can even use pytest, the Python testing framework, to run the tests and make full use of an advanced testing framework. If any of the assertions fail, pytest will report them, and you can investigate the error, fix the config change, and rerun the test suite. 

Good testing tools also make it easy to debug test failures. How you do that depends on the test. For example, if we had assigned an incorrect interface IP address on the new leaf, _test\_bgp\_status()_ would fail because not all sessions would be in ESTABLISHED state. You may then look at the output of _bgpSessionStatus_ question, which for this example will show that the sessions on leaf03 and spine01 are incompatible. To understand why, you may then run the _bgpSessionCompatibility_ question as shown in the figure below. ![figure](/assets/images/2021-05-batfish-suzieq/fig-3.png).
This output tells you that you likely have the wrong IP address on leaf03 (NO_LOCAL_IP), and that spine01 expects to establish a session to 10.127.0.5 but no such IP is present in the snapshot (UNKNOWN_REMOTE). If you fix the configs, and rerun the tests,  they should all pass now, and you can be confident that your change is ready to be scheduled for deployment.

### Deployment pre-testing

Pre-approval testing happened against the network snapshot that existed then. When the time comes to deploy the change during the maintenance window, the network may be in a different state. Some links may have failed and the planned change could interact with the failures in unexpected ways. Or, the network’s configurations may have drifted in an incompatible way. We must thus test that the change is safe to deploy right before deploying it. 

A combination of Batfish and Suzieq enables deployment pre-testing. Suzieq will fetch the latest network configs and state (new in version 0.12). You can feed those new configs to Batfish along with the planned config change and re-run the tests that you ran before. This re-run confirms that the change is still correct and is compatible with the current network configuration. 

Suzieq helps you test that the network is in a state that is ready for the change you’re about to make. For example, if one of the spines is down, then attempts to configure it will fail. Alternatively, you must verify that the spine configuration change is using a port that is not already in use. It is important to double check our assumptions about the state of the network. Measure twice, cut once, as the adage goes. If there’s an unexpected surprise, you can abort proceeding with the change (no rollback needed). 
 
As in the case of Batfish, your automated test suite will be a Python program. ![snippet](/assets/images/2021-05-batfish-suzieq/fig-4.png) The screenshot above shows how you can use Suzieq to test that the spines are alive, the port Ethernet3 being provisioned to connect to the new leaf is free, and that the SVI prefix being allocated is unused.

Each test uses _get\_sqobject()_ to get the relevant tables, then uses the get function to retrieve the rows and columns of interest, and finally checks that a specific column has an expected value on all nodes. The .all() checks that the field has that value on all rows of the retrieved dataset. Thus, the test to check that all spines are alive uses the “device” table to retrieve information about the spines, and checks that the “status” column has the value “alive” in all rows. _test\_spine\_port\_is\_free()_ assumes that the spine ports have been cabled up and uses the lack of an LLDP peer to confirm that the port connecting to the new leaf is unused

Like Batfish, this code is vendor-agnostic and works for any Suzieq-supported vendor. If you add additional leafs, you just need to change the values of SPINE\_IFNAME and NEW\_SVI\_PREFIX. This is the power of writing tests using frameworks like Suzieq.

If all deployment pre-tests pass, you can confidently deploy the change. But before you declare victory, you still need to test that the deployment went as planned. So, let's do that next.

### Deployment post-testing

Deployment post-testing aims to verify that the change was successful. A simple list of things to test our example change include: the spines are now peering correctly with the new leaf, the new SVI prefix is correctly assigned, and that the SVI prefixes of all the leafs are reachable via all the other leafs.

The Python program to test all this looks as follows: ![follows](/assets/images/2021-05-batfish-suzieq/fig-5.png)
Before running deployment post-tests, make sure that you have added the new leaf to the Suzieq inventory and restarted the poller to gather data from the new leaf as well. 

Suzieq can also perform a battery of tests using the aver method of the  table. For example, if you had accidentally deployed the config with the incorrect IP address on leaf03, you can use the interface aver which checks for consistency of addresses, MTUs, VLANs, etc. The output would look as follows: ![follows](/assets/images/2021-05-batfish-suzieq/fig-6.png)
Now you can examine the interface IP addresses of leaf03/Ethernet1 and spine01/Ethernet3 to determine which of the two has the incorrect IP address and fix it.

## Summary

As in the software domain, rigorous testing is key to evolving the network rapidly and safely. Closed-loop testing leads to the least surprise with the most reliability. It is  done in three stages—-pre-approval testing, deployment pre-testing, and deployment post-testing—-and catches errors at the appropriate point during the deployment. However, writing automated tests can be a complex process without the help of appropriate tools. Fortunately, open source tools like Batfish and Suzieq greatly simplify writing and maintaining automated tests. Give them a try and make your network changes robust and error free. 
