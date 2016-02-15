---
layout: post
title: Debug Kernel Space Memory Leak
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

### 1. Detect Memory Leak

Memory leak can be detected by monitoring the free memory periodically. Command `free` can be used to show rough memory usage. A more detailed way to analyze memroy is `cat /proc/meminfo` and `cat /proc/slabinfo`. `/proc/meminfo` contains info about total memory, free memory, total highmem, free highmem, total lowmem, free lowmem and etc. Usually highmem is user-space memory, while lowmem is kernel-space memory. If free highmem(HighFree) or free lowmem(LowFree) is continuously decreasing, in most cases it means user space or kernel space memory leaking.

Once detected memory leaking, we need to determine which slab(s) is(are) leaking. This can be done by monitoring `/proc/slabinfo`. If the number of a slab's active objects(column 2) or total objects(column 3) keep increasing, then this slab is very likely leaking memory.

Following script can be used to monitor both meminfo and slabinfo:

    #!/bin/sh
    
    MAX_SIZE=20000000
    MON_FILE=/path/to/monitor_output_$(uname -n)
    while true
    do
        date >> $MON_FILE
        cat /proc/meminfo >> $MON_FILE
        echo "----------------------" >> $MON_FILE
        cat /proc/slabinfo >> $MON_FILE
        echo "----------------------" >> $MON_FILE
        ps -o pid,comm,stat,time,rss,vsz >> $MON_FILE
        echo "++++++++++++++++++++++" >> $MON_FILE
        fsize=`ls -l $MON_FILE | awk '{print $5}'`
        if [ $fsize -gt $MAX_SIZE ]; then
        	# upload file to TFTP server
            suffix=`cat /proc/uptime | cut -d" " -f1`
            mv $MON_FILE $MON_FILE.$suffix
            # upload to cloud
        fi

        # kmemleak
        kmemleak=/sys/kernel/debug/kmemleak
        [ -r $kmemleak ] && cat $kmemleak > /storage/kmemleak_$(uname -n).out

        sleep 300
    done

After continuously monitoring for hours, simply grep some keywords(e.g. HighFree, LowFree, kmalloc-192, etc.) could find the trend of memory usage. Following data is extracted from a real memory leak monitoring log:

    LowFree:          599392 kB
    LowFree:          571072 kB
    LowFree:          544484 kB
    LowFree:          516832 kB
    LowFree:          489232 kB
    LowFree:          462280 kB
    LowFree:          433680 kB
    LowFree:          405244 kB
    LowFree:          378572 kB
    LowFree:          350136 kB
    LowFree:          322648 kB
    LowFree:          295824 kB
    LowFree:          267532 kB
    LowFree:          238272 kB
    LowFree:          210856 kB
    LowFree:          181148 kB
    LowFree:          153652 kB
    LowFree:          123148 kB
    LowFree:          599392 kB
    LowFree:          571072 kB
    LowFree:          544484 kB
    LowFree:          181148 kB
    LowFree:          153652 kB
    LowFree:          123148 kB
    LowFree:           94548 kB
    LowFree:           94548 kB

[__update 02/14/2016__] To plot the trends of free/unreclaimed memory usage and slab allocation, a tool [memleak_plot.py](https://github.com/bo-yang/misc/blob/master/memleak_plot.py) is developed to visualize the memory leak.

### 2. Debug Memory Leak

After finding out the leaking slab, more info could be detected by tracing that slab. If the kernel was built with option `CONFIG_SLUB_DEBUG`, the simplest way is to issue command `echo 1 > /sys/kernel/slab/<leaking_slab>/trace`. Then the memory allocation trace for this slab will be printed to the console:

    [  375.201468] TRACE kmalloc-4096 alloc 0xe6be6000 inuse=8 fp=0x  (null)
    [  375.207872] Backtrace:
    [  375.210309] [<c0012378>] (dump_backtrace+0x0/0x114) from [<c03a0a5c>] (dump_stack+0x18/0x1c)
    [  375.218712]  r6:ef300480 r5:e6be6000 r4:c0d40c00 r3:c07004c4
    [  375.224367] [<c03a0a44>] (dump_stack+0x0/0x1c) from [<c03a2738>] (alloc_debug_processing+0xc8/0x164)
    [  375.233489] [<c03a2670>] (alloc_debug_processing+0x0/0x164) from [<c03a2d0c>] (__slab_alloc.isra.50.constprop.56+0x538/0x5dc)
    [  375.244767]  r7:80080008 r6:00080007 r5:e6be6000 r4:c0d40c00
    [  375.250421] [<c03a27d4>] (__slab_alloc.isra.50.constprop.56+0x0/0x5dc) from [<c00e872c>] (__kmalloc_track_caller+0xbc/0x190)
    [  375.261605] [<c00e8670>] (__kmalloc_track_caller+0x0/0x190) from [<c031f96c>] (__alloc_skb+0x58/0xf4)
    [  375.270790] [<c031f914>] (__alloc_skb+0x0/0xf4) from [<c03200d4>] (dev_alloc_skb+0x40/0x64)
    [  375.279131] [<c0320094>] (dev_alloc_skb+0x0/0x64) from [<bf6bb504>] (__adf_nbuf_alloc+0x24/0xa4 [adf])
    [  375.288409]  r4:ea5c8600 r3:00000004
    [  375.292158] [<bf6bb4e0>] (__adf_nbuf_alloc+0x0/0xa4 [adf]) from [<bf8d5f40>] (htt_rx_ring_fill_n+0x34/0x108 [umac])
    [  375.302405]  r7:00000000 r6:000005b1 r5:ea5c8720 r4:ea5c8600
    [  375.308372] [<bf8d5f0c>] (htt_rx_ring_fill_n+0x0/0x108 [umac]) from [<bf8d6888>] (htt_rx_msdu_buff_replenish+0x54/0x6c [umac])
    [  375.319400]  r8:bf927c04 r7:eaaee9c0 r6:eb5b3c00 r5:ea5c8720 r4:ea5c8600
    [  375.326429] [<bf8d6834>] (htt_rx_msdu_buff_replenish+0x0/0x6c [umac]) from [<bf8c6b24>] (ol_rx_indication_handler+0x7bc/0x8cc [umac])
    [  375.338081]  r5:ea5c8600 r4:00000000
    [  375.341955] [<bf8c6368>] (ol_rx_indication_handler+0x0/0x8cc [umac]) from [<bf8d770c>] (htt_t2h_msg_handler_fast+0xac/0x280 [umac])
    [  375.353764] [<bf8d7660>] (htt_t2h_msg_handler_fast+0x0/0x280 [umac]) from [<bf8c02dc>] (CE_per_engine_service_each+0x178/0x4b4 [umac])
    [  375.365823] [<bf8c0164>] (CE_per_engine_service_each+0x0/0x4b4 [umac]) from [<bf8c3634>] (ath_tasklet+0x68/0x128 [umac])
    [  375.376507] [<bf8c35cc>] (ath_tasklet+0x0/0x128 [umac]) from [<c0064478>] (tasklet_action+0xa0/0x11c)
    [  375.385567]  r6:e8b88000 r5:c435ef44 r4:c435ef40
    [  375.390159] [<c00643d8>] (tasklet_action+0x0/0x11c) from [<c006495c>] (__do_softirq+0x140/0x34c)
    [  375.398937] [<c006481c>] (__do_softirq+0x0/0x34c) from [<c0064d38>] (do_softirq+0x4c/0x58)
    [  375.407185] [<c0064cec>] (do_softirq+0x0/0x58) from [<c0064dd0>] (local_bh_enable_ip+0x8c/0xcc)
    [  375.415870]  r4:e8b88000 r3:0000004a
    [  375.419431] [<c0064d44>] (local_bh_enable_ip+0x0/0xcc) from [<c03aa68c>] (_raw_spin_unlock_bh+0x54/0x58)
    [  375.428865]  r5:00000304 r4:e9662c00
    [  375.432427] [<c03aa638>] (_raw_spin_unlock_bh+0x0/0x58) from [<c038a47c>] (packet_poll+0xa4/0xe4)
    [  375.441299] [<c038a3d8>] (packet_poll+0x0/0xe4) from [<c0316e34>] (sock_poll+0x24/0x28)
    [  375.449265]  r7:ea9ffe40 r6:00000000 r5:e8b89c4c r4:e8b89c04
    [  375.454920] [<c0316e10>] (sock_poll+0x0/0x28) from [<c00fc914>] (do_sys_poll+0x20c/0x3e8)
    [  375.463074] [<c00fc708>] (do_sys_poll+0x0/0x3e8) from [<c00fcbb0>] (sys_poll+0x64/0xd0)
    [  375.471071] [<c00fcb4c>] (sys_poll+0x0/0xd0) from [<c000e7c0>] (ret_fast_syscall+0x0/0x30)
    [  375.479318]  r6:0007a120 r5:00000000 r4:6b3f60a0

The first line could only be "TRACE kmalloc-4096 alloc" or "free", which logs the entry address of this slab. So if the memory leak is very fast, it is possible to monitor all of the alloc/free slabs before the system out of memory. Then find out addresses that never freed, analyze the call traces, and hopefull we could detect the problematic module or functions.

### 3. `kmemleak`

[`kmemleak`](https://www.kernel.org/doc/Documentation/kmemleak.txt) is a kernel space package to detect potential memory leak and periodically print the call stack. To build it into kernel, `CONFIG_DEBUG_KMEMLEAK` in "Kernel hacking" has to be enabled. For some systems `kmemleak` may not work due to the small buffer size for early log, so you also need to set `CONFIG_DEBUG_KMEMLEAK_EARLY_LOG_SIZE` to a larger number, e.g. 20000.

`kmemleak` also relies on `debugfs`. To automatically detect memory leak after boot, you need to make sure `debugfs` is mounted by default - you can add following line to `/etc/init.d/S10boot`:

~~~shell
grep -q debugfs /proc/filesystems && mount -t debugfs none /sys/kernel/debug
~~~

To check if `kmemleak` is working, `cat /sys/kernel/debug/kmemleak`. By default `kmemleak` scans memory every 10 minutes and prints the number of new unreferenced objects found. To trigger an intermediate memory scan, `echo scan > /sys/kernel/debug/kmemleak`.
