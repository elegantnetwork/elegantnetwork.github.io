---
layout: post
comments: true
author: Dinesh G Dutt
title: A Tale of Four Python SSH Libraries
excerpt: An evaluation of different SSH libraries in Python
description: Why asynchronous support and readability in code are first among equals.
---
In Suzieq, we needed to select a python library to fetch data from network devices (and servers) via SSH. This led us to an evaluation of the four popular SSH libraries and the selection of one of them for use in Suzieq. In this article, we'll discuss our requirements and our selection process.

## Our Requirements

It is 2020. Python 3 is the only choice for anyone choosing to program in Python. We were lucky to have begun when we did. Not everyone had the choice of automatically selecting Python 3. Not so long ago, Python 3 was still evolving while Python 2 was steady as a rock. Most OS distributions shipped with Python 2, and the majority of libraries that were available were in Python 2, not Python 3. Nevertheless, we automatically selected libraries that were robust in Python 3, or took advantage of Python 3's advanced functionalities.

In Suzieq, we're only interested in read only commands. We do not use a single configuration command. In other words, we're only executing show commands on the devices we're probing. This simplifies the requirements we have of our SSH library. Most network devices are modal in nature i.e. the command assumes the context of where in the command hierarchy the user is when executing the command. For example, if we issue a command "neighbor Ethernet1/1 remote-as 65000", the network device's command line parser accepts this as a valid command only within the BGP context i.e. the user has issued for example "router bgp 65001" prior to this command. They sadly lack the equivalent of a "&&" or ";" that Unix shells provide.

Network operation lends itself well to asynchronous operation. In Python 3, especially starting with python 3.5, asynchronous programming became easy. In subsequent releases, all the way upto python 3.7, the readbaility and ease of use of the async mode only got better and many new libraries sprang up that did asynchronous versions of the common use cases such as REST and SSH. There's even an async version of SNMP use. 

To those who may not know the difference, asynchronous mode enables concurrency, which is not the same as parallelism. In concurrent tasks, a task executes unless it has to perforn some I/O operation at which point it yields while waiting for tje I/O to complete. Another task can start executing at the yield point, which will then yield when it is waiting for an I/O to complete. In other words, concurrent tasks use cooperative multi-tasking. With parallel processes, the two tasks can be preempted by the kernel scheduler at any point i.e. they use preemptive multitasking. If there perform no I/O, two concurrent tasks will execute serially one after the other. Parallel tasks will execute a little at each scheduling slice even without any I/O. The answers that address this can be found in this [StackOverflow thread](https://stackoverflow.com/questions/27435284/multiprocessing-vs-multithreading-vs-asyncio-in-python-3) for those interested in knowing more.

Given that Suzieq will be polling many network devices, an async library version is preferred over the non-async version. 

We were also looking for a library that performs well i.e. is fast. And finally, we were looking for one that would support the standard models for using SSH:
- Ignoring Host Key Files
- Supporting Private Key FIles instead of password, including private key file with passphrase
- Support communicating via a jump host
- Support ssh config files
- Support using ssh-agent

## The Contenders

Networking devices are notorious for not behaving like proper shells when you use SSH to connect to them. Kirk Byers, the author of netmiko, the first contender, wrote a [nice post](https://pynet.twb-tech.com/blog/automation/netmiko.html) about why he created the Netmiko library. He explains some of these problems. Netmiko also comes with the ability to parse the command output and return it as a structured output via the popular command parsing library, [textfsm](https://github.com/google/textfsm). Netmiko is used in the popular [NAPALM](https://napalm.readthedocs.io/en/latest/) network device access library. netmiko itself is based off of paramiko, the second popular SSH library in python. Paramiko is what popular tools such as [Ansible](https://docs.ansible.com/ansible/latest/index.html) use. The third contender is a relatively new library that billed itself as being very fast and the basis for a fast parallel SSH client library, parallel-ssh (remember note about parallelism and concurrency above). The final contender is asyncssh, an asynchronous full-featured SSH implementation of SSH.

All four libraries satisfy all the SSH functionality defined in the previous section. Needless to say, asyncssh is the only async library of the four libraries. 

## Benchmarking Setup

I spun up a number of topologies using Vagrant. The simulations had different NOS such as Cumulus, Cisco's NXOS, Arista EOS and JunOS. This allowed me to verify that changing the NOS didn't affect the test results. I ran the benchmark test from my laptop, a Lenovo Yoga with i7-8550U CPU and 16GB RAM and SSD. The simulations, except for Junos, ran on a different machine, an Intel NUC with an i7-8550U processor with 64GB RAM and an SSD. The network connectivity between my laptop and the NUC was wireless. The simulation using Junos ran on my laptop as well because of the IP addressing of Virtualbox (the addressing is only exposed on the local machine by default). JunOS VM is available on virtualbox only and I could not make it work on libvirt, unlike the other NOS. 

I verified that the overall timing values were not affected if I shifted the order in which the different libraries were run i.e. I sometimes ran asyncssh first, netmiko second and so on while at other times I ran paramiko first, ssh2 second and so on.

Netmiko has a lot more possible parameters to configure to ensure that I was doing an apples to apples comparison between the SSH performance of the various libraries. For example, I verified if setting the NOS type specifically in the connection parameters versus asking netmiko to autodetect made a difference. I didn't test this option against all NOS, but against NXOS version 9.3.4, setting it to autodetect consistently performed better and so I left that parameter to autodetect in all the tests. Similarly, I set the use_textfsm parameter in command execution to False to ensure that the timings were not affected by any additional parsing that the library was performing after the data was obtained.

I ran what I think is a simple, common command in each case for the NOS, "show version" (for classical NOS) and "uname -a" for Cumulus and Linux servers.

### Benchmarking using timeit

Python comes with a module [timeit](https://docs.python.org/3/library/timeit.html) that's supported with the base Python distribution. I used it to get the execution times for each of the four libraries. I measured a single host execution time as well as a multi-host execution time. While it is possible to write more complex code to do thread management myself, I first chose to ignore this model and execute the comamands in as simple a fashion as possible using the library. In benchmarking methodology, its generally accepted practice to execute multiple runs of the command and average out the execution time across all those times. I used a repeat count of 3 and 10 to obtain the timings because of the time to execute a command, though most benchmarking methodologies typically employ much larger numbers. 

## The results

Here are some of the outputs of running the test: 
```
$ python ssh_timeit.py 
Running single host timing for simulation: nxos_sim
SINGLE HOST RUN(Avg of 3 runs)
-------------------------------------------
asyncssh: 3.1735380279715173
ssh2: 4.196927884011529
paramiko: 4.212026820983738
netmiko: 7.261537566955667

Running multi-host timing for simulation: nxos_sim, 4 hosts
MULTI HOST RUN(Avg of 3 runs)
------------------------------------------
asyncssh: 5.695307980990037
ssh2: 9.726953078003135
paramiko: 15.177161411033012
netmiko: 25.630355022032745
```

And here is the same thing with 10 runs:
```
$ python ssh_timeit.py 
Running single host timing for simulation: nxos_sim
SINGLE HOST RUN(Avg of 10 runs)
-------------------------------------------
asyncssh: 4.884247861045878
ssh2: 10.369624491024297
paramiko: 9.094426751020364
netmiko: 26.214536760991905

Running multi-host timing for simulation: nxos_sim, 4 hosts
MULTI HOST RUN(Avg of 10 runs)
------------------------------------------
asyncssh: 5.8694626719807275
ssh2: 34.19502033101162
paramiko: 35.254284786991775
netmiko: 97.73253851704067
```

For 10 hosts, here are the test numbers (with Cumulus):
```
$ python ssh_timeit.py 
Running single host timing for simulation: k8s_sim
SINGLE HOST RUN(Avg of 3 runs)
-------------------------------------------
asyncssh: 1.6649398019653745
ssh2: 3.8646505440119654
paramiko: 2.153029495035298
netmiko: 8.268776792043354

Running multi-host timing for simulation: k8s_sim, 10 hosts
MULTI HOST RUN(Avg of 3 runs)
------------------------------------------
asyncssh: 4.410694238031283
ssh2: 16.54564989701612
paramiko: 22.772781688021496
netmiko: 66.13297272496857
```

And one more with Junos:
```
$ python ssh_timeit.py 
Running single host timing for simulation: junos_sim
SINGLE HOST RUN(Avg of 10 runs)
-------------------------------------------
asyncssh: 1.7394950980087742
ssh2: 1.5635987159912474
paramiko: 1.7370304619544186
netmiko: 12.3502985839732

Running multi-host timing for simulation: junos_sim, 4 hosts
MULTI HOST RUN(Avg of 10 runs)
------------------------------------------
asyncssh: 2.4084964870125987
ssh2: 5.61457060900284
paramiko: 5.622337078035343
netmiko: 54.29702621902106
```
A few things jump out at these outputs:
- The single host performace across the three libraries, asyncssh, libssh2 and paramiko, are roughly equivalent. If I run the test enough times, I can make any one of the three best the others.
- Netmiko is consistently the slowest
- In the case of multihost performance, asyncssh always beats out the others, by a fairly wide margin, at least 15x faster than the slowest.

## Acting on the Results

So, you may wonder, how do people deal with multiple hosts using libraries other than asyncssh. They build their own version of concurrency by either using threads or processes. Ansible uses processes, if I remember correctly. Here is a link to a [post](https://www.consentfactory.com/python-threading-queuing-netmiko/) that shows how such a code might be written (I just randomly picked an entry from the search result). Python's asyncio library uses multi-threading by default, not multi-processing, though you can write code to adapt it to use multiprocessing. 

### Code Readability

But this brings me to another criterion in helping us decide on which library did we wanted to use in Suzieq. How simple and easy to read is the code. First up are the best answers in my opinion, asyncssh and netmiko. Here they are:
```python
async def async_ssh(host, port=22, user='vagrant', password='vagrant'):
    conn = await asyncssh.connect(host, port=port,
                                  username=user, password=password,
                                  client_keys=None,  known_hosts=None)
    output = await conn.run(command)
    # print(f'{host}, {output.stdout.strip()}, {output.exit_status}')
    conn.close()

def netmiko_ssh(host, port=22, user='vagrant', password='vagrant'):

    dev_connect = {
        'device_type': 'autodetect',
        'host': host,
        'port': port,
        'username': user,
        'password': password
    }

    net_connect = ConnectHandler(**dev_connect)
    output = net_connect.send_command(command, use_textfsm=False)
    net_connect.disconnect()
    # print(output)
```
Both are fairly easy to follow and hide all sorts of low level details from the code. asyncssh has the equivalent of netmiko's dev_connect variable model that you can use instead of passing the parameters in the call to connect as we've done. Paramiko is fairly equivalent, if not as terse as asyncssh.

Next up is ssh2-python (this is taken from ssh2-python's examples):
```python
def ssh2_ssh(host, port=22, user='vagrant', password='vagrant'):
    # Make socket, connect
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((host, port))

    # Initialise
    session = Session()
    session.handshake(sock)

    session.userauth_password(user, password)

    # Public key blob available as identities[0].blob

    # Channel initialise, exec and wait for end
    channel = session.open_session()
    channel.execute(command)
    channel.wait_eof()
    channel.close()
    channel.wait_closed()

    # Print output
    output = b''
    size, data = channel.read()
    while size > 0:
        output += data
        size, data = channel.read()

    # Get exit status
    output = output.decode("utf-8").strip()
    # print(f'{host}, {output}, {channel.get_exit_status()}')
```
This is far less elegant than the first two, exposing sockets, sessions and channels. While one could argue that all this code could be tucked away in a routine and made to look as elegant as asyncssh or netmiko, I'm inherently lazy. I don't want to do work that I don't want to, unless there's a real strong motivation to do it.

## Summary

Asyncssh is the python ssh library used in Suzieq. Its successfully connected to Juniper MX, Juniper QFX, Cisco's 9K, Cumulus, Arista and SONIC machines without a problem. We use textfsm internally on the gathered data, if structured output is not available. Given our requirements, this was the best choice. Our choice was validated even more by how helpful the maintainer of asyncssh, Ron Fredericks is. I needed help with figuring something out, and he sent me an excellent, detailed and thoughtful response.

While this is not a professional benchmarking article, I hope it helped the readers appreciate what a serious difference asyncio makes in performance.
