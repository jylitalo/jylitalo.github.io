---
title: VirtualBox notes
date: 2015-09-14T12:00:00+02:00
categories: ["index", "vm"]
---
Random notes about VirtualBox and related topics.
<!--more-->
Problem:
Virtual machines clock keeps on drifting so much that time difference between your VM and Amazon EC2 will go out of thresholds. Once you are beyond threshold limits, EC2 will refuse all commands that require authentication.

You can check this in host OS with:
```
# VBoxManage list vms
# for VM in ...
# do
#   VBoxManage guestproperty get VM --timesync-set-threshold
# done
<b>No value set!</b></tt>
...
```

Solution:
```
# VBoxManage guestproperty set VM --timesync-set-threshold 60000
```

If you now recheck the value, it should respond with `Value: 60000`
