---
layout: post
title: "Notes about IPTables"
status: publish
date: 2015-09-14 12:00
---
Random notes about iptables, ufw and such tools in Linux.

<h3>Mandatory lines for NAT instance</h3>

Your /etc/sysconfig/iptables should have following lines:
<pre>
-A POSTROUTING -s <i>PRIVATE_IP_RANGE</i>/16 -o eth0 -j MASQUERADE
</pre>

<h3>Logging connections that use forwarding</h3>

If you are forwarding TCP connections from NAT instance to some instance that doesn't have public IP, you can easily add logging into iptables. Log entries will appear in /var/log/messages.
<p/>
Config in /etc/sysconfig/iptables:
<pre>
-A PREROUTING -m state --state NEW -d <i>IP_AT_NAT</i>/32 -p tcp -m tcp --dport 443 -j LOG
--log-prefix " [>] New 443 Forward"
-A PREROUTING -d <i>IP_AT_NAT</i>/32 -p tcp -m tcp --dport 443 -j DNAT --to-destination <i>IP_AT_DESTINATION</i>:443
</pre>

Lines in /var/log/messages:
<pre>
<i>TIMESTAMP HOSTNAME</i> kernel: [<i>...</i>]  [>] New 443 ForwardIN=eth0 OUT= MAC=<i>MAC_ADDRESS</i> SRC=<i>PUBLIC_IP</i> DST=<i>IP_AT_NAT</i> LEN=60 TOS=0x00 PREC=0x00 TTL=47 ID=8304 DF PROTO=TCP SPT=38660 DPT=443 WINDOW=29200 RES=0x00 SYN URGP=0
</pre>
