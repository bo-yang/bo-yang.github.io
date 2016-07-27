---
layout: post
title: Be careful to pthread_exit() in main() 
categories: 
- Unix/Linux
- Concurrency
- Notes
tags:
- Unix/Linux
- Concurrency
- Notes
status: publish
type: post
published: true
author: Bo Yang
---

When using `pthread` for multithreading, most threads call `pthread_exit()` implicitly on return from the thread start routine. Besides, `pthread_exit()` also can be used to terminate the initial process thread in `main()`, leaving other threads to continue operation. The process will go away automatically when the last thread terminates. If you don't care about the process exit status, or if is difficult to know the created thread IDs(e.g. created by third party APIs), you can call 

	pthread_detach(pthread_self());
	pthread_exit(NULL);
	
at the end of the `main()` function. 

However, you must be carefull when using `pthread_exit()` in the main thread. Because after calling `pthread_exit()` and before the process really terminate, the process becomes "zombie" - it still exists even though it is "dead", just like a Unix/Linux process that's terminated but hasn't yet been "reaped" by a `wait` operation. The zombie process may retain most or all of the system resources that it used when running, so it is not a good idea to leave threads in this state for longer than necessary. And obviously, zombie process cannot save you resources! So also don't try `pthread_exit()` for saving CPU and memory.

I also noticed a undocumented problem caused by `pthread_exit()` - it may lead to failure of open procfs(`/proc/`) files! If one of your threads would open `/proc/mounts`(_currently I only find this file will go wrong, and other procfs files like `/proc/cpuinfo` or `/proc/uptime` can be successfully opened_) during its life, and `pthread_exit()` is called after creating these threads in the main thread, you will meet the "Invalid argument" error because of functions like `open("/proc/mounts",'r')`.

Following program demonstrates this problem:

~~~ cpp
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#define NUM_THREADS	5

void *PrintHello(void *threadid)
{
   long tid;
   tid = (long)threadid;
   printf("Hello World! It's me, thread #%ld!\n", tid);

   sleep(10); // sleep sometime

   /* Read file */
   FILE* fp=fopen("/proc/mounts","r"); // would fail
   //FILE* fp=fopen("/proc/cpuinfo","r");  // would succeed
   if(fp==NULL) {
	   fprintf(stderr,"Failed to open file!\n");
   } else {
	   char line[80];
	   if(fgets(line,80,fp)==NULL)
		   fprintf(stderr,"Failed to read file!\n");
	   else
		   fprintf(stdout,"%s\n",line);
	   fclose(fp);
   }

   sleep(1200);
   pthread_exit(NULL);
}

int main(int argc, char *argv[])
{
   pthread_t threads[NUM_THREADS];
   int rc;
   long t;
   for(t=0;t<NUM_THREADS;t++){
     printf("In main: creating thread %ld\n", t);
     rc = pthread_create(&threads[t], NULL, PrintHello, (void *)t);
     if (rc){
       printf("ERROR; return code from pthread_create() is %d\n", rc);
       exit(-1);
       }
     }

   /* Last thing that main() should do */
   pthread_detach(pthread_self());
   pthread_exit(NULL);
}
~~~

After building above code, say `pthread_exit_test`, run this program and find the PID of it. Then `cat /proc/<PID>/status`, you will find the process status like

	Name:	pthread_exit_te
	State:	Z (zombie)
	Tgid:	3091
	Pid:	3091
	PPid:	2849
	TracerPid:	0
	Uid:	370845	370845	370845	370845
	Gid:	25	25	25	25
	Utrace:	0
	FDSize:	0
	Groups:	25 1000000312 
	Threads:	6
	SigQ:	0/78966
	SigPnd:	0000000000000000
	ShdPnd:	0000000000000000
	SigBlk:	0000000000000000
	SigIgn:	0000000000000004
	SigCgt:	0000000180000000
	CapInh:	0000000000000000
	CapPrm:	0000000000000000
	CapEff:	0000000000000000
	CapBnd:	ffffffffffffffff
	Cpus_allowed:	3f
	Cpus_allowed_list:	0-5
	Mems_allowed:	00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
	Mems_allowed_list:	0
	voluntary_ctxt_switches:	14
	nonvoluntary_ctxt_switches:	1

For the above program, if replace `pthread_exit()` with `pthread_join()` or `while(1) {sleep(120); }`, it would work well. The `while` loop is especially usefull when you don't know the thread id to be joined.

Until now I am still don't know why openning `/proc/mounts` would fail and why openning `/proc/cpuinfo` could succeed in above code. I also tried other system file or link, and found all of them could be successively readed. 

### References
1. [Should pthread_exit() be used in main()?](https://groups.google.com/forum/#!topic/comp.programming.threads/b1r0oUwG4rM)
2. [Programming with POSIX Threads](http://books.google.com/books?id=_xvnuFzo7q0C&printsec=frontcover&source=gbs_ge_summary_r&cad=0#v=onepage&q&f=false)
