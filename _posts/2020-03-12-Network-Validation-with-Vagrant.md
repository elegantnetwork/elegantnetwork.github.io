---
layout: post
comments: true
author: Justin Pietsch
title: Network Simulation for Design Validation using Vagrant on Cloud Computing
excerpt: Simulating network design is really important and we can use cloud computing to try things out.
description: 
---


## Network Simulation for Design Validation using Vagrant on Cloud Computing

## Simulation for Design Validation

A very important tool in understanding networks and network design choices is simulation. Simulating networks is **critical for designing almost any interesting network**, and yet it’s rarely done. It’s a travesty in our discipline that we don’t have good and easy ways to simulate networks. **Engineers need ways to test ideas and get good information to make the right decisions**. For us network engineers, network simulation is an important way to verify and test idea. Great simulation helps make good ideas clear and helps test out thoughts much more quickly.

## Network Simulation at Amazon

This lack of great simulation is largely pervasive across the industry. Even at Amazon where I previously worked, in too many cases the only reason the network worked (and works) is because really great network engineers would whiteboard and argue with each other to prove out designs. They used themselves to be the protocol engines to figure out how things would work. If things were really complicated they would have somebody setup a lab and try out specific features on limited deployments. Especially when Amazon was small, this was very difficult; we rarely had extra lab gear to use to try out ideas.

When Amazon decided to go to dis-aggregated commodity merchant silicon direction ([https://perspectives.mvdirona.com/2010/10/datacenter-networks-are-in-my-way/](https://perspectives.mvdirona.com/2010/10/datacenter-networks-are-in-my-way/)) **I used simulation to make sense of all our different design options**. The hardware that was available had low end processors, and the Network Stack available to us was unknown and not widely deployed by anyone. Going from large chassis devices to pizza box, top-of-rack switches meant an order of magnitude more devices, so it required a different design, including routing. I couldn’t get my head around the implications and trade-offs. The only simulation system I knew was [Dynamips](https://github.com/GNS3/dynamips), which is a hardware emulator that works for MIPS based Cisco software images for hardware that does CPU based forwarding. This is not the hardware or software we were going to use, but it had working protocol stacks which is all that I needed. **Obviously we wouldn’t find software bugs, or performance issues, but we would find bugs in our design. This was highly successful; I could refine the design until it scaled well inside the simulation.** I would not have believed our design would have worked until we could test a full setup without this, and that would have been much later and much harder to try out and compare options.

## What to Simulate in Network Simulation

Engineering is the art of making trade-offs. Network simulation is no different. It is not perfect, but neither is it useless. **The first rule is to understand the purpose of the simulation**. You can learn a lot from what and how you decide to simulate. Furthermore, it is much faster, cheaper, and easier to turn up simulations than build test labs. When you use a box like Ixia, for example, you’re typically running a simulator too, to test the data path of the switch. With the network simulation I discuss in this post, you can’t try out packet forwarding performance or test the switching processor data path, but you can examine network design, and protocol design trade0offs. For instance, in my simulation if I used aggregation and sumarization it would converge in a reasonable amount of time. If I didn’t, it wouldn’t converge.

## Current State of Network Simulation

I wish there was a good open source platform for network simulation. Simulation can be complicated. You want a system that simulates the network software, but also you want a way to generate different configuration ideas as well as a set of tools to test that the design behaves well. We don’t have all those pieces. While the ones that exist are lacking in many ways, they’re good enough to be useful.

**I’m going to focus on [Vagrant](https://www.vagrantup.com/) for device simulation. **Vagrant is fairly simple and focuses on packaging and running Virtual Machines. This means it has good flexibility and that we can run fairly large simulations.

There other simulation platforms that might also be interesting. Microsoft has an internal network simulation that they have been working to open source: [https://www.microsoft.com/en-us/research/wp-content/uploads/2017/10/p599-liu.pdf](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/10/p599-liu.pdf), and [https://www.facebook.com/atscaleevents/videos/networking-scale-2019-preventing-network-changes-from-becoming-network-outages/370479100557737/](https://www.facebook.com/atscaleevents/videos/networking-scale-2019-preventing-network-changes-from-becoming-network-outages/370479100557737/). As far as I can tell, this still hasn’t been open sourced or released to the public yet.

Another interesting, but not open software is from [https://www.tesuto.com/](https://www.tesuto.com/). I haven’t tried this out. I did watch a Tech Field Day presentation of Tesuto that was really good, and convinced me that they have good ideas on what is actually needed for network engineers. [https://www.youtube.com/watch?v=Wo5qef7WsC4](https://www.youtube.com/watch?v=Wo5qef7WsC4)

While not the same thing, [https://nrelabs.io/](https://nrelabs.io/) uses network simulation to teach network automation. I believe that [Antitote](https://github.com/nre-learning/antidote), the platform nrelabs runs on uses vagrant for simulation.

## Network Simulation on the Cloud

**I’m going to show you how to setup simulation in the cloud.** While you can simulate on your laptop or any local computer, the advantage of using cloud computing is that you can spin up something quickly and then take it down quickly. As we get better tools, I hope that we can simulate bigger networks and then we can take advantage of more hardware. Also, if you want to run several different experiments, then you can easily spin up parallel cloud computing simulation environments.

The cool thing about Cloud Computing is that you can just spin up instances quickly and then turn them off and not get charged. So while monthly or yearly you might get expensive instances, it’s really cheap. You can get a lot done in an hour with one instance. **Just don’t forget to turn your instances off when you are done.**

## Choosing topology and configuration to try out

The topologies and configuration I uses are from [https://github.com/ddutt/cloud-native-data-center-networking](https://github.com/ddutt/cloud-native-data-center-networking) (CNDCN). One of the big bottlenecks to simulation is that there isn’t easy ways to describe network designs and produce configs, so we will start with some pre-built ones. These are small and interesting topology and you can pick different routing options and compare. There are examples with BGP, OSPF, EVPN, docker, and routing on the host. It also includes ways to use Cumulus Linux or Arista. I’ve not tried the Arista vagrant image. I do know that the CL instances use much less memory than the Arista one and boot up significantly faster, both of which mean they are easier to scale.

## Vagrant Intro

Vagrant ([https://www.vagrantup.com/intro/index.html](https://www.vagrantup.com/intro/index.html)) is a tool to manage Virtual Machines, which it calls a box. Vagrant can run on several different Operating Systems and Virtual Machine technology. If you want to run on a Mac or Windows machine, then you probably want to use VirtualBox. But these don’t scale well. [https://libvirt.org/](https://libvirt.org/) scales much better, but it’s a little tricky to setup, which is why I wrote this post.

## Setting up cloud computing

**I’m going to focus on using EC2 and Azure, using libvirt to get good performance performance**, but this means that libvirt on cloud computing is running virtualization on top of virtualization. EC2 doesn’t support this, but you can run on the bare metal EC2 instances. These are all named *.metal and provide more hardware than we actually need in the Cloud Native Datacenter Networking examples.

If you haven’t used your EC2 account very much or are new, then you do not have permission to use any of the *.metal instances; they have more vCPU than your account allow so you must ask for greater access. I got access to one *.metal instance in one region; it seemed too hard to get more instances or in more than one region. I picked a C5.metal instances and it’s at least $4.00 an hour. I recommend trying to use spot instances, which should be dramatically cheaper.

Azure does allow virtualization on top of virtualization, so many of their images work. When you first start an Azure account, you get $200 of credits, but you can only apply them to instances that have 4 vCPU or less, which is pretty limited but still works. I chose the E4-V3 instances which have 4vCPU and 32 GB of RAM. I’ve not used larger images so I don’t know the performance impact of running virtualization on top of virtualization

**When you choose a host image, choose Ubuntu 18.04**. I know that all the software works on that image. I’ve not tried anything else. I believe Dinesh has tried Ubuntu 19.04, but those images aren’t available on cloud computing at this point.

To check that your Ubuntu image will allow the virtualization that libvirt needs, run ‘kvm-ok’. If that doesn’t work, then install cpu-checker ‘sudo apt install cpu-checker’

you’ll either see something like

    ubuntu@ip-172-31-62-27:~$ kvm-ok
    INFO: Your CPU does not support KVM extensions  
    INFO: For more detailed results, you should run this as root  
    HINT: sudo /usr/sbin/kvm-ok

or if it works

    jpiet@tra1:~$ kvm-ok  
    INFO: /dev/kvm exists  
    KVM acceleration can be used

If you get the first entry, then you need a different instance type.

There is a lot of software that is needed to get [ansible](https://www.ansible.com/), vagrant, and libvirt working. I’ve created a setup script to do all of that [setup_vagrant.sh](https://github.com/jopietsch/network_simulation_tools/blob/master/setup_vagrant.sh). If anybody has better ways of packaging all these together, please let me know. All the information in that script are from: [https://github.com/ddutt/cloud-native-data-center-networking](https://github.com/ddutt/cloud-native-data-center-networking), [https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-debian](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-debian), and [https://github.com/vagrant-libvirt/vagrant-libvirt#installation](https://github.com/vagrant-libvirt/vagrant-libvirt#installation)

This will take 10–15 minutes.

    PS C:\Users\jpiet\code\network_simulation_tools> scp setup_vagrant.sh jpiet@40.71.19.175:~/ setup_vagrant.sh                                             100% 1533    17.4KB/s   00:00

    jpiet@b3:~$ ls setup_vagrant.sh 
    jpiet@b3:~$ chmod a+rx setup_vagrant.sh 
    jpiet@b3:~$ time ./setup_vagrant.sh  
    ...  
    real    9m2.273s user    4m29.169s sys     0m56.578s

## Trying out virtualization

At this point, you have an Ubuntu image that is ready to try out Cloud Native Datacenter Networking, and you’ve downloaded that code.

Vagrant relies on a file called Vagrantfile. It can run as many of those as you have. In the CNDCN repository, there are two of those that are based on the topologies. Either single-attach or dual-attach. cd into CNDCN/topologies and pick one of the two. I’ll pick dual-attach. cd into that directory. Then type ‘sudo vagrant up.’ Then sets up the vagrant instances and connects them. At this point they aren’t doing anything interesting. It’s worth it to look at

[README.md](https://github.com/ddutt/cloud-native-data-center-networking/blob/master/README.md) if you haven’t already.

    jpiet@b3:~/cloud-native-data-center-networking/topologies/dual-attach$ ls 
    Vagrantfile  Vagrantfile-kvm  Vagrantfile-vbox  bgp  dummy.yml  evpn helper_scripts  ospf 
    jpiet@b3:~/cloud-native-data-center-networking/topologies/dual-attach$ sudo vagrant up

    jpiet@t14:~/cloud-native-data-center-networking/topologies/dual-attach$ sudo vagrant status 
    Current machine states: 
    exit02                    running (libvirt) 
    exit01                    running (libvirt) 
    spine02                   running (libvirt) 
    spine01                   running (libvirt) 
    leaf04                    running (libvirt) 
    leaf02                    running (libvirt) 
    leaf03                    running (libvirt) 
    leaf01                    running (libvirt) 
    edge01                    running (libvirt)
    server101                 running (libvirt) 
    server103                 running (libvirt) 
    server102                 running (libvirt) 
    server104                 running (libvirt) internet                  running (libvirt)

I’m going to show BGP un-numbered, but there are lots of other options. cd into the bgp directory and use Ansible to push the software. ‘ansible-playbook -b deploy.yml’.

    jpiet@t14:~/cloud-native-data-center-networking/topologies/dual-attach/bgp$ ansible-playbook -b deploy.yml  
    ...   
    PLAY RECAP ********************************************************************************************************
    edge01                     : ok=9    changed=2    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0 
    exit01                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    exit02                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    internet                   : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    leaf01                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    leaf02                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    leaf03                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    leaf04                     : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    server101                  : ok=6    changed=2    unreachable=0    failed=0    skipped=18   rescued=0    ignored=0 
    server102                  : ok=6    changed=2    unreachable=0    failed=0    skipped=18   rescued=0    ignored=0 
    server103                  : ok=6    changed=2    unreachable=0    failed=0    skipped=18   rescued=0    ignored=0 
    server104                  : ok=6    changed=2    unreachable=0    failed=0    skipped=18   rescued=0    ignored=0 
    spine01                    : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    spine02                    : ok=15   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

When that is done, you can check to make sure it has connectivity. ansible-playbook ping.yml

    jpiet@t14:~/cloud-native-data-center-networking/topologies/dual-attach/bgp$ ansible-playbook ping.yml  
    ...  
    PLAY RECAP ********************************************************************************************************
    server101                  : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    server102                  : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    server103                  : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
    server104                  : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

At this point, you can log into any of the devices and see what’s going on.

    jpiet@t14:~/cloud-native-data-center-networking/topologies/dual-attach/bgp$ vagrant ssh leaf01  

    Welcome to Cumulus VX (TM)  

    Cumulus VX (TM) is a community supported virtual appliance designed for experiencing, testing and prototyping Cumulus Networks' latest technology. For any questions or technical support, visit our community site at: http://community.cumulusnetworks.com  

    The registered trademark Linux (R) is used pursuant to a sublicense from LMI, the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide basis.

You can investigate the setup, or try out different configuration.

    leaf01# sh ip bgp summary 
    IPv4 Unicast Summary: 
    BGP router identifier 10.0.0.11, local AS number 65101 vrf-id 0 
    BGP table version 33 
    RIB entries 29, using 4408 bytes of memory 
    Peers 2, using 39 KiB of memory 
    Peer groups 1, using 64 bytes of memory 
    Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd 
    spine01(swp1)   4      65000     159     165        0    0    0 00:06:54           12 
    spine02(swp2)   4      65000     159     165        0    0    0 00:06:53           12 
    Total number of neighbors 2

You can try new topologies or new routing. If you want to just try one of the different routing scenarios, then run ‘ansible-playbook reset.yml, and then start again.

When you are all done, **remember to stop your cloud instance**.

Here is some performance data. I also have a script [check_everything.sh](https://github.com/jopietsch/network_simulation_tools/blob/master/check_everything.sh) that runs through all of the different scenarios in CNDCN and you can see the different performance between the two instance types.

![Comparison of Cloud Computing](/assets/images/2020-03-13-vagrant_times.png)

The EC2 instances is way more hardware than this simulation needs and the Azure is under powered.

**Why not use container images**? Container images would allow larger scale testing, but don’t allow things like VXLAN. If the only thing you are simulating is the routing protocol suite. If I want to simulate more than that, like EVPN I can’t do that. With a VM you are

**Limitations of this approach**? The biggest limitation is that **not every vendor provides VMs of their network OS**. Cumulus Linux is the most featured. Arista used to release a Vagrant box, but has since stopped and switched to a container-only model; both were limited. Cisco doesn’t have a VM image as far as I know. Juniper also releases a Vagrant image. The other constraint with all the vendors except Cumulus is that the image is quite restricted, for example allowing only a handful of interfaces, and with phenomenally poor packet forwarding rates (a few Mbps instead of whatever the processor can handle). This approach from the vendors does not make it easy to use network simulation.

Use the discussion format or email me if you know of anything that would make this better. Tips or tricks, software, etc?
