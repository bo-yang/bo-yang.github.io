---
layout: post
title: How to terminate a C++ std::thread?
categories: 
- C/C++
- Concurrency
tags:
- C/C++
- Concurrency
status: publish
type: post
published: true
author: Bo Yang
---

In C++11, C++ introduced built-in support for threads `std::thread`. Since then, starting a new thread in C++ is as easy as defining an object. However, dynamically terminating a running C++ thread is still very tricky, especially for the joined/detached thread. There are plenty of discussions of this topic, and the conclusion is that

> ["terminate 1 thread + forcefully (target thread doesn't cooperate) + pure C++11 = No way"](https://stackoverflow.com/questions/12207684/how-do-i-terminate-a-thread-in-c11).

This post is not aimed to provide a magic to kill C++ thread with built-in C++ APIs - apologize for it. Instead, I am going to explain how to kill a C++ thread using native(OS/compiler-dependent) function.

_The complete code in this post can be found on [my github](https://github.com/bo-yang/terminate_cpp_thread), which is built with GCC 4.8.5 and verified on CentOS 7._

### 1. `std::thread` destructor not work

The destructor [`std::thread::~thread()`](http://en.cppreference.com/w/cpp/thread/thread/~thread) can only terminate the thread when the thread is still [`joinable`](http://en.cppreference.com/w/cpp/thread/thread/joinable). A C++ thread is `joinable` after it is started and before calling `join` or `detach`(see [this example](http://en.cppreference.com/w/cpp/thread/thread/joinable)). So for joined/detached thread, the `std::thread` destructor cannot terminate the thread at all.

Following is an class to dynamically start and stop a thread. The `std::unordered_map<std::string, std::thread>` is used to record thread name and thread. And in `stop_thread()`, the `std::thread` destructor is directly called.

```cpp
class Foo {
public:
    void sleep_for(const std::string &tname, int num)
    {
        prctl(PR_SET_NAME,tname.c_str(),0,0,0);        
        sleep(num);
    }

    void start_thread(const std::string &tname)
    {
        std::thread thrd = std::thread(&Foo::sleep_for, this, tname, 3600);
        thrd.detach();
        tm_[tname] = std::move(thrd);
        std::cout << "Thread " << tname << " created:" << std::endl;
    }

    void stop_thread(const std::string &tname)
    {
        ThreadMap::const_iterator it = tm_.find(tname);
        if (it != tm_.end()) {
            it->second.std::thread::~thread(); // thread not killed
            tm_.erase(tname);
            std::cout << "Thread " << tname << " killed:" << std::endl;
        }
    }

private:
    typedef std::unordered_map<std::string, std::thread> ThreadMap;
    ThreadMap tm_;
};

```

To show the running threads, function `show_thread()` is called everytime starting/stopping a thread.


```cpp
void show_thread(const std::string &keyword)
{
    std::string cmd("ps -T | grep ");
    cmd += keyword;
    system(cmd.c_str());
}
 
int main()
{
    Foo foo;
    std::string keyword("test_thread");
    std::string tname1 = keyword + "1";
    std::string tname2 = keyword + "2";

    // create and kill thread 1
    foo.start_thread(tname1);
    show_thread(keyword);
    foo.stop_thread(tname1);
    show_thread(keyword);

    // create and kill thread 2
    foo.start_thread(tname2);
    show_thread(keyword);
    foo.stop_thread(tname2);
    show_thread(keyword);

    return 0;
}
```

The output of the the above test code is like:

```
$ g++ -Wall -std=c++11 kill_cpp_thread.cc -o kill_cpp_thread -pthread -lpthread
$ ./kill_cpp_thread
Thread test_thread1 created:
29469 29470 pts/5    00:00:00 test_thread1
Thread test_thread1 killed:
29469 29470 pts/5    00:00:00 test_thread1
Thread test_thread2 created:
29469 29470 pts/5    00:00:00 test_thread1
29469 29477 pts/5    00:00:00 test_thread2
Thread test_thread2 killed:
29469 29470 pts/5    00:00:00 test_thread1
29469 29477 pts/5    00:00:00 test_thread2
```

Obviously, the test threads could not be killed in `Foo::stop_thread()`.


### 2. `std::thread::id` vs. pthread_t

To terminate threads with OS/compiler-dependent functions, we need to know how to get the native thread data type from C++ `std::thread`. Fortunately, `std::thread` provides an API `native_handle()` to get the thread's native handle type - before calling `join()` or `detach()`. And this native handle can be passed to native OS thread terminate function, e.g. `pthread_cancel()`.

Here's a demo code to show the fact that the `std::thread::native_handle()`, `std::thread::get_id()` and [`pthread_self()`](http://pubs.opengroup.org/onlinepubs/9699919799/) return the same `pthread_t` to handle a C++ thread for Linux/GCC. You also can [try it out online](http://coliru.stacked-crooked.com/view?id=44c988ce133260ee).

```cpp
#include <mutex>
#include <iostream>
#include <chrono>
#include <cstring>
#include <pthread.h>
 
std::mutex iomutex;
void f(int num)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::lock_guard<std::mutex> lk(iomutex);
    std::cout << "Thread " << num << " pthread_t " << pthread_self() << std::endl;
}
 
int main()
{
    std::thread t1(f, 1), t2(f, 2);
    
    //t1.join(); t2.join();
    //t1.detach(); t2.detach();
    
    std::cout << "Thread 1 thread id " << t1.get_id() << std::endl;
    std::cout << "Thread 2 thread id " << t2.get_id() << std::endl;
    
    std::cout << "Thread 1 native handle " << t1.native_handle() << std::endl;
    std::cout << "Thread 2 native handle " << t2.native_handle() << std::endl;
    
    t1.join(); t2.join();
    //t1.detach(); t2.detach();
}
```

Build and run it get the following result:

```
$ g++ -Wall -std=c++11 cpp_thread_pthread.cc -o cpp_thread_pthread -pthread -lpthread
$ ./cpp_thread_pthread 
Thread 1 thread id 140109390030592
Thread 2 thread id 140109381637888
Thread 1 native handle 140109390030592
Thread 2 native handle 140109381637888
Thread 1 pthread_t 140109390030592
Thread 2 pthread_t 140109381637888
```

However, after calling `join()` or `detach()`, the C++ thread loses the info of native handle type:

```cpp
int main()
{
    std::thread t1(f, 1), t2(f, 2);
    
    t1.join(); t2.join();
    //t1.detach(); t2.detach();
    
    std::cout << "Thread 1 thread id " << t1.get_id() << std::endl;
    std::cout << "Thread 2 thread id " << t2.get_id() << std::endl;
    
    std::cout << "Thread 1 native handle " << t1.native_handle() << std::endl;
    std::cout << "Thread 2 native handle " << t2.native_handle() << std::endl;
}
```

```
$ ./cpp_thread_pthread
Thread 1 pthread_t 139811504355072
Thread 2 pthread_t 139811495962368
Thread 1 thread id thread::id of a non-executing thread
Thread 2 thread id thread::id of a non-executing thread
Thread 1 native handle 0
Thread 2 native handle 0
```

So if you call `pthread_cancel(t1.native_handle())` after `t1` joined/detached, your program will definitely coredump.


### 3. The Terminator

So in summary, to effectively call native thread termination function(e.g. `pthread_cancel`), you need to save the native handle before calling `std::thread::join()` or `std::thread::detach()`. So that your native terminator always has a valid native handle to use.

Following is the revised `Foo` class that can stop C++ thread:

```cpp
class Foo {
public:
    void sleep_for(const std::string &tname, int num)
    {
        prctl(PR_SET_NAME,tname.c_str(),0,0,0);        
        sleep(num);
    }

    void start_thread(const std::string &tname)
    {
        std::thread thrd = std::thread(&Foo::sleep_for, this, tname, 3600);
        tm_[tname] = thrd.native_handle();
        thrd.detach();
        std::cout << "Thread " << tname << " created:" << std::endl;
    }

    void stop_thread(const std::string &tname)
    {
        ThreadMap::const_iterator it = tm_.find(tname);
        if (it != tm_.end()) {
            pthread_cancel(it->second);
            tm_.erase(tname);
            std::cout << "Thread " << tname << " killed:" << std::endl;
        }
    }

private:
    typedef std::unordered_map<std::string, pthread_t> ThreadMap;
    ThreadMap tm_;
};
```

And the result is:


```
$ g++ -Wall -std=c++11 kill_cpp_thread.cc -o kill_cpp_thread -pthread -lpthread
$ ./kill_cpp_thread 
Thread test_thread1 created:
30332 30333 pts/5    00:00:00 test_thread1
Thread test_thread1 killed:
Thread test_thread2 created:
30332 30340 pts/5    00:00:00 test_thread2
Thread test_thread2 killed:
```

