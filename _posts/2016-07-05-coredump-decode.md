---
layout: post
title: Coredump Decode
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

This is a summary of decoding Linux userspace process corefiles using GDB. For how to enable coredump in Linux, please refer to [HOWTO enable core-dumps](http://en.linuxreviews.org/HOWTO_enable_core-dumps).

1. [Preparation](#prep)
2. [Decode](#decode)
3. [Examples](#examples)
    - [Example 1 - Buffer overflow](#buffer_overflow)
    - [Example 2 - Busybox crash](#busybox_crash)
    - [Example 3 - String race condition](#string_race_condition)

### <a name="prep">Preparation</a>

To decode the corefile, both gdb and unstripped binary are necessary.

1. Get the corresponding GDB for target corefile. The GDB must match with the target executables. This is especially important for corss-platform compilers.

2. Find the unstripped binary - the unstripped binary contains the symbols of the coredumped process. To check for the debug info in a binary, either `objdump` or `readelf` can be used. Sometimes even the `file` command can tell the difference.

```
$ file busybox
busybox: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), stripped
$ file busybox_unstripped
busybox_unstripped: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), not stripped

$ objdump -h busybox         

busybox:     file format elf32-little

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .interp       00000014  00008114  00008114  00000114  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .hash         00000938  00008128  00008128  00000128  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .dynsym       00001450  00008a60  00008a60  00000a60  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .dynstr       00000a08  00009eb0  00009eb0  00001eb0  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .rel.dyn      00000050  0000a8b8  0000a8b8  000028b8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .rel.plt      00000970  0000a908  0000a908  00002908  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .init         00000010  0000b278  0000b278  00003278  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  7 .plt          00000e3c  0000b288  0000b288  00003288  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  8 .text         00065808  0000c0c8  0000c0c8  000040c8  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  9 .fini         00000010  000718d0  000718d0  000698d0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 10 .rodata       0000e525  000718e0  000718e0  000698e0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 11 .ARM.extab    0000003c  0007fe08  0007fe08  00077e08  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 12 .ARM.exidx    000000d0  0007fe44  0007fe44  00077e44  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 13 .eh_frame     00000004  0007ff14  0007ff14  00077f14  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 14 .init_array   00000004  00080000  00080000  00078000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 15 .fini_array   00000004  00080004  00080004  00078004  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 16 .jcr          00000004  00080008  00080008  00078008  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 17 .dynamic      000000d8  0008000c  0008000c  0007800c  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 18 .got          000004d4  000800e4  000800e4  000780e4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 19 .data         0000005e  000805b8  000805b8  000785b8  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 20 .bss          00001254  00080618  00080618  00078616  2**3
                  ALLOC
 21 .ARM.attributes 00000031  00000000  00000000  00078616  2**0
                  CONTENTS, READONLY

$ objdump -h busybox_unstripped

busybox_unstripped:     file format elf32-little

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .interp       00000014  00008114  00008114  00000114  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .hash         00000938  00008128  00008128  00000128  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .dynsym       00001450  00008a60  00008a60  00000a60  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .dynstr       00000a08  00009eb0  00009eb0  00001eb0  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .rel.dyn      00000050  0000a8b8  0000a8b8  000028b8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .rel.plt      00000970  0000a908  0000a908  00002908  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .init         00000010  0000b278  0000b278  00003278  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  7 .plt          00000e3c  0000b288  0000b288  00003288  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  8 .text         00065808  0000c0c8  0000c0c8  000040c8  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  9 .fini         00000010  000718d0  000718d0  000698d0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 10 .rodata       0000e525  000718e0  000718e0  000698e0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 11 .ARM.extab    0000003c  0007fe08  0007fe08  00077e08  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 12 .ARM.exidx    000000d0  0007fe44  0007fe44  00077e44  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 13 .eh_frame     00000004  0007ff14  0007ff14  00077f14  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 14 .init_array   00000004  00080000  00080000  00078000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 15 .fini_array   00000004  00080004  00080004  00078004  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 16 .jcr          00000004  00080008  00080008  00078008  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 17 .dynamic      000000d8  0008000c  0008000c  0007800c  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 18 .got          000004d4  000800e4  000800e4  000780e4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 19 .data         0000005e  000805b8  000805b8  000785b8  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 20 .bss          00001254  00080618  00080618  00078616  2**3
                  ALLOC
 21 .comment      0000003f  00000000  00000000  00078616  2**0
                  CONTENTS, READONLY
 22 .ARM.attributes 00000031  00000000  00000000  00078655  2**0
                  CONTENTS, READONLY
 23 .debug_aranges 00004bf8  00000000  00000000  00078688  2**3
                  CONTENTS, READONLY, DEBUGGING
 24 .debug_info   000cc9f3  00000000  00000000  0007d280  2**0
                  CONTENTS, READONLY, DEBUGGING
 25 .debug_abbrev 00025932  00000000  00000000  00149c73  2**0
                  CONTENTS, READONLY, DEBUGGING
 26 .debug_line   000360e9  00000000  00000000  0016f5a5  2**0
                  CONTENTS, READONLY, DEBUGGING
 27 .debug_frame  0000c4c8  00000000  00000000  001a5690  2**2
                  CONTENTS, READONLY, DEBUGGING
 28 .debug_str    0001622c  00000000  00000000  001b1b58  2**0
                  CONTENTS, READONLY, DEBUGGING
 29 .debug_loc    000607d6  00000000  00000000  001c7d84  2**0
                  CONTENTS, READONLY, DEBUGGING
 30 .debug_ranges 00008ee8  00000000  00000000  00228560  2**3
                  CONTENTS, READONLY, DEBUGGING

$ readelf -e busybox_unstripped
    ...
Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00008114 000114 000014 00   A  0   0  1
  [ 2] .hash             HASH            00008128 000128 000938 04   A  3   0  4
  [ 3] .dynsym           DYNSYM          00008a60 000a60 001450 10   A  4   1  4
  [ 4] .dynstr           STRTAB          00009eb0 001eb0 000a08 00   A  0   0  1
  [ 5] .rel.dyn          REL             0000a8b8 0028b8 000050 08   A  3   0  4
  [ 6] .rel.plt          REL             0000a908 002908 000970 08   A  3   8  4
  [ 7] .init             PROGBITS        0000b278 003278 000010 00  AX  0   0  4
  [ 8] .plt              PROGBITS        0000b288 003288 000e3c 04  AX  0   0  4
  [ 9] .text             PROGBITS        0000c0c8 0040c8 065808 00  AX  0   0  8
  [10] .fini             PROGBITS        000718d0 0698d0 000010 00  AX  0   0  4
  [11] .rodata           PROGBITS        000718e0 0698e0 00e525 00   A  0   0  8
  [12] .ARM.extab        PROGBITS        0007fe08 077e08 00003c 00   A  0   0  4
  [13] .ARM.exidx        ARM_EXIDX       0007fe44 077e44 0000d0 00  AL  9   0  4
  [14] .eh_frame         PROGBITS        0007ff14 077f14 000004 00   A  0   0  4
  [15] .init_array       INIT_ARRAY      00080000 078000 000004 00  WA  0   0  4
  [16] .fini_array       FINI_ARRAY      00080004 078004 000004 00  WA  0   0  4
  [17] .jcr              PROGBITS        00080008 078008 000004 00  WA  0   0  4
  [18] .dynamic          DYNAMIC         0008000c 07800c 0000d8 08  WA  4   0  4
  [19] .got              PROGBITS        000800e4 0780e4 0004d4 04  WA  0   0  4
  [20] .data             PROGBITS        000805b8 0785b8 00005e 00  WA  0   0  8
  [21] .bss              NOBITS          00080618 078616 001254 00  WA  0   0  8
  [22] .comment          PROGBITS        00000000 078616 00003f 01  MS  0   0  1
  [23] .ARM.attributes   ARM_ATTRIBUTES  00000000 078655 000031 00      0   0  1
  [24] .debug_aranges    PROGBITS        00000000 078688 004bf8 00      0   0  8
  [25] .debug_info       PROGBITS        00000000 07d280 0cc9f3 00      0   0  1
  [26] .debug_abbrev     PROGBITS        00000000 149c73 025932 00      0   0  1
  [27] .debug_line       PROGBITS        00000000 16f5a5 0360e9 00      0   0  1
  [28] .debug_frame      PROGBITS        00000000 1a5690 00c4c8 00      0   0  4
  [29] .debug_str        PROGBITS        00000000 1b1b58 01622c 01  MS  0   0  1
  [30] .debug_loc        PROGBITS        00000000 1c7d84 0607d6 00      0   0  1
  [31] .debug_ranges     PROGBITS        00000000 228560 008ee8 00      0   0  8
  [32] .shstrtab         STRTAB          00000000 231448 00013a 00      0   0  1
  [33] .symtab           SYMTAB          00000000 231afc 017f00 10     34 5132  4
  [34] .strtab           STRTAB          00000000 2499fc 007cfe 00      0   0  1
    ...

```

The unstripped binary should match with the target process, which means that their symbols should be aligned. Otherwise, the decoded info might be inaccurate.

3. Collect the dependent shared libraries. If the target process needs some shared libraries, better to collect all the unstripped versions. If `ldd` command is available on the target OS, then the dependent shared libraries can be checked by

```
$ ldd `which ps`
	linux-vdso.so.1 =>  (0x00007fff8dff5000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x000000321c000000)
	libproc-3.2.8.so => /lib64/libproc-3.2.8.so (0x000000321a800000)
	libc.so.6 => /lib64/libc.so.6 (0x000000321a400000)
	libdl.so.2 => /lib64/libdl.so.2 (0x000000321b000000)
	/lib64/ld-linux-x86-64.so.2 (0x000000321a000000)
```

If `ldd` is not available on the running system, then this can be checked after loading the corefile into gdb(next section).

### <a name="decode">Decode</a>

The basic command to decode corefile is

    `gdb <unstripped_binary> <corefile>`.

If you have all shared libraries at hand, they can be automatically loaded by command

    `gdb <unstripped_binary> -q -ex "set solib-search-path <lib1>:<lib2>:...:<libn>" <corefile>`.

To check for loaded/needed shared libraries, gdb command `info sharedlibrary` can be used. To load multiple shared libaries, you can use gdb command `set solib-search-path <lib1>:<lib2>:...:<libn>`, where `libn` is the path to the shared libary. Colon(:) can be used to concatenate multiple paths.

```
(gdb) info sharedlibrary 
warning: Could not load shared library symbols for /tmp/libTest.so.
Do you need "set solib-search-path" or "set sysroot"?
From To Syms Read Shared Object Library
0x0019b820 0x001b3b9f Yes /lib/ld-linux.so.2
No /tmp/libTest.so
0x004bcf10 0x005f15cc No /lib/i386-linux-gnu/libc.so.6
(gdb) set solib-search-path /home/testuser/libtest
Reading symbols from /home/testuser/libtest/libTest.so...done.
Loaded symbols for /home/testuser/libtest/libTest.so
Reading symbols from /lib/i386-linux-gnu/libc.so.6...
Reading symbols from /usr/lib/debug/lib/i386-linux-gnu/libc-2.15.so...done.
done.
Loaded symbols for /lib/i386-linux-gnu/libc.so.6
(gdb) info sharedlibrary 
From To Syms Read Shared Object Library
0x0019b820 0x001b3b9f Yes /lib/ld-linux.so.2
0x00f893a0 0x00f894c8 Yes /home/testuser/libtest/libTest.so
0x004bcf10 0x005f15cc Yes /lib/i386-linux-gnu/libc.so.6
````

gdb command `backtrace`(or `bt`) is used to show the callback of the core dump. Other useful commands are:

    `frame <N>` - switch between call stack
    `info proc all` - show process command and memory mappings
    `info registers` - dump registers of current frame
    `x/64x $sp` - print current frame stask in hex
    `disassemble /m $pc` - show the compiled assembly code together with source code of current frame

### <a name="examples">Examples</a>

The examples are based on real coredump found in our products. However, the proprietary callstack is replaced by toy snippet.

#### <a name="buffer_overflow">Example 1 - packet encoding buffer overflow<a>

In this example, process A encodes a packet and then send it to process B through IPC socket. Process B copy this packet to its own buffer and add additional header before sending it out. The buffer sizes for both processes are `4096` bytes.

In most cases, process B just store one packet from process A into its buffer. However, there are chances that process B need to put two packets sent by process A into the buffer. This is controlled by a variable `enc_attr`. The pseudo code for encoding is: 

```cpp
for (int i = 0; i < enc_attr; i++) {
    ......
    encode_one_packet(uchar_t * buffer, uint32_t len);
    ......
}

```

When process B crashed, the `enc_attr` was wrongly set to `2`. Since this kind of encoding packets could be as long as `4006` bytes in process A, two packets would definitely exceed the buffer size of process B. Following is the decode of the corefile.


```c
(gdb) disassemble /m $pc
1292	in ./msgproc/msg_proc.cpp
   0x00075c44 <+544>:	mov	r3, #0
   0x00075c4c <+552>:	str	r3, [sp, #32]
   0x00075c58 <+564>:	b	0x75ddc
   0x00075dd0 <+940>:	ldr	r2, [sp, #32]
   0x00075dd4 <+944>:	add	r2, r2, #1
   0x00075dd8 <+948>:	str	r2, [sp, #32]           // i = 1, x $sp+32, 0x00000001
   0x00075ddc <+952>:	ldr	r3, [sp, #32]
   0x00075de0 <+956>:	ldr	r1, [sp, #60]	; 0x3c  // enc_attr = 2
   0x00075de4 <+960>:	cmp	r3, r1
   0x00075de8 <+964>:	bcc	0x75c5c 
   0x00075dec <+968>:	b	0x75eac 

    ......
1340	in ./msgproc/msg_proc.cpp
1341	in ./msgproc/msg_proc.cpp
1342	in ./msgproc/msg_proc.cpp
1343	in ./msgproc/msg_proc.cpp
1344	in ./msgproc/msg_proc.cpp
   0x00075d24 <+768>:	ldr	r3, [sp, #184]	; 0xb8
   0x00075d34 <+784>:	ldr	r2, [sp, #44]	; 0x2c
   0x00075d38 <+788>:	add	r1, sp, #80	; 0x50
   0x00075d3c <+792>:	str	r3, [sp]
   0x00075d40 <+796>:	stmib	sp, {r1, r6}
   0x00075d44 <+800>:	str	r2, [sp, #12]
   0x00075d48 <+804>:	ldr	r0, [sp, #40]	; 0x28
   0x00075d4c <+808>:	ldr	r1, [sp, #64]	; 0x40
   0x00075d50 <+812>:	ldr	r2, [sp, #36]	; 0x24
   0x00075d54 <+816>:	mov	r3, r4
   0x00075d58 <+820>:	str	r12, [sp, #16]
   0x00075d5c <+824>:	bl	0x7573c

1345	in ./msgproc/msg_proc.cpp
=> 0x00075d60 <+828>:	subs	r10, r0, #0
   0x00075d64 <+832>:	ldr	r12, [sp, #16]
   0x00075d68 <+836>:	beq	0x75dc0

1346	in ./msgproc/msg_proc.cpp
1347	in ./msgproc/msg_proc.cpp
   0x00075d6c <+840>:	ldr	r3, [sp, #36]	; 0x24
   0x00075d74 <+848>:	add	r3, r3, r10
   0x00075d78 <+852>:	str	r3, [sp, #36]	; 0x24

1348	in ./msgproc/msg_proc.cpp
   0x00075d70 <+844>:	add	r11, r11, r10
    ......

(gdb) p *encInfo
$62 = {msglvl = 6, type = 239, enc_attr = 2 '\002', override = false, slotId = 0 '\000', 
  skip = false, part = 0 '\000'}
(gdb) x $sp+32
0x40dd1bc8:	0x00000001  // i
(gdb) x $sp+60
0x40dd1be4:	0x00000002  // enc_attr

```

`i == 1` and `enc_attr == 2` means that process B was trying to put two packets into the 4K buffer.

#### <a name="busybox_crash">Example 2 - Busybox crash</a>

This is a weird problem caused by some kernel failure - after this failure, all busybox commands lead to coredump. The investigation is still in progress.


```c
(gdb) info proc all
exe = '/usr/bin/killall -q -15 process1 process2'
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	    0x8000    0x84000    0x7c000        0x0 /bin/busybox
	   0x8c000    0x8d000     0x1000    0x7c000 /bin/busybox
	0x76eb7000 0x76f48000    0x91000        0x0 /lib/libuClibc-0.9.33.2.so
	0x76f50000 0x76f51000     0x1000    0x91000 /lib/libuClibc-0.9.33.2.so
	0x76f51000 0x76f52000     0x1000    0x92000 /lib/libuClibc-0.9.33.2.so
	0x76f57000 0x76f6a000    0x13000        0x0 /lib/libm-0.9.33.2.so
	0x76f71000 0x76f72000     0x1000    0x12000 /lib/libm-0.9.33.2.so
	0x76f72000 0x76f73000     0x1000    0x13000 /lib/libm-0.9.33.2.so
	0x76f73000 0x76f7a000     0x7000        0x0 /lib/ld-uClibc-0.9.33.2.so
	0x76f81000 0x76f82000     0x1000     0x6000 /lib/ld-uClibc-0.9.33.2.so
(gdb) bt full
#0  0x000112ec in bb_strtoul (arg=0x7eb64f01 "15", endp=0x7eb64f01, base=1929393457)
    at libbb/bb_strtonum.c:102
        v = <optimized out>
        endptr = 0x7eb64dec ""
#1  0x000228f0 in kill_main (argc=496, argv=0x8c684 <ruid>) at procps/kill.c:147
        arg = 0x7eb64f01 "15"
        pid = <optimized out>
        signo = 15
        errors = 0
        quiet = 2128038736
        char3 = 97 'a'
#2  0x0000f568 in run_applet_no_and_exit (applet_no=54, argv=0x7eb64de4, argv@entry=0x5)
    at libbb/appletlib.c:755
        argc = 5
#3  0x0000f5ac in run_applet_and_exit (name=0x7eb64de4 "\354N\266~\375N\266~", argv=0x5, 
    argv@entry=0x7eb64de4) at libbb/appletlib.c:762
        applet = <optimized out>
#4  0x0000f804 in main (argc=<optimized out>, argv=0x7eb64de4) at libbb/appletlib.c:819
No locals.
(gdb) disassemble /m $pc
Dump of assembler code for function bb_strtoul:
35		errno = ERANGE; /* this ain't as small as it looks (on glibc) */
   0x000112dc <+32>:	ldr	r3, [r3]
   0x000112f0 <+52>:	movhi	r2, #34	; 0x22
   0x000112f4 <+56>:	strhi	r2, [r3]
    ......
93	#if ULONG_MAX != ULLONG_MAX
94	unsigned long FAST_FUNC bb_strtoul(const char *arg, char **endp, int base)
95	{
   0x000112bc <+0>:	push	{r0, r1, r2, r4, r5, lr}

96		unsigned long v;
97		char *endptr;
98	
99		if (!endp) endp = &endptr;
   0x000112c0 <+4>:	subs	r4, r1, #0
   0x000112c4 <+8>:	addeq	r4, sp, #4

100		*endp = (char*) arg;
   0x000112c8 <+12>:	str	r0, [r4]

101	
102		if (!isalnum(arg[0])) return ret_ERANGE();
   0x000112cc <+16>:	ldrb	r1, [r0]
   0x000112e8 <+44>:	sub	r1, r1, #97	; 0x61
=> 0x000112ec <+48>:	cmp	r1, #25
   0x000112f8 <+60>:	mvnhi	r0, #0
   0x000112fc <+64>:	bhi	0x1131c <bb_strtoul+96>

103		errno = 0;
   0x00011300 <+68>:	mov	r5, #0
   0x00011308 <+76>:	str	r5, [r3]

104		v = strtoul(arg, endp, base);
   0x00011304 <+72>:	mov	r1, r4
   0x0001130c <+80>:	bl	0xb678 <strtoul>

105		return handle_errors(v, endp);
   0x00011310 <+84>:	mov	r1, r5
   0x00011314 <+88>:	ldr	r2, [r4]
   0x00011318 <+92>:	bl	0x11134 <handle_errors>

106	}
   0x0001131c <+96>:	pop	{r1, r2, r3, r4, r5, lr}
   0x00011320 <+100>:	bx	lr
   0x00011324 <+104>:	andeq	sp, r8, r0, asr r8

End of assembler dump.
```

#### <a name="string_race_condition">Example 3 - Click String race condition</a>

This is a coredump caused by [Click String library](https://github.com/kohler/click/blob/master/include/click/string.hh). This library is not thread-safe for assignment like `String str = (char *)pstr;` - the _r.memo is possible to be reused before it's set to NULL in `String::deref()` after deleting memo by `String::delete_memo()`.

```cpp
class DLL_PUBLIC String { public:
    ......
    inline String &operator=(const char *cstr);
    ......

private:

    /** @cond never */
    struct memo_t {
	volatile uint32_t refcount;
	uint32_t capacity;
	volatile uint32_t dirty;
#if HAVE_STRING_PROFILING > 1
	memo_t **pprev;
	memo_t *next;
#endif
	char real_data[8];	// but it might be more or less
    };

    struct rep_t {
	const char *data;
	int length;
	memo_t *memo;
    };
    mutable rep_t _r;		// mutable for c_str()

    inline void deref() const {
	if (_r.memo) {
	    assert(_r.memo->refcount);
	    if (atomic_uint32_t::dec_and_test(_r.memo->refcount))
		delete_memo(_r.memo);
	    _r.memo = 0;
	}
    }
    ......
};

```

Decode:


```cpp
(gdb) bt full
#0  0x4065975c in __GI_raise (sig=6) at libpthread/nptl/sysdeps/unix/sysv/linux/raise.c:67
#1  0x4064eeb4 in __GI_abort () at libc/stdlib/abort.c:89
        sigs = {__val = {32, 0}}
#2  0x404eadb4 in __assert (assertion=<optimized out>, file=<optimized out>, line=<optimized out>, 
    function=<optimized out>) at click/library/util.cc:2333
No locals.
#3  0x4046dfd0 in deref (this=<optimized out>) at click/library/string.hh:364
No locals.
#4  String::deref (this=0x8e83f8 <testCfg+58248>) at click/library/string.hh:362
No locals.
#5  0x404f8a54 in String::assign (this=this@entry=0x8e83f8 <testCfg+58248>, 
    str=str@entry=0x9a813e <radCfg+114> "sample-wlc1", len=16, len@entry=-1, 
    need_deref=need_deref@entry=true) at click/library/string.cc:376
        __PRETTY_FUNCTION__ = "void String::assign(const char*, int, bool)"
#6  0x00057330 in String::operator= (this=this@entry=0x8e83f8 <testCfg+58248>, 
    cstr=cstr@entry=0x9a813e <radCfg+114> "sample-wlc1")
    at click/library/string.hh:798
No locals.
#7  0x0005d144 save_config () at ./save_config.cc:2382
        sa = {r_ = {s = 0xfc1b0 <String::null_data> "", len = 0, cap = 0}}
        venue_cfg = <optimized out>
#8  0x0002b764 in set_ssh_state (ssh_enabled=<optimized out>)
    at ./linux_platform_shim.c:1826
    ....

(gdb) frame 5
(gdb) p str1._r.memo
$28 = (String::memo_t *) 0x0
(gdb) p *(str1._r.memo)
Cannot access memory at address 0x0
(gdb) p str2._r.memo
$31 = (String::memo_t *) 0x152c100
(gdb) p *(str2._r.memo)
$32 = {refcount = 1, capacity = 20, dirty = 16, real_data = "sample"}
(gdb) p str3._r.memo
$33 = (String::memo_t *) 0x15bd5e0 // _r.memo != 0
(gdb) p *(str3._r.memo)
$34 = {refcount = 0, capacity = 0, dirty = 16777216, real_data = "\000\000\000\000\000\000\000\017"} // _r.memo->refcount == 0, assert failure in String::deref().

```

In the above decode, both `str1` and `str2` are expected. However, the memo of `str3` is wrong.

### References

1. [http://visualgdb.com/gdbreference/commands/set_solib-search-path](http://visualgdb.com/gdbreference/commands/set_solib-search-path)
