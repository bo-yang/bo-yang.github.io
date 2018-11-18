---
layout: post
title: BACI Simple Install and Compile Guide
categories: 
- BACI
tags:
- BACI
- Tips
status: publish
type: post
published: true
author: Bo Yang
---
This is a guideline for a course of this semester, for which I am serving as a teaching assistant. I publish it here in hope that it can also help someone else who want to use BACI.

#### Install BACI

Go to <a href="http://inside.mines.edu/~tcamp/baci/baci_index.html#Obtain">http://inside.mines.edu/~tcamp/baci/baci_index.html#Obtain</a> and download BACI executables:

<a href="http://inside.mines.edu/~tcamp/baci/badosxe-2005Nov09.zip">BACI DOS executables</a> for Windows,

<a href="http://inside.mines.edu/~tcamp/baci/balnxxe32-2007Nov25.tar.gz">BACI LINUX executables (32 bit)</a> and <a href="http://inside.mines.edu/~tcamp/baci/balnxxe64-2012Oct01.tar.gz">BACI LINUX executables (64 bit)</a> for Linux.

For windows user, after download and unzip the BACI DOS executables, please put the folder called badosxe directly under the `C:\` drive so it’s easy to find it. You can do all your BACI work inside this directory.

If your badosxe folder is `C:\badosxe`, then in your command prompt (cmd) your path should be `C:\badosxe> bacc <file_name>`.

For Linux user, after download and gunzip the BACI executables, please make sure to add `balnxxe` into your PATH: `export PATH=$PATH:/path/to/your/balnxxe` (You also can add this line into `$HOME/.profile`, so that this command will be executed every time you login).

If you want to use jBACI(BACI with a graphical IDE), please go to <a href="https://code.google.com/p/jbaci/downloads/list">https://code.google.com/p/jbaci/downloads/list</a>, and choose the <a href="https://code.google.com/p/jbaci/downloads/detail?name=jbaci-1.4.6.binaries.zip&amp;can=2&amp;q=">jbaci-1.4.6.binaries.zip</a>. Unfortunately, currently jBACI is only available for Windows.

If you want to use BACI on the Eustis server, please download Linux version BACI first, gunzip it and then upload them into your home folder of Eustis server. Please remember to add BACI executables into your PATH(as step 3).

NetBeans + BACI plugin is the best choice for people who prefer a modern IDE or MAC users. You need to install NetBeans first, and then follow the <a href="http://www.cslab.pepperdine.edu/warford/cosc450/cosc-450-Setup-for-BACI.pdf">guide of setting up BaciBeans</a>. NetBeans is a cross-platform IDE, so it can work on all operating systems that have Java JDK installed.

#### Compile and run BACI program

Make sure that bacc is in your PATH, or put your BACI program in the same folder of bacc. Note the .cm extension. This is the extension that identifies BACI source files to the BACI compiler.

Invoke the compile command bacc your_prog.cm. This creates an object file called your_prog.pco and also a listing file called `your_prog.lst`.

Invoke the interpreter through the command bainterp `your_prog` (please note, there is no suffix here) to execute your code. bainterp has an option `-t` that will display the order in which processes terminate within the program.

#### Verify the installation of BACI

Download <a href="http://www.cs.ucf.edu/courses/cop4020/spr2014/BACI/example.cm">example.cm</a>.

Run command  `bacc example.cm`. If you see following outputs, then the example.cm is successfully compiled:

~~~
bo@HEC-2GQFTK1:~/Documents/BACI$ bacc example.cm
  
Pcode and tables are stored in example.pco
  
Compilation listing is stored in example.lst
~~~

Execute command `bainterp -t example`. If you get following message, then `bainterp` runs correctly.

~~~
bo@HEC-2GQFTK1:~/Documents/BACI$ bainterp -t example
Source file: example.cm  Wed Jan 22 15:16:12 2014
Executing PCODE ...
before v(count) value of count is 0
process 2 increment,  procedure increment ended
before p(count) value of count is 1
process 1 decrement,  procedure decrement ended
~~~
  
#### [More Options of BACI Compiler](http://inside.mines.edu/~tcamp/baci/baci_index.html#Using)
  
A BACI source file using the C-- compiler should use a `.cm` suffix. To execute a program in BACI, there are two steps:
  
1. Compile a `.cm` file to obtain a PCODE file (`.pco`)

Usage: bacc [optional_flags] source_filename

Optional_flags:

* -h show this help
* -c make a .pob object file for subsequent linkingInterpret a PCODE file (.pco) to execute the program

2. Usage bainterp [optional_flags] pcode_filename
  
Optional_flags:
  
* -d enter the debugger, single step, set breakpoints
* -e show the activation record (AR) on entry to each process
* -x show the AR on exit from each process
* -t announce process termination
* -h show this help
* -p show PCODE instructions as they are executed

There is a shell script, `baccint`, that will call the compiler and then call the interpreter for you. It passes the options that you give it along to the interpreter. If you are using the Pascal compiler syntax, then the source file should be with a .pm suffix, and you compile the program with the bapas compiler.
