---
layout: post
comments: true
author: Dinesh G Dutt
title: A Tale of Five Python SSH Libraries
excerpt: An evaluation of different SSH libraries in Python
description: An evaluation of different SSH libraries in Python
---
In Suzieq, we needed to select a python library to fetch data from network devices (and servers) via SSH. This led us to a search for and evaluation of five SSH libraries, which I thought were the most suitable for the task at hand.

## Requirements

We had a fairly simple list of requirements from the SSH libraries:
- Good performance
- Scalable
- Python 3 support 
- Support the common authentication and connection models used in a majority of the enterprises for connecting to network devices.

Lets examine them in some detail now.

### Asyncio

Network operation lends itself well to asynchronous operation. In Python 3, especially starting with python 3.5, asynchronous programming became easy. In subsequent releases, all the way upto python 3.7, the readbaility and ease of use of the async mode only got better and many new libraries sprang up that did asynchronous versions of the common use cases such as REST and SSH. There's even a Python [async library for SNMP](https://pypi.org/project/aiosnmp/).

To those who may not know the difference, asynchronous mode enables concurrency, which is not the same as parallel execution. A simple to understand model for understanding this difference is that concurrency is useful even with a single core while parallel execution inherently relies on multiple cores. To explain this with more words, any task has two pieces in its execution: CPU and I/O. When a task is executing I/O, its typically waiting for a long time for the IO to complete and fetch the results. Think of disk reads or network communication as examples of waiting for I/O. While the task is waiting, other tasks can be scheduled to execute. Any application that can take advantage of this waiting to work on other things is said to support concurrency. In case of parallel execution, the task is broken up into multiple individual pieces and each piece executes simultaneously on multiple cores. Consider performing a common operation on multiple elements in a list as an example of a parallel operation. 

On the basis of this explanation, it's hopefully clearer that concurrency is far more critical and natural with network I/O and benefits networking code independently of whether the task can be parallelized. For example, you can use a pool of processes to parallelise rendering of a template while at the same time using a pool of threads for pushing the template to different network devices.

Given that Suzieq will be polling many network devices, an async library version is preferred over the non-async version to provide both good performance and for scaling.

### Authentication & Connection Method Support

Connecting to network devices in enterprises and most organizations (except hyperscalars who have the ability to put together a good public key infrastructure to use certificate-based authentication), requires support for at least the following features:
- Ignoring Host Key Files
- Supporting Private Key FIles instead of password, including private key file with passphrase
- Support communicating via a jump host
- Support ssh config files
- Support using ssh-agent

Any python library we used in Suzieq was required support for these libraries.

### SSH and Network Devices

Networking devices are notorious for not behaving like proper shells when you use SSH to connect to them. Kirk Byers, the author of netmiko, the first contender, wrote a [nice post](https://pynet.twb-tech.com/blog/automation/netmiko.html) about why he created the Netmiko library. He explains some of these problems. 

The primary problem has to do with how configuration commands work with network devices. Network device shells are contextual, especially so for configuration. For example, if you want to issue a command to assign an IP address to an interface. You need to first issue a command "configure", followed by "interface Ethernet 1/1", followed by "ip address 192.168.1.1". Issuing three independent commands will not work nor will issuing the three separated by ";". So, it would be very helpful for an SSH library intending to support communicating with network devices to help with this. 

Fortunately, Suzieq never uses any configuration command. All its commands are so called "show" commands. Because of this, communicating with network devices for Suzieq is no different than communicating with any Linux server or Linux NOS such as Cumulus or SONIC(I don't know if SONIC supports Linux-like shells or only traditional network device-like shells). As a consequence, we can ignore requiring the SSH library used within Suzieq from needing to support context-driven configuration commands. 

## The Contenders

The first contender is [Netmiko](https://pypi.org/project/netmiko/). Netmiko also comes with the ability to parse the command output and return it as a structured output via the popular command parsing library, [textfsm](https://github.com/google/textfsm). Netmiko is used in the popular [NAPALM](https://napalm.readthedocs.io/en/latest/) network device access library. netmiko itself is based off of [paramiko](https://github.com/paramiko/paramiko/), the next contender. Paramiko is what popular tools such as [Ansible](https://docs.ansible.com/ansible/latest/index.html) use. The third contender is a library that I ran into because it billed itself as being very fast and the basis for a fast parallel SSH client library, parallel-ssh (remember note about parallelism and concurrency above). The fourth contender is asyncssh, an asynchronous full-featured SSH implementation of SSH. The final contender is a relative newcomer called [scrapli](https://github.com/carlmontanari/scrapli). Scrapli is not itself an SSH library, but a wrapper around paramiko, asyncssh and ssh2 SSH libraries. It provides both a synchronous and an asynchronous version of SSH connection to network devices. However, unlike asyncssh which doesn't provide any additional support for network devices, scrapli tries to make it easy to connect to the network devices and issue configuration commands. We'll be using the scrpali wrapper around asyncssh in the tests below. However, scrapli doesn't provide any support for connecting to anything Linux-y like Cumulus and SONIC(??) or servers, and it also supports far fewer devices than Netmiko supports, at the time of this writing.

All five libraries satisfy all the SSH functionality desired by Suzieq. Here is a table to compare the different libraries and their features:

|  | asyncssh | scrapli | ssh2-python | paramiko | netmiko |
| :--: | :--------: | :-------: | :-----------: | :--------: | :-------:|
| Version Tested | 2.4.0 | 2020.8.28 | 0.17.0 | 2.7.2 | 3.3.0 |
| Python3 Support | yes | yes | yes | yes | yes |
| Asyncio Support | yes | yes | no  | no  | no  |
| Network Device Config Support | no | yes | no | yes | yes |
| Linux Device Support | yes | no | yes | yes | yes |
| Password Auth | yes | yes | yes | yes | yes |
| Private Key | yes | yes | yes | yes | yes |
| Private Key with Passphrase | yes | yes | yes | yes | yes |
| Host Key Ignore | yes | yes | yes | yes | yes |
| Jumphost Support | yes | ?? | yes | yes | yes |

## Benchmarking Setup

I spun up a number of topologies using Vagrant. The simulations had different NOS such as Arista's EOS, Cisco's NXOS, Cumulus Linux, and JunOS. If the number of the hosts tested varies across the NOS, its because of whatever simulation I had ready to spin up. This allowed me to verify that changing the NOS didn't affect the test results. I ran the benchmark test from my laptop, a Lenovo Yoga with i7-8550U CPU and 16GB RAM and SSD. The simulations, except for Junos, ran on a different machine, an Intel NUC with an i7-8550U processor with 64GB RAM and an SSD. The network connectivity between my laptop and the NUC was wireless. The simulation using Junos ran on my laptop as well because of the IP addressing of Virtualbox (the addressing is only exposed on the local machine by default). JunOS VM is available on virtualbox only and I could not make it work on libvirt, unlike the other NOS. I used python 3.7.5.

I verified that the overall timing values were not affected if I shifted the order in which the different libraries were run i.e. I sometimes ran asyncssh first, netmiko second and so on while at other times I ran paramiko first, ssh2 second and so on.

Netmiko has a lot more possible parameters to configure to ensure that I was doing an apples to apples comparison between the SSH performance of the various libraries. For example, I verified if setting the NOS type specifically in the connection parameters versus asking netmiko to autodetect made a difference. I didn't test this option against all NOS, but against NXOS version 9.3.4, setting it to autodetect consistently performed better and so I left that parameter to autodetect in all the tests. Similarly, I set the use_textfsm parameter in command execution to False to ensure that the timings were not affected by any additional parsing that the library was performing after the data was obtained.

I ran what I think is a simple, common command in each case for the NOS, "show version" (for classical NOS) and "uname -a" for Cumulus and Linux servers. The code that I used for benchmarking is available via [this github gist](https://gist.github.com/ddutt/b511f462d05cafa67ae39d2243037c35).

### Benchmarking using timeit

Python comes with a module [timeit](https://docs.python.org/3/library/timeit.html) that's supported with the base Python distribution. I used it to get the execution times for each of the four libraries. I measured a single host execution time as well as a multi-host execution time. While it is possible to write more complex code to do thread management myself, I first chose to ignore this model and execute the comamands in as simple a fashion as possible using the library. In benchmarking methodology, its generally accepted practice to execute multiple runs of the command and average out the execution time across all those times. I used a repeat count of 3 and 10 to obtain the timings because of the time to execute a command, though most benchmarking methodologies typically employ much larger numbers. I noticed however that the times were consistently slower with 10 repeats vs 3 repeats.

## The results

Here are some of the outputs of running the test: 
```
$ python ssh_timeit.py nxos 10
Running single host timing for simulation: nxos
SINGLE HOST RUN(Avg of 10 runs)
-------------------------------------------
asyncssh: 5.310472506971564
scrapli: 14.621411228028592
ssh2: 7.677067372016609
paramiko: 11.580183147045318
netmiko: 27.1105942610302

Running multi-host timing for simulation: nxos, 4 hosts
MULTI HOST RUN(Avg of 10 runs)
------------------------------------------
asyncssh: 9.257326865044888
scrapli: 17.880212849995587
ssh2: 32.365094934997614
paramiko: 39.12168894999195
netmiko: 91.02830006496515
```
For 10 hosts, I used only 3 repeats because 10 repeats seemed to stress my server too much. The numbers are with Cumulus, and so no scrapli values:
```
$ python ssh_timeit.py cumulus 3
Running single host timing for simulation: cumulus
SINGLE HOST RUN(Avg of 3 runs)
-------------------------------------------
asyncssh: 0.6976866699988022
scrapli: -1
ssh2: 0.7593440869823098
paramiko: 0.9228836600086652
netmiko: 4.598790391988587

Running multi-host timing for simulation: cumulus, 10 hosts
MULTI HOST RUN(Avg of 3 runs)
------------------------------------------
asyncssh: 4.895733051002026
scrapli: -1
ssh2: 31.351762487960514
paramiko: 28.609976636013016
netmiko: 64.02421957900515
```
And one more with Junos:
```
$ python ssh_timeit.py junos 10
Running single host timing for simulation: junos
SINGLE HOST RUN(Avg of 10 runs)        
-------------------------------------------
asyncssh: 1.6286625529755838        
scrapli: 1.9315025190007873                  
ssh2: 1.5417965339729562                      
paramiko: 1.6812862670049071                 
netmiko: 12.91307156701805                   
                                                                                                         
Running multi-host timing for simulation: junos, 2 hosts
MULTI HOST RUN(Avg of 10 runs)     
------------------------------------------
asyncssh: 1.9371595539851114
scrapli: 2.3284904189640656
ssh2: 3.1178728850209154
paramiko: 3.258504494035151
netmiko: 25.267352762049995
```
A few things jump out at these outputs:
- The single host performace across the four libraries, asyncssh, scrapli, libssh2 and paramiko, are roughly equivalent. If I run the test enough times, I can make any one of them best the others, with the exception that Scrapli never bested asyncssh.
- Netmiko is consistently the slowest
- In the case of multihost performance, asyncssh always beats out the others, by a fairly wide margin, at least 15x faster than the slowest.

The results are summarized in this table below (note the slower times with 10 repeats vs 3 repeats):

|  | asyncssh | scrapli | ssh2-python | paramiko | netmiko |
| :--: | :--------: | :-------: | :-----------: | :--------: | :-------:|
| NXOS, Single Host, 10 Repeats | 5.3104 | 14.6214 | 7.677 | 11.58 | 27.1105 |
| NXOS, 4 Hosts, 10 Repeats | 9.2573 | 17.8802 | 32.265 | 39.1216 | 91.0283 |
| NXOS, Single Host, 3 Repeats | 3.1058 | 4.0649 | 3.9866 | 4.3019 | 9.1016 |
| NXOS, 4 Hosts, 3 Repeats | 4.1840 | 5.2396 | 17.8295 | 21.4979 | 35.8407 |
| Cumulus, Single Host, 10 Repeats | 1.9506 | -1 | 2.4926 | 3.3294 | 19.7455 |
| Cumulus, 10 Hosts, 10 Repeats | 11.315 | -1 | 66.3002 | 90.0536 | 215.1264 |
| Cumulus, Single Host, 3 Repeats | 0.6976 | -1 | 0.7593 | 0.9228 | 4.5987 |
| Cumulus, 10 Hosts, 3 Repeats | 4.8957 | -1 | 31.3517 | 28.6099 | 64.0242 |
| Junos, Single Host, 10 Repeats | 1.6236 | 1.9315 | 1.5417 | 1.6812 | 12.913 |
| Junos, 2 Hosts, 10 Repeats | 1.9371 | 2.3284 | 3.1178 | 3.3258 | 25.2673 |

## Concurrency with Synchronous IO

So, you may wonder, how do people deal with multiple hosts using libraries other than asyncssh. The answer is that they build their own version of concurrency by either using threads or processes. Ansible uses processes, if I remember correctly. Here is a link to a [post](https://www.consentfactory.com/python-threading-queuing-netmiko/) that shows how such a code might be written (I just randomly picked an entry from the search result). Python's asyncio library uses multi-threading by default, not multi-processing, though you can write code to adapt it to use multiprocessing. I don't want to do thread management if I can help it. The more I can rely on well-tested code, the more I can focus on my tool's value add and also focus on testing what's essential. 

### Code Readability

This brings me to another criterion in helping us decide on which library did we wanted to use in Suzieq. How simple and easy to read is the code. First up are the best answers in my opinion, asyncssh and netmiko. Here they are:
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
Both are fairly easy to follow and hide all sorts of low level details from the code. asyncssh has the equivalent of netmiko's dev_connect variable model that you can use instead of passing the parameters in the call to connect as we've done. Paramiko is fairly equivalent, if not as terse as asyncssh. Scrapli follows netmiko's model except that it doesn't have the autodetect mode, and so its code looks like this:
```python
async def scrapli_ssh(host, port=22, user='vagrant', password='vagrant'):

    dev_connect = {
        "host": host,
        "auth_username": user,
        "auth_password": password,
        "port": port,
        "auth_strict_key": False,
        "transport": "asyncssh",
    }

    if use_sim == nxos_sim:
        driver = AsyncNXOSDriver
    elif use_sim == eos_sim:
        driver = AsyncEOSDriver
    elif use_sim == junos_sim:
        driver = AsyncJunosDriver

    async with driver(**dev_connect) as conn:
        # Platform drivers will auto-magically handle disabling paging for you
        output = await conn.send_command(command)
        # print(output)
```

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

The main takeaways from these results are:
- async versions far outstrip their synchronous equivalents when it comes to performance
- Before the advent of Scrapli, it was difficult for network operators to use asynchronous SSH
- Network devices make it far more difficult to work with compared to Linux-y NOS because what they offer is not a programmable shell.

While this is not a professional benchmarking article, I hope it helped the readers appreciate what a serious difference asyncio makes in performance. Networking tools can get a good leg up in performance and staying simple by moving their libraries to the async versions.

Asyncssh is the python ssh library used in Suzieq. Its successfully connected to Juniper MX, Juniper QFX, Cisco's 9K, Cumulus, Arista and SONIC machines without a problem. We use textfsm internally on the gathered data, if structured output is not available. Given our requirements, this was the best choice. Our choice was validated even more by how helpful the maintainer of asyncssh, Ron Frederick is. I needed help with figuring something out, and he sent me an excellent, detailed and thoughtful response. What more could a developer or user ask for?


Update: I made minor edits after publishing to improve readability and fix formatting errors.

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.