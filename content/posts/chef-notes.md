---
title: "Notes about Chef"
date: 2015-09-14T12:00:00+02:00
categories: ["index", "chef", "ruby"]
---
Random notes about chef configuration management tool and related topics.
<!--more-->
Problem with `knife bootstrap`:

```
[root@localhost chef-repo]# knife bootstrap --run-list "role[phpapp]" --json-attributes "{\"phpapp\": {\"server_name\": \"localhost.localdomain\"}}" --ssh-user root --ssh-password PASSWORD HOSTNAME<br/>
Node&nbsp;&nbsp;exists, overwrite it? (Y/N) y<br/>
<span style="color: red">ERROR:</span> Method Not Allowed<br/>
Response: &lt;html&gt;&lt;head>&lt;title&gt;405 Method Not Allowed&lt;/title&gt;&lt;/head&gt;&lt;body&gt;&lt;h1&gt;Method Not Allowed&lt;/h1&gt;Method Not Allowed&lt;p&gt;&lt;hr&gt;&lt;address&gt;mochiweb+webmachine web server&lt;/address&gt;&lt;/body&gt;&lt;/html&gt;
[root@localhost chef-repo]#
```

Solution:

```
[root@localhost chef-repo]# knife bootstrap <b>--node-name NODENAME</b> --run-list "role[phpapp]" --json-attributes "{\"phpapp\": {\"server_name\": \"localhost.localdomain\"}}" --ssh-user root --ssh-password PASSWORD HOSTNAME<br/>
...<br/>
10.25.4.134 Chef Client finished, X/Y resources updated in x.xx seconds<br/>
[root@localhost chef-repo]#
```

<hr/>
Problem with `knife ec2 server list`:

```
[root@localhost chef-repo]# knife ec2 server list<br/>
<span style="color: red">ERROR:</span> Fog::Compute::AWS::Error: RequestExpired => Request has expired. Timestamp date is 2015-04-13T09:54:47Z<br/>
[root@localhost chef-repo]#<br/>
```


Solution:

```
[root@localhost chef-repo]# /etc/init.d/ntpd restart<br/>
...<br/>
Starting ntpd:                                             [  <span style="color:green">OK</span>  ]<br/>
[root@localhost chef-repo]# ntpq -p<br/>
     remote           refid      st t when poll reach   delay   offset  jitter<br/>
================================================================<br/>
...<br/>
[root@localhost chef-repo]# knife ec2 server list<br/>
Instance ID  Name  Public IP  Private IP  Flavor  Image  SSH Key  Security Groups  IAM Profile  State<br/>
[root@localhost chef-repo]#
```
