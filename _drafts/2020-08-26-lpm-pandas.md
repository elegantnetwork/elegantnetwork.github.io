---
layout: post
comments: true
author: Dinesh G Dutt
title: IP routing and Pandas
excerpt: 
description: 
---
People who dapple in IP networks know about how packet forwarding works in IP networks. The algorithm is called Longest Prefix Match (LPM) and works by selecting the entry in the routing table that best matches the destination address in the IP packet that the router is attempting to forward. I had to grapple implementing the algorithm using the popular python data analysis package, [pandas](https://pandas.pydata.org/pandas-docs/stable/user_guide/index.html). This post is a description of the evolving techniques to implement LPM in pandas. The goal obviously was to ensure that LPM completed as fast as possible on the full Internet feed.

For those who don't know how LPM works, let us use the next two short paragraphs to describe it. Any IP address consists of two parts: the network part and the host specific part. The network part is also called the subnet. The network part is often written as a combination of 2 pieces: the IP network address and the prefix length. An example of an IP subnet written this way is: 192.168.0.0/24, where the network address is 192.168.0.0 and the prefix length is 24. 24 represents the number of bits that represent the network address and the remanining bits can be used to construct the host part. IPv4 addresses are 32 bits in length. So, in the IPv4 subnet 192.168.0.0/24, 24 bits are used to represent the network part and the remaining 8 bits are used to assign to hosts. Thus, the subnet 192.168.0.0/24 can contain upto 256 hosts, though in reality, the first and last entries are used up to create a special entry called the subnet broadcast network. So, 192.168.0.1 is the first assignable entry to a host in this subnet. IPv6 behaves the same way except that it has 128 bits instead of IPv4 32 bits. 

An IP routing table, which a router looks up to decide how to forward a packet, consists of many of these network address entries. The network address entry is also called a prefix because it forms the prefix of an IP address. To select the best matching entry for an IP address, logically, the router must select all the network addresses that can contain the address in question. Thus, if the routing table contains 192.168.0.0/16, 192.168.0.0/24, 192.168.0.0/28, and 192.168.0.1/32, then all of these entries are valid entries for an IP address of 192.168.0.1, while only the first three are valid entries for the address 192.168.0.4, and only the first two are valid entries for the address 192.168.0.20. Once the valid entries are selected, to select only one amongst these, the routing logic selects the entry with the longest prefix. Thus 192.168.0.0/24 is selected over the entry 192.168.0.0/16, 192.168.0.0/28 is selected over 192.168.0.0/24, and the /32 entry wins over all of them for an IPv4 address (a /128 entry in the case of IPv6). Thus, the IP packet forwarding algorithm is called longest prefix match.

I had to implement this LPM logic in Suzieq using pandas. A full Internet routing table is 800K routes at the time of this writing. This could easily run into millions of entries with multiple routers in Suzieq's database. My first attempt was to see if I could make IP networks a first class data type in Pandas. I had hoped that would make the performance better than if I tried it by iteratively applying the logic to each entry. To those of you who don't know pandas, the basic pandas data structure Suzieq uses is called the DataFrame. This is nothing but a table with rows and columns. Thus in pandas, the routing table is just a set of rows with the columns being the vrf, the prefix, the nexthop IP address and the outgoing interface. Here is an ![example of a couple of routing entries as pandas DataFrame in Suzieq](/assets/images/2020-08-26/route-show.png).

A naive implementation would do follow a logic that looks something like this (though it has to preserve :
```
selected_entry = None
for each row in the route table
   if the given address is a subnet of the prefix
      if the prefix length of the prefix is larger than the existing prefix length
	     selected_entry = row
return selected_entry
```

But this results in a terrible performance. I chanced upon a library called [cyberpandas](https://github.com/ContinuumIO/cyberpandas) which made an IP address a basic data type in pandas. I extended this to make IP networks a basic data type in pandas. This resulted in code that looked as follows:
```python
route_df['prefix'] = route_df['prefix'].astype('ipnetwork')
result = route_df[['namespace', 'hostname', 'vrf', 'prefix']] \
         .query("prefix.ipnet.supernet_of('{}')".format(ipaddr)) \
		 .groupby(by=['namespace', 'hostname', 'vrf'])['prefix'] \
		 .max() \
		 .dropna() \
		 .reset_index()
result_df = result.merge(route_df)
return result_df
```
The third line elegantly captures the checking if the prefix contains the address, and the fifth picks the entry with the longest prefix length.

This model looks more like pandas data pipeline code ought to look, no iterating with for loops explicitly over the entire route table. It all worked well, until it ran into the full Internet routing table. Donald Sharp, one of the key maintainers of the open source routing suite, had Suzieq collect the data from a router receiving the full Internet feed and provided me a copy of this data. The second fragment of code took close to 3 minutes to perform an lpm!

Investigating the code, I determined that the time taken was caused by two things: converting 800K prefixes into the IP network data type, and then searching through the entire 800K prefixes to find the longest prefix match. pandas had a different model for iterating over all the rows that was faster than manually iterating over the rows as shown in the pseudo code of the first fragment. That was to use the apply function. This code looked as follows:
```python
match = route_df.apply(lambda x, addr: addr.subnet_of(ip_network(x['prefix'])), args=(addr, ), axis=1)
route_df['prefixlen'] = int_df.prefix.str.split('/').str[1]
rslt_df = route_df.loc[match] \
          .sort_values('prefixlen', ascending=False) \
		  .drop_duplicates(['namespace', 'hostname', 'vrf'])
return rslt_df
```

This reduced the time window from almost 3 minutes to 1 minute 40 seconds. Better, but still way too long. 

Python is not known for being particularly fast, but almost no other language has libraries such as pandas for data analysis. So, stuck with python I was. I had read that the trick to the best performance with pandas was to vectorize the operations. By vectorizing an operation, we'd be reducing it to something that another library, numpy, could perform. In our case, we'd have to reduce the longest prefix match to a set of bit operations that numpy could be used for. The longest prefix match can be reduced to:
```
convert the IP network address to a number 
construct the netmask as a number using the prefixlen
if (addr & netmask) == (prefix & netmask), then the address is a subnet of the prefix
```

In python, this ended up looking as follows:
```python
intaddr = route_df.prefix.str.split('/').str[0] \
			      .map(lambda y: int(''.join(['%02x' % int(x)
				  						      for x in y.split('.')]), 16))
netmask = route_df.prefixlen \
            .map(lambda x: (0xffffffff << (32 - x)) & 0xffffffff)
match = (ipaddr._ip & netmask) == (intaddr & netmask)
rslt = route_df.loc[match.loc[match].index] \
               .sort_values('prefixlen', ascending=False) \
	           .drop_duplicates(['namespace', 'hostname', 'vrf'])
```

Essentially, the apply line has been reduced to the first 3 lines in the code fragment above. Even though it looks like intaddr. netmask and match are operating on a single value, they're actually operating on all the rows of the route table. To someone used to standard programming techniques, this code looks a bit strange. But with this change, lpm was reduced to 2 seconds! 

Thus, we systematically reduced the lpm performance from close to 3 minutes to 2s for a full Internet routing table. Doing lpm over 6.5 million rows went from over 6 minutes to 4 seconds!

The next version of Suzieq will ship with this higher performance lpm.