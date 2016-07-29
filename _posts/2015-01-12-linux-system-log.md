---
layout: post
title: Linux System Log
categories: 
- Notes
- Unix/Linux
tags:
- Notes
- Unix/Linux
status: publish
type: post
published: true
author: Bo Yang
---

This is a note of Linux system logging mechanism.

1. [Overview](#overview)
2. [printk](#printk)
3. [klogd](#klogd)
4. [syslog](#syslog)
5. [dmesg](#dmesg)
6. [syslog-ng](#syslog-ng)
7. [Convert Timestamp](#convert_timestamp)

### 1. <a name="overview">Overview</a>

Linux adopts a ring buffer in kernel with a size of `__LOG_BUF_LEN` bytes to store system logs, where `__LOG_BUF_LEN` equals (`1 << CONFIG_LOG_BUF_SHIFT`) (see `kernel/printk.c` for details). Using a ring buffer implies that older messages get overwritten once the buffer fills up, but this is only a minor drawback compared to the robustness of this solution (i.e. minimum memory footprint, callable from every context, not many resources wasted if nobody reads the buffer, no filling up of disk space/ram when some kernel process goes wild and spams the buffer, ...). Using a reasonably large buffer size should give you enough time to read your important messages before they are overwritten.

The kernel log buffer is accessible for reading from userspace by `/proc/kmsg`. `/proc/kmsg` behaves more or less like a FIFO and blocks until new messages appear. Please note, reading from `/proc/kmsg` consumes the messages in the ring buffer so they may not be available for other programs. It is usually a good idea to let `klogd` or `syslog` do this job and read the content of the buffer via dmesg.

![Linux Kernel Log]({{ site.url }}/assets/images/2015-01-12-linux-system-log/linux_kernel_log.png)

### 2. <a name="printk">`printk`</a>

`printk` is the kernel function to classify messages according to their severity by loglevels and write them to the circular system message buffer. The function then wakes any process that is waiting for messages, that is, any process that is sleeping in the `syslog` system call or that is reading `/proc/kmsg`. `printk` can be invoked from anywhere, even from an interrupt handler, with no limit on how much data can be printed.

```c
printk( KERN_CRIT "Error code %08x.\n", val );
```

There are eight possible loglevel strings, defined in the header <linux/kernel.h>; we list them in order of decreasing severity:

Name | String | Meaning | alias function
---- | ------ | ------- | --------------
KERN_EMERG | "0" | Emergency messages, system is about to crash or is unstable | pr_emerg
KERN_ALERT | "1" | Something bad happened and action must be taken immediately | pr_alert
KERN_CRIT | "2" | A critical condition occurred like a serious hardware/software failure | pr_crit
KERN_ERR | "3" | An error condition, often used by drivers to indicate difficulties with the hardware | pr_err
KERN_WARNING | "4" | A warning, meaning nothing serious by itself but might indicate problems | pr_warning
KERN_NOTICE | "5" | Nothing serious, but notably nevertheless. Often used to report security events. | pr_notice
KERN_INFO | "6" | Informational message e.g. startup information at driver initialization | pr_info
KERN_DEBUG | "7" | Debug messages | pr_debug, pr_devel if DEBUG is defined
KERN_DEFAULT | "d" | The default kernel loglevel |
KERN_CONT | "" | "continued" line of log printout (only done after a line that had no enclosing `\n`) |pr_cont


Each string (in the macro expansion) represents an integer in angle brackets. Integers range from 0 to 7, with smaller values representing higher priorities.

A `printk` statement with no specified priority defaults to `DEFAULT_MESSAGE_LOGLEVEL`, specified in `kernel/printk.c` as an integer. For this the kernel compares the log level of the message to the `console_loglevel` (a kernel variable) and if the priority is higher (i.e. a lower value) than the console_loglevel the message will be printed to the current console. The `console_loglevel` can be checked by

    # cat /proc/sys/kernel/printk
    7       4       1       7

The first integer shows you your current `console_loglevel`; the second is the `DEFAULT_MESSAGE_LOGLEVEL`.

Kernel log timestamp is added by `vprintk()`, in `kernel/printk.c`:

```c
#if defined(CONFIG_PRINTK_TIME)
static bool printk_time = 1;
#else
static bool printk_time = 0;
#endif

if (printk_time) {
		/* Add the current time stamp */
		char tbuf[50], *tp;
		unsigned tlen;
		unsigned long long t;
		unsigned long nanosec_rem;

		t = cpu_clock(printk_cpu);
		nanosec_rem = do_div(t, 1000000000);
		tlen = sprintf(tbuf, "[%5lu.%06lu] ",
				(unsigned long) t,
				nanosec_rem / 1000);

		for (tp = tbuf; tp < tbuf + tlen; tp++)
			emit_log_char(*tp);
		printed_len += tlen;
}
```

### 3. <a name="klogd">`klogd`</a>

**If the `klogd` process is running, it retrieves kernel messages and dispatches them to `syslogd`, which in turn checks `/etc/syslog.conf` to find out how to deal with them.** `syslogd` differentiates between messages according to a facility and a priority; allowable values for both the facility and the priority are defined in `<sys/syslog.h>`. Kernel messages are logged by the `LOG_KERN` facility at a priority corresponding to the one used in `printk` (for example, `LOG_ERR` is used for `KERN_ERR` messages). **If `klogd` isn't running, data remains in the circular buffer until someone reads it or the buffer overflows.**

If you want to avoid clobbering your system log with the monitoring messages from your driver, you can either specify the (file) option to `klogd` to instruct it to save messages to a specific file, or customize `/etc/syslog.conf` to suit your needs. Yet another possibility is to take the brute-force approach: kill `klogd` and verbosely print messages on an unused virtual terminal, or issue the command `cat /proc/kmsg` from an unused xterm.


### 4. <a name="syslog">`syslog`</a>

Accessing to the log buffer is provided at the core through the multi-purpose `syslog` system call. The prototype for the `syslog system` call is defined in `./linux/include/linux/syslog.h`; its implementation is in `./linux/kernel/printk.c`.

The `syslog` call serves as the input/output (I/O) and control interface to the kernel's log message ring buffer. From the syslog call, an application can read log messages (partial, in their entirety, or only new messages) as well as control the behavior of the ring buffer (clear contents, set the level of messages to be logged, enable or disable console, and so on).

Although reading from `/proc/kmsg` consumes the data from the log buffer, the `syslog` system call can optionally return log data while leaving it for other processes as well.

Kernel space syslog API:

```c
#include <syslog.h>

void openlog(const char *ident, int option, int facility);
void syslog(int priority, const char *format, ...);
void closelog(void);

#include <stdarg.h>

void vsyslog(int priority, const char *format, va_list ap);
```

- `closelog()` closes the descriptor being used to write to the system logger. 
- `openlog()` opens a connection to the system logger for a program.
- `syslog()` generates a log message, which will be distributed by `syslogd`. It does this by writing to the Unix domain socket `/dev/log`.
- `vsyslog()` is functionally identical to `syslog()`, with the BSD style variable length argument.

User space syslog API(glibc wrapper):

    int syslog( int type, char *bufp, int len );
    int klogctl( int type, char *bufp, int len );

`klogctl()` is the glibc wrapper to control the kernel `printk()` buffer. 

### 5. <a name="dmesg">`dmesg`</a>

The `dmesg` command is used to print and control the kernel ring buffer. This command uses the `klogctl` system call to read the kernel ring buffer and emit it to standard output (stdout). The command can also be used to clear the kernel ring buffer (using the `-c` option), set the level for logging to the console (the `-n` option), and define the size of the buffer used to read the kernel log messages (the `-s` option).

`dmesg` reads by default a buffer of max 16392 bytes, so if you use a larger log buffer you have to invoke `dmesg` with the `-s` parameter e.g.:

    ### CONFIG_LOG_BUF_SHIFT 17 = 128k
    $ dmesg -s 128000

### 6. <a name="syslog-ng">`syslog-ng`</a>

The `syslog-ng` application is a flexible and highly scalable system logging application that is ideal for creating centralized and trusted logging solutions. It extends the original syslogd model with content-based filtering, rich filtering capabilities, flexible configuration options and adds important features to syslog. 

`syslog-ng` also supports ISO/RFC timestamp for system logs. For more info about this powerful log system, please refer to [the manual](http://www.balabit.com/sites/default/files/documents/syslog-ng-ose-latest-guides/en/syslog-ng-ose-guide-admin/html-single/index.html).

### 7. <a name="convert_timestamp">Converting Timestamps</a>

By default the time stamps are printed in "seconds since boot" (this is the way the kernel is programmed to print the time stamps in `vprintk()`, and it can not be changed to print the time stamps in a human readable format). The system uptime can be helpful to calculate an absolute time stamp if needed (run the `uptime` command).

Given timestamp:

    [196149.728085] hello world

The algorithm below converts the printed time stamps to a human readable format:

    1. Take the log's time stamp in seconds: 
    
    196149.728085 seconds (round the number down/up if needed) 
    
    
    2. Divide the time stamp in seconds by 60 to get the total amount of minutes: 
    
    196150 : 60 = 3269.1667 minutes (round the number down/up if needed) 
    
    
    3. Divide the time stamp in minutes by 60 to get the total amount of hours: 
    
    3269.1667 : 60 = 54.486111666666666666666666666667 hours 
    
    
    4. Break the decimal number into 2 parts: 
    
    54.486111666666666666666666666667 hours = (54 hours) + (0.486111666666666666666666666667 decimal hours) 
    
    
    5. Use the time conversion charts below to convert decimal hours to minutes: 
    
    0.486111666666666666666666666667 decimal hours ~ 0.48 decimal hours ~ 29 minutes 
    
    6. Note: For more precise conversion (down to seconds), you can use various time converters available on the Internet. Just search for 'convert decimal time' in any search engine. 
    
    
    7. Hence, we get that the log was created this amount of time since boot: 
    
    196149.728085 seconds ~ 54 hours 29 minutes 
    
    8. Check the current system's uptime: 
    
    [Expert@HostName]# uptime 
    
    9. To get the log's real time stamp, subtract the log's time stamp in dmesg kernel ring buffer from the current system's uptime.

Following is a Shell script to transform uptime timestamp to human-readable timestamp:

```sh
#!/bin/bash
# Translate dmesg timestamps to human readable format

# desired date format
date_format="%a %b %d %T %Y"

# uptime in seconds
uptime=$(cut -d " " -f 1 /proc/uptime)

# run only if timestamps are enabled
if [ "Y" = "$(cat /sys/module/printk/parameters/time)" ]; then
  dmesg | sed "s/^\[[ ]*\?\([0-9.]*\)\] \(.*\)/\\1 \\2/" | while read timestamp message; do
    #date +"%s" -d "1970-01-01 00:00:00"
    #awk '{printf("%d:%02d:%02d\n", ($1/3600), ($1%3600/60),($1%60))}' /proc/uptime
    printf "[%s] %s\n" "$(date --date "now - $uptime seconds + $timestamp seconds" +"${date_format}")" "$message"
  done
else
  echo "Timestamps are disabled (/sys/module/printk/parameters/time)"
fi
```

### References
- [http://elinux.org/Debugging_by_printing](http://elinux.org/Debugging_by_printing)
- Linux Device Drivers, Corbet, Rubini and Kroah-Hartmann, 3rd Edition, Chapter 4 Section 2
- [Kernel logging: APIs and implementation](http://www.ibm.com/developerworks/library/l-kernel-logging-apis/index.html)
- [http://unixhelp.ed.ac.uk/CGI/man-cgi?dmesg+8](http://unixhelp.ed.ac.uk/CGI/man-cgi?dmesg+8)
- [https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk92677](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk92677)
- [http://www.balabit.com/sites/default/files/documents/syslog-ng-ose-latest-guides/en/syslog-ng-ose-guide-admin/html-single/index.html](http://www.balabit.com/sites/default/files/documents/syslog-ng-ose-latest-guides/en/syslog-ng-ose-guide-admin/html-single/index.html)
