---
layout: post
title: How To Use Local Facilities For Logging?
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

This post introduces how to configure and use syslogd-compatible syslog tools. These tips should be supported by rsyslog, but rsyslog-specific commands are not covered.

As documented in the man page, the Linux system log is configured in file `/etc/syslog.conf` by default.You can specify other config file with `-f` option for syslogd. The format of syslog config file is

    <facility>.<priority>   [logfile]

The supported facilities and priorites are defined in `syslog.h`:

    #define	LOG_EMERG	0	/* system is unusable */
    #define	LOG_ALERT	1	/* action must be taken immediately */
    #define	LOG_CRIT	2	/* critical conditions */
    #define	LOG_ERR		3	/* error conditions */
    #define	LOG_WARNING	4	/* warning conditions */
    #define	LOG_NOTICE	5	/* normal but significant condition */
    #define	LOG_INFO	6	/* informational */
    #define	LOG_DEBUG	7	/* debug-level messages */
    
    /* facility codes */
    #define	LOG_KERN	(0<<3)	/* kernel messages */
    #define	LOG_USER	(1<<3)	/* random user-level messages */
    #define	LOG_MAIL	(2<<3)	/* mail system */
    #define	LOG_DAEMON	(3<<3)	/* system daemons */
    #define	LOG_AUTH	(4<<3)	/* security/authorization messages */
    #define	LOG_SYSLOG	(5<<3)	/* messages generated internally by syslogd */
    #define	LOG_LPR		(6<<3)	/* line printer subsystem */
    #define	LOG_NEWS	(7<<3)	/* network news subsystem */
    #define	LOG_UUCP	(8<<3)	/* UUCP subsystem */
    #define	LOG_CRON	(9<<3)	/* clock daemon */
    #define	LOG_AUTHPRIV	(10<<3)	/* security/authorization messages (private) */
    #define	LOG_FTP		(11<<3)	/* ftp daemon */
    
    	/* other codes through 15 reserved for system use */
    #define	LOG_LOCAL0	(16<<3)	/* reserved for local use */
    #define	LOG_LOCAL1	(17<<3)	/* reserved for local use */
    #define	LOG_LOCAL2	(18<<3)	/* reserved for local use */
    #define	LOG_LOCAL3	(19<<3)	/* reserved for local use */
    #define	LOG_LOCAL4	(20<<3)	/* reserved for local use */
    #define	LOG_LOCAL5	(21<<3)	/* reserved for local use */
    #define	LOG_LOCAL6	(22<<3)	/* reserved for local use */
    #define	LOG_LOCAL7	(23<<3)	/* reserved for local use */

The local facilities can be used for redirecting/filtering the log of your own programs. For example, given a program `foo`, if you want to log all the non-critical messages in `/var/log/foo.log`, and make the critical logs go to system log file `/var/log/messages`, you can use the following config file

    # use facility local1 for capwap_brain & connection_monitor logs
    local1.debug;local1.info;local1.notice;local1.warn   -/var/log/foo.log
    local1.panic;local1.alert;local1.crit;local1.err   -/var/log/messages
    *.*;local1.none   /var/log/messages 

The special priority `none` prevents those messages from being logged even though they would have been included in the *.*. In the above config, all facilities except for `local1` will be logged to `/var/log/messages`.

However, the dash(-) in front of the log filename is not documented in the man page, but it turns out to mean "Don't sync after every write to the file". Except that rsyslogd won't sync anyway, unless you add a special directive in the Global Directives section. Note that you might lose information if the system crashes right behind a write attempt. Nevertheless this might give you back some performance, especially if you run programs that use logging in a very verbose manner. So for most people, a dash makes no difference one way or the other -- it will be ignored.

And in program `foo`, what you need to do is open log file by specifing `LOG_LOCAL1` facility. The use of `openlog()` is mandatory here. Otherwise, it will automatically be called by syslog(), in which case ident will default to `NULL`.

    #include <syslog.h>
    int main(int, char**)
    {
        openlog("foo", 0, LOG_LOCAL1);
        syslog(LOG_INFO, "test info log"); /* go to /var/log/foo.log */
        syslog(LOG_ERR, "test error log"); /* go to /var/log/messages */
        closelog();
    }


### References
- [http://linux.die.net/man/5/syslog.conf](http://linux.die.net/man/5/syslog.conf)
- [http://linux.die.net/man/3/openlog](http://linux.die.net/man/3/openlog)
- [A Brief Tutorial on rsyslog.conf](http://shallowsky.com/blog/linux/rsyslog-conf-tutorial.html)
