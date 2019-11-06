---
layout: post
title:  "Hacking Tricks: Identifying Outgoing TCP Port for Reverse Shell"
date:   2019-11-06
categories: hacking-tricks
tags: hacking-tricks network pentest
lang: en
ref: check-outgoing-ports
has-asciinema: false
---

Sometimes during the daily engagements, we access (e.g. RCE) hosts where we are unable to get a reverse shell on the standard ports due to firewall restrictions. In these cases, we have to find alternative ways to exfiltrate data (e.g. DNS) but, first of all, check if we don't have access to any port.

During a pentest, I came across this situation and as I was looking to practice my coding, I decided to implement a tool that would accept any connection that came (any port). It was useful to identify if the compromised host had any outgoing port exposed to the Internet.


### Trick


This trick is basically a python code that anwser `SYN+ACK` to any incoming `SYN` package. Using the python Scapy library, it was possible to easly develop this mechaninsm, as you can see bellow:

``` python
#!/usr/bin/env python3

#
# SYN+ACK ALL - Written by Maycon Vitali
#

from scapy.all import *
import time

def PacketCallback(pkt):
    print (
        "[{}] SYN packet received from {} on port {}. Replying with SYN+ACK...".format(
                str(time.strftime("%Y-%m-%d %H:%M:%S")),
                pkt[IP].src,
                pkt[TCP].dport
        )
    )

    # Create the IP datagram exchanging the source and the destination
    ip  = IP(
        src=pkt[IP].dst,
        dst=pkt[IP].src
    )
    
    # Create the TCP datagrana exchanging the ports annd adjusting the flags
    # and the seq/ack numbers.
    tcp = TCP(
        sport=pkt[TCP].dport,
        dport=pkt[TCP].sport,
        flags='SA',
        seq=pkt[TCP].ack,
        ack=pkt[TCP].seq+1
    )

    # Do it!
    send(ip/tcp, verbose=False)

# Start the sniffing
sniff(
    iface="eth0",
    prn=PacketCallback,
    filter="(tcp[tcpflags] & tcp-syn != 0)"
)
```

The the above code work properly, we need to avoid the Linux kernel to sennd `RST` packades when it receives a `SYN` package to a closed port. It can be done usign the following `iptables` command:

```
root@hacknroll:~# iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP
root@hacknroll:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  anywhere             anywhere             tcp flags:RST/RST
```

Having done that, just run our script to start receiving connection messages from the unknown hosts:

```
root@hacknroll:~# ./synack-all.py
[2019-11-05 14:05:12] SYN packet received from 117.xxx.xxx.160 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:41] SYN packet received from 193.xxx.xxx.46 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:41] SYN packet received from 193.xxx.xxx.46 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:48] SYN packet received from 45.xxx.xxx.87 on port 6052. Replying with SYN+ACK...
[2019-11-05 14:05:54] SYN packet received from 185.xxx.xxx.14 on port 4590. Replying with SYN+ACK...
[2019-11-05 14:06:05] SYN packet received from 14.xxx.xxx.191 on port 445. Replying with SYN+ACK...
...
```


Or we also can change the filter parameter (pcap filtering) to anwser to only a specific host or network:

``` python
sniff(
    iface="eth0",
    prn=PacketCallback,
    filter="(tcp[tcpflags] & tcp-syn != 0) and (src net 192.111.0.0 mask 255.255.0.0)"
)
```


To verify if the host you have access has an specific outgoing port open (e.g. 80/tcp) to the internet, you can use the following command:

```
$ ncat -z -v 68.xxx.xxx.181 80
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 68.xxx.xxx.181:80.
Ncat: 0 bytes sent, 0 bytes received in 0.24 seconds.
```

And with little effort, you can scan all ports and validate which ports can go out to the Internet and then use them to get your desired reverse shell:

``` bash
$ for PORT in $(seq 65535); do ncat -z -v 68.xxx.xxx.181 $PORT 2>&1 | grep Connected; done
Ncat: Connected to 68.xxx.xxx.181:80.
Ncat: Connected to 68.xxx.xxx.181:443.
Ncat: Connected to 68.xxx.xxx.181:3389.
Ncat: Connected to 68.xxx.xxx.181:8080.
```

In the above example, the TCP ports 80, 443, 3389 (rdp) and 8080 have outgoing access to the internet.

If you wanna increase the scan speed and avoid some firewall/ips rules, you can use the following trick provided by [@usscastro](https://twitter.com/usscastro?lang=pt):
``` bash
$ seq 65535 | shuf | xargs -P32 -I@ sh -c "ncat -z -v 68.xxx.xxx.181 @" 2>&1 | grep Connected
Ncat: Connected to 68.xxx.xxx.181:3389.
Ncat: Connected to 68.xxx.xxx.181:443.
Ncat: Connected to 68.xxx.xxx.181:80.
Ncat: Connected to 68.xxx.xxx.181:8080.

```

### Conclusionn

That's all folks!


Hack N' Roll

### Referências
- [Scapy Project](https://scapy.net)