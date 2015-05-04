---
layout: post
title: Customizing OpenWRT System Log Timestamp
categories: 
- Notes
- Unix/Linux
- OpenWRT
tags:
- Notes
- Unix/Linux
status: publish
type: post
published: true
author: Bo Yang
---

As explained in previous post [Linux System Log](http://www.bo-yang.net/2015/01/12/linux-system-log/), threre is no way to directly change the default timestamps printed on Linux console. However, it is still possible to get human readable timestamp without using fancy syslog tools like [syslog-ng](http://www.balabit.com/sites/default/files/documents/syslog-ng-ose-latest-guides/en/syslog-ng-ose-guide-admin/html-single/index.html).

A bash script of customizing Linux dmesg timestamp is also given in my post post [Linux System Log](http://www.bo-yang.net/2015/01/12/linux-system-log/). Unfortunately, that script won't work if bash is not available in your Linux system. For example, the default shell in OpenWRT is Busybox ash, which lacks many powerful features of bash. The most significant shortcoming for ash is the lame arithmetic operations. In addition, many useful options for standard commands are not supported in Busybox ash. Fortunately, we have sed and awk installed in OpenWRT. So the arithmetic operations can be done by awk.

The basic idea of customizing console syslog timestamp is periodically calling `dmesg -c`, which clears system circular buffer after dumping the system log. In order to also store the log in syslog file(like /var/log/messages), the dumped message also needs to be redirected(or appending) to the syslog file. The timestamps of `dmesg` can be replaced with a human readable one. And the required addition/subtraction is implemented by awk in a very special way.

<strike>Instead of `dmesg`, someone may be tempted to use command `tail -f /var/log/messages`. Unfortunately, the `tail` command would automatically stop printing from `/var/log/messages` after a while. It may be caused by the implementation of Busybox `tail` or OpenWRT system log mechanism.</strike> Although `tail -f /var/log/messages` would automatically stop after some time, the `-F` option works for ash tail command, i.e. `tail -F /var/log/messages`.

Following is the source code in `ash`:

    #!/bin/ash
    
    #
    # This script is used to tail the latest syslogs to stdout,
    # syslog file and specified log file.
    #
    
    SYSLOG=/var/log/messages
    custom_log=
    [ ! -z "$1" ] && custom_log=$1
    
    base=$(cut -d" " -f1 /proc/uptime);
    ut=$(date +%s);
    # FIXME: Arithmetic operations are not fully supported in ash, use awk instead.
    base=`date | awk "{now=$ut - $base; printf \"%d\", now}"`
    
    dmesg -c >> $SYSLOG # clear circular syslog buffer
    while true
    do
        dmesg -c | sed "s/^\[[ ]*\?\([0-9.]*\)\] \(.*\)/\\1 \\2/" |
        while read ts msg; do
            now=`date | awk "{now=$base + $ts; printf \"%d\", now}"`
            newts=`date +"%m/%d/%Y %H:%M:%S" --date "@$now"` # human readable timestamp
            printf "[%s] %s\n" "$newts" "$msg";
        done | sed "s/$/$(printf '\r')/" | tee -a $SYSLOG $custom_log
    
        sleep 1
    done

