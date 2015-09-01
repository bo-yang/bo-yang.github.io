---
layout: post
title: A Complete Guide For Installing TFTP Server In CentOS 7
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

Since CentOS 7(or RedHat 7) is quite different from CentOS 6.x, most notes online for installing TFTP server in CentOS are obsolete already. This post not only summarizes the procedure of installing & configuring TFTP server, but also introduces a general strategy of configuring network services in CentOS 7.

### 1. Install `tftp-server`

TFTP server can be installed using following command, where `xinetd` is necessary. 

    yum install tftp tftp-server* xinetd*
    
Then edit `/etc/xinetd.d/tftp` - set `disable` to `no` and add `-c` option into `server_args` if you need to upload files to TFTP server from client. 

    service tftp
    {
    	socket_type		= dgram
    	protocol		= udp
    	wait			= yes
    	user			= root
    	server			= /usr/sbin/in.tftpd
    	server_args		= -c -s /tftpboot
    	disable			= no
    	per_source		= 11
    	cps			    = 100 2
    	flags			= IPv4
    }

### 2. Enable TFTP Service

The CentOS 7 services(`systemd`) can be configured from files under `/usr/lib/systemd/system/`. Go to this dir, and edit `tftp.service` as follows:

    [root@localhost system]# cat tftp.service
    [Unit]
    Description=Tftp Server
    
    [Service]
    ExecStart=/usr/sbin/in.tftpd -c -s /tftpboot
    StandardInput=socket
    
    [Install]
    WantedBy=multi-user.target

The default `tftp.service` doesn't have the `[Install]` unit, but it's required by `systemd`. Besides, the tftpd options also need to be changed in the `ExecStart` entry.

Although `service` commands are deprecated in CentOS 7, they are still available but simply redirected to `systemctl`. So you still can use `service xinetd start` and `service tftp start` to start `xinetd` and TFTP.

However, to make them automatically start after boot, following commands are needed:

    [root@localhost system]# systemctl enable xinetd
    [root@localhost system]# systemctl enable tftp

After these two commands, permanent links will be made for  `xinetd` and TFTP services.

### 3. Configure SELinux

In CentOS 7, the SELinux is not supposed to be disabled(the system will abort booting if you disable SELinux). So the TFTP read and write must be allowed in SELinux. By default, the SELinux uses `enforcing` policy, which does not accept any change. To make any change to SELinux, first modify `/etc/selinux/config` and change the policy to `permissive`:

    [bo@ucs-c200 notes]$ cat /etc/selinux/config 
    
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=permissive
    # SELINUXTYPE= can take one of three two values:
    #     targeted - Targeted processes are protected,
    #     minimum - Modification of targeted policy. Only selected processes are protected. 
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted 

Then reboot the system. After system boot up, check SELinux status:

    [root@localhost system]# sestatus
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   permissive
    Mode from config file:          permissive
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Max kernel policy version:      28

Then check the tftp permissions in SELinux:

    [root@localhost bobyan]# getsebool -a | grep tftp
    tftp_anon_write --> off
    tftp_home_dir --> off

If the TFTP write is off as shown above, enable it with `setsebool` command:

    [root@localhost bobyan]# setsebool -P tftp_anon_write 1
    [root@localhost bobyan]# setsebool -P tftp_home_dir 1

Above changes to SELinux are permanent, so no need to change any SELinux config files any more.

### 4. Configure `firewalld`

Unlike CentOS 6.x, the `firewalld` is used to replace `iptables` as default firewall in CentOS 7. Fortunately, `iptable` config file `/etc/sysconfig/iptables` is also used by `firewalld`. So to allow TFTP services, following line should be added to `/etc/sysconfig/iptables`

    -A INPUT -m state --state NEW -m udp -p udp -m udp --dport 69 -j ACCEPT

Then restart `firewalld` using command `firewall-cmd --reload`.

A more standard way to allow TFTP is to use `firewall-cmd` command:

    firewall-cmd --zone=public --add-service=tftp --permanent

Where the `--permanent` option is used to permanently enable the TFTP port. Command `firewall-cmd --reload` is needed every time changing the firewall config. 

To check the status or enable `firewalld`, following commands can be used:
    
    systemctl status firewalld
    systemctl enable firewalld
    systemctl start firewalld

