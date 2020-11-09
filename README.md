[![Build Status](https://travis-ci.org/farrokhi/dnsdiag.svg)](https://travis-ci.org/farrokhi/dnsdiag) [![PyPI](https://img.shields.io/pypi/v/dnsdiag.svg?maxAge=8600)](https://pypi.python.org/pypi/dnsdiag/) [![PyPI](https://img.shields.io/pypi/l/dnsdiag.svg?maxAge=8600)]() [![PyPI](https://img.shields.io/pypi/pyversions/dnsdiag.svg?maxAge=8600)]() [![GitHub stars](https://img.shields.io/github/stars/farrokhi/dnsdiag.svg?style=social&label=Star&maxAge=8600)](https://github.com/farrokhi/dnsdiag/stargazers)

DNS Diagnostics and Performance Measurement Tools
==================================================

Ever been wondering if your ISP is [hijacking your DNS traffic](https://medium.com/decentralize-today/is-your-isp-hijacking-your-dns-traffic-f3eb7ccb0ee7)? Ever observed any
misbehavior with your DNS responses? Ever been redirected to wrong address and
suspected something is wrong with your DNS? Here we have a [set of tools](http://github.com/farrokhi/dnsdiag) to
perform basic audits on your DNS requests and responses to make sure your DNS is
working as you expect.

You can measure the response time of any given DNS server for arbitrary requests
using `dnsping`. Just like traditional ping utility, it gives you similar
functionality for DNS requests.

You can also trace the path your DNS request takes to destination to make sure
it is not being redirected or hijacked. This can be done by comparing different
DNS queries being sent to the same DNS server using `dnstraceroute` and observe
if there is any difference between the path.

`dnseval` evaluates multiple DNS resolvers and helps you choose the best DNS
server for your network. While it is highly recommended to use your own DNS
resolver and never trust any third-party DNS server, but in case you need to
choose the best DNS forwarder for your network, `dnseval` lets you compare
different DNS servers from performance (latency) and reliability (loss) point
of view.

# prerequisites
This script requires python3 as well as latest
[dnspython](http://www.dnspython.org/) and
[cymruwhois](https://pythonhosted.org/cymruwhois/).

# installation

There are several ways that you can use this toolset. However using the sourcecode is always recommended.

## From Source Code

1. You can checkout this git repo and its submodules

```
git clone https://github.com/Martas73/dnsdiag.git
cd dnsdiag
pip3 install -r requirements.txt
```

2. You can alternatively install the package using pip:

```
pip3 install dnsdiag
```

## From Binary

From time to time, binary version will be released for Windows, Mac OS X and Linux platforms. You can grab the latest release from [releases page](https://github.com/Martas73/dnsdiag/releases).

# dnsping
dnsping pings a DNS resolver by sending an arbitrary DNS query for given number
of times:
```
% ./dnsping.py -c 3 -t AAAA -s 8.8.8.8 dnsdiag.org
dnsping.py DNS: 8.8.8.8:53, hostname: dnsdiag.org, rdatatype: AAAA
4 bytes from 8.8.8.8: seq=0   time=123.509 ms
4 bytes from 8.8.8.8: seq=1   time=115.726 ms
4 bytes from 8.8.8.8: seq=2   time=117.351 ms

--- 8.8.8.8 dnsping statistics ---
3 requests transmitted, 3 responses received,   0% lost
min=115.726 ms, avg=118.862 ms, max=123.509 ms, stddev=4.105 ms
```
This script calculates minimum, maximum and average response time as well as
jitter (stddev)

# dnstraceroute
dnstraceroute is a traceroute utility to figure out the path that your DNS
request is passing through to get to its destination. You may want to compare
it to your actual network traceroute and make sure your DNS traffic is not
routed to any unwanted path.

```
% ./dnstraceroute.py --expert -C -t A -s 8.8.4.4 facebook.com
dnstraceroute.py DNS: 8.8.4.4:53, hostname: facebook.com, rdatatype: A
1	192.168.0.1 (192.168.0.1) 1 ms
2	192.168.28.177 (192.168.28.177) 4 ms
3	192.168.0.1 (192.168.0.1) 693 ms
4	172.19.4.17 (172.19.4.17) 3 ms
5	google-public-dns-b.google.com (8.8.4.4) 8 ms

=== Expert Hints ===
 [*] public DNS server is next to a private IP address (possible hijacking)
```

Using `--expert` will instruct dnstraceroute to print expert hints (such as warnings of possible DNS traffic hijacking).

# dnseval
dnseval is a bulk ping utility that sends an arbitrary DNS query to a give list
of DNS servers. This script is meant for comparing response time of multiple
DNS servers at once:
```
% ./dnseval.py -t AAAA -f public-servers.txt -c10 yahoo.com
server           avg(ms)     min(ms)     max(ms)     stddev(ms)  lost(%)  ttl     flags
------------------------------------------------------------------------------------------------------
8.8.8.8          270.791     215.599     307.498     40.630      %0       298     QR -- -- RD RA -- --
8.8.4.4          222.955     171.753     307.251     60.481      %10      291     QR -- -- RD RA -- --
ns.ripe.net      174.855     160.949     187.458     10.099      %0       289     QR -- -- RD RA -- --
4.2.2.1          172.798     163.892     189.918     7.823       %0       287     QR -- -- RD RA -- --
4.2.2.2          178.594     169.158     184.696     5.067       %0       285     QR -- -- RD RA -- --
4.2.2.3          153.574     138.509     173.439     12.015      %0       284     QR -- -- RD RA -- --
4.2.2.4          153.182     141.023     162.323     6.700       %0       282     QR -- -- RD RA -- --
4.2.2.5          154.840     141.557     163.889     7.195       %0       281     QR -- -- RD RA -- --
209.244.0.3      156.270     147.320     161.365     3.958       %0       279     QR -- -- RD RA -- --
209.244.0.4      159.329     151.283     163.726     3.958       %0       278     QR -- -- RD RA -- --
195.46.39.39     171.098     163.612     181.147     5.067       %0       276     QR -- -- RD RA -- --
195.46.39.40     175.335     160.920     185.618     8.726       %0       274     QR -- -- RD RA -- --
```
```
# dnseval.py -t A -f dns-list.txt -c10 seznam.cz
server              avg(ms)     min(ms)     max(ms)     stddev(ms)  lost(%)  ttl        flags
------------------------------------------------------------------------------------------------------------
8.8.8.8             7.878       6.964       9.522       1.013       %0       226        QR -- -- RD RA -- --
8.8.4.4             9.502       7.004       16.042      3.412       %0       225        QR -- -- RD RA -- --
1.1.1.1             14.958      13.397      16.360      1.029       %0       170        QR -- -- RD RA -- --
9.9.9.9             14.662      13.006      16.010      0.944       %0       280        QR -- -- RD RA -- --
208.67.222.222      36.723      35.928      38.531      0.817       %0       280        QR -- -- RD RA -- --
208.67.220.220      37.297      35.687      40.189      1.488       %0       86         QR -- -- RD RA -- --
199.85.126.20       17.996      15.572      19.123      1.226       %0       279        QR -- -- RD RA -- --
185.228.168.168     16.668      15.671      18.287      0.916       %0       280        QR -- -- RD RA -- --
77.88.8.7           100.427     40.359      172.630     42.922      %0       280        QR -- -- RD RA -- --
8.26.56.26          40.032      37.646      49.245      3.857       %20      281        QR -- -- RD RA -- --
217.31.204.130      8.731       8.004       10.083      0.831       %0       140        QR -- -- RD RA -- --
193.29.206.206      19.888      18.010      22.150      1.245       %0       281        QR -- -- RD RA -- --
213.46.172.36       8.485       7.115       10.012      0.897       %0       246        QR -- -- RD RA -- --
213.46.172.37       8.261       7.002       12.007      1.459       %0       232        QR -- -- RD RA -- --
```

### Author

Babak Farrokhi 

- twitter: [@farrokhi](https://twitter.com/farrokhi)
- github: [github.com/farrokhi](https://github.com/farrokhi/)
- website: [farrokhi.net](https://farrokhi.net/)


### License

dnsdiag is released under a 2 clause BSD license.

