---
layout: post
title: Python - Run Command With Timeout
categories: 
- Python
tags:
- Python
status: publish
type: post
published: true
author: Bo Yang
---

This post introduces a Python class `Command` that can run a shell command with a time limit - the command will be killed after timer expires. This is implemented by `threading` and `subprocess` modules.

The `Command` class supports running command in two modes: run the command ignoring the output, or capture the output of the command through pipe.

The source code is also available in [my GitHub](https://github.com/bo-yang/misc/blob/master/run_command_timeout.py).

```python

import subprocess
import threading

""" Run system commands with timeout
"""
class Command(object):
    def __init__(self, cmd):
        self.cmd = cmd
        self.process = None
        self.out = None

    def run_command(self, capture = False):
        if not capture:
            self.process = subprocess.Popen(self.cmd,shell=True)
            self.process.communicate()
            return
        # capturing the outputs of shell commands
        self.process = subprocess.Popen(self.cmd,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE,stdin=subprocess.PIPE)
        out,err = self.process.communicate()
        if len(out) > 0:
            self.out = out.splitlines()
        else:
            self.out = None

    # set default timeout to 2 minutes
    def run(self, capture = False, timeout = 120):
        thread = threading.Thread(target=self.run_command, args=(capture,))
        thread.start()
        thread.join(timeout)
        if thread.is_alive():
            print 'Command timeout, kill it: ' + self.cmd
            self.process.terminate()
            thread.join()
        return self.out

'''basic test cases'''

# run shell command without capture
Command('pwd').run()
# capture the output of a command
date_time = Command('date').run(capture=True)
print date_time
# kill a command after timeout
Command('echo "sleep 10 seconds"; sleep 10; echo "done"').run(timeout=2)

```

The output of above test cases are:

```shell
$ python run_command_timed.py 
/Users/boyang/Documents/prog
['Fri Dec  2 09:17:50 PST 2016']
sleep 10 seconds
Command timeout, kill it: echo "sleep 10 seconds"; sleep 10; echo "done"
```

####References:

- [http://stackoverflow.com/questions/1191374/using-module-subprocess-with-timeout](http://stackoverflow.com/questions/1191374/using-module-subprocess-with-timeout)
- [https://pymotw.com/2/threading/](https://pymotw.com/2/threading/)
