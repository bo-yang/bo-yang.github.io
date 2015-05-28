---
layout: post
title: Retrieve Last Log After Crash
categories: 
- Notes
- Unix/Linux
- OpenWRT
tags:
- Notes
- Unix/Linux
status: public
type: post
published: true
author: Bo Yang
---

In Linux, there are two kinds of crashes - kernel panic/oom and user space core dump. For kernel panic, the standard config is rebooting the system. Unfortunately, the panic log can only be printed to console and will disappear after reboot if there is no additional device to record the console log - most kernel panic/oom logs won't be written to system log, and even they do, there is no way to sync them to disk storage during panic.

As for user-sapce core dump, core files will be generated to `/tmp/pid.core`(core pattern can be changed in `/proc/sys/kernel/core_pattern`) by default. It is up to the admin to decide if the system or process needs reboot after core dump. Although a script can be used to record more logs for the coredump, sometimes it's still useful to retain some info in the persistent memory, like the backtrace of the coredumped process.

### Modules Needed

- [**phram**](http://lxr.free-electrons.com/source/drivers/mtd/devices/phram.c)

`phram` is a Memory Technology Device(MTD) driver, which supports accessing non-system memory, i.e. from the PCI address space. This module can act as a special memory file to store important data during reboots.

- [**ramoops**](https://www.kernel.org/doc/Documentation/ramoops.txt)

`ramoops` is an kernel oops/panic logger that writes its logs to predefined memory area before the system crashes. It works by logging oopses and panics in a circular buffer. As long as the device has power supply during reboot, the content stored in ramoops won't go away.

Both `phram` and `ramoops` can be compiled to `.ko` shared library, which can be configured using Linux menuconfig. 

#### Loading Modules

`phram` can be dynamically loaded using following command:

    insmod /lib/modules/phram.ko phram=<name>,<addr>,<len>

where `<name>` is the device(i.e. file) name under `/dev/mtdchar/`, `<addr>` is the reserved starting memory address, and `<len>` is the predefined length of the memory area.

To load `ramoops`

    insmod /lib/modules/ramoops.ko mem_address=<addr> mem_size=<len> [record_size=<chunks>]

 where `<addr>` and `<len>` are same as above, and `record_size` is the chunks of reserved memory area.

#### Copy Log To `phram` Memory

For kernel panic, the ramoops will automatically copy the panic log into the reserved memory. The panic log begins with leading "====" followed by a timestamp and a new line.

If you need to store other info, `dd` command can be used to copy file to the memory, e.g.

    dd if=/var/log/messages bs=1 count=65536 skip=<fsize - len> of=/dev/mtdchar/phram-oops

According to `dd` manual, `if` is the the input file stream, `of` is the output file, `bs` specifies the bytes to be read and write at a time, and `skip` configs the input blocks to be skipped from the input file. Here the size of _block_ is defined by `bs`. In the above example, the block size is set to 1 byte, and only the last `len` bytes will be copied to `phram` memory.

Unfortunately, reading & writing byte-by-byte usaully is very slow. A faster way is setting `bs` to a larger number, like 128, 512, 1024, etc. In this case, the `skip` and `count` need to be calculated correspondingly: say the size of reserved memory is `len`, input file size is `fsize`, `bs` is set to `bsize`, then

    // len: memory size
    // fsize: input file size
    bs = bsize
    if (fsize <= bsize)
        skip =  0;
    else
        skip = (fsize - len)/bsize + 1;
    count = len/bsize + 1;

#### Dump Last Log

`dd` command can also be used for dumping data from reserved phram memory area, e.g.

    dd if=/dev/mtdchar/phram-oops bs=<len> count=1

Other file operation commands/APIs can also be used for `phram` memory device.

#### Reset `phram` Memory

`phram` memory can be cleared by file operation coomands/APIS, e.g.

    dd if=/dev/zero bs=<len> count=1 of=/dev/mtdchar/phram-oops

#### References

1. [Use Memory On Video Card As Swap](http://www.gentoo-wiki.info/TIP_Use_memory_on_video_card_as_swap)
2. [Ramoops oops/panic logger](https://www.kernel.org/doc/Documentation/ramoops.txt)
3. [`dd` manual](http://linux.die.net/man/1/dd)
