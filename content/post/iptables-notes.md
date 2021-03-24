---
title: "Notes about IPTables"
date: 2015-09-14T12:00:00+02:00
categories: ["index", "linux"]
---
Random notes about iptables, ufw and such tools in Linux.
<!--more-->
## Mandatory lines for NAT instance

Your /etc/sysconfig/iptables should have following lines:
```
-A POSTROUTING -s _PRIVATE_IP_RANGE_/16 -o eth0 -j MASQUERADE
```

## Logging connections that use forwarding

If you are forwarding TCP connections from NAT instance to some instance that doesn't have public IP, you can easily add logging into iptables. Log entries will appear in /var/log/messages.

Config in /etc/sysconfig/iptables:
```
-A PREROUTING -m state --state NEW -d _IP_AT_NAT_/32 -p tcp -m tcp --dport 443 -j LOG --log-prefix " [>] New 443 Forward "
-A PREROUTING -d _IP_AT_NAT_/32 -p tcp -m tcp --dport 443 -j DNAT --to-destination _IP_AT_DESTINATION_:443
```

Lines in /var/log/messages:
> _TIMESTAMP_ _HOSTNAME_ kernel: [...]  [>] New 443 Forward IN=eth0 OUT= MAC=_MAC_ADDRESS_ SRC=_PUBLIC_IP_ DST=_IP_AT_NAT_ LEN=60 TOS=0x00 PREC=0x00 TTL=47 ID=8304 DF PROTO=TCP SPT=38660 DPT=443 WINDOW=29200 RES=0x00 SYN URGP=0
