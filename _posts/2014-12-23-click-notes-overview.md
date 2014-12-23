---
layout: post
title: Click Notes I: Overview
categories: 
- Unix/Linux
- Notes
tags:
- Unix/Linux
- Notes
status: draft
type: post
published: false
author: Bo Yang
---

Click is a modular router toolkit, which can be run in both user space and OS kernel space. Since its invention in late 1990s by Eddie Kohler, Click is has gained great successes in both research and industry. This series of notes aims to (1)introduce Click platform, (2)analyze the implementation of Click, and (3) discuss some general problems related to Operating System and Network Programming.

### Router Abstraction

Routers are pure packet processors: packets arrive on one network interface, travel through the packet processor’s forwarding path, and are emitted on another network interface. Forwarding paths behave like pipelines through which packets flow. Packets lose their identity after they arrive - only data is transferred to applications. In packet processors, packets move horizontally between peers, not vertically between application layers. 

Firewall limit access to a protected network, often by dropping inappropriate packets.

Network address translators allow a large set of machines to share a single public IP address; they work by rewriting packet headers, and occasionally some data.

Load balancers send packets to one of a set of servers, dynamically choosing a server based on load or some application characteristic.

### Click Router Architecture

To build a router configuration, the user chooses a collection of elements and connects them into a directed graph. A Click router is assembled from packet processing modules called elements. Individual elements implement simple router functions like packet classification, queueing, scheduling, and interfacing with network devices. Configurations are written in a declarative language that supports user-defined abstractions. Due to Click’s architecture and language, Click router configurations are modular and  easy to extend.

The Click architecture is centered on the element. Each element is a software component representing a unit of router processing, which generally examines or modifies packets in some way. At run time, elements pass packets to one another over links called connections. Each connection represents a possible path for packet transfer. A router configuration is a directed graph with elements at the vertices and connections as the edges.  Router configurations, run in the context of some driver, either at user level or in the Linux kernel. 

### References

- [Eddie Kohler, _The Click modular router_, Ph.D. thesis, MIT, November 2000. This has more detail and examples than the TOCS and SOSP papers of the same name.](http://pdos.csail.mit.edu/papers/click:kohler-phd/thesis.pdf)
- [http://read.cs.ucla.edu/click/](http://read.cs.ucla.edu/click/)
- [http://www.pats.ua.ac.be/software/click/](http://www.pats.ua.ac.be/software/click/)
