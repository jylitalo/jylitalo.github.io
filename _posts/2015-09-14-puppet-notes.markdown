---
layout: post
title: "Notes about Puppet"
status: publish
date: 2015-09-14 12:00
---
*Puppet doesn't like symlinks*

When you start building _/etc/puppet/environments/production_ in your puppet master, don't use symlinks on directory tree. I once made the mistake of using symlinks for manifests and modules directories and was getting frustrated, when puppet agent didn't seem to get my _manifests/site.pp_ file.

Once I copied all the directly into _/etc/puppet/environments/production_ tree, things started to work.

*Use info(), notice() or debug() in site.pp to describe what role each client host has*

Many users would like to see, which role was applied to which host. My approach for this challenge has been to put notice(), info() or debug() in _node_ definitions to log this information into puppet masters log files (probably in _/var/log/messages_ or _/var/log/syslog_).

*puppet cert commands takes awfully long time*

`puppet cert` commands do hostname resolving along the way and if your DNS configuration is incorrect, it will eventually fail. This problem might appear, if you have invalid information in _/etc/resolv.conf_.

*Potential pitfalls on legacy configs*

If you are given task to transform existing puppet configuration for a new hosting provider, check this things for potential problem:

* does it overwrite _/etc/resolv.conf_?
This will mess your DNS configuration and cause all kind of issues.</li>
* does it overwrite _/etc/puppet/puppet.conf_?
Puppet works beatifully on a first run, but ruins all future runs.
