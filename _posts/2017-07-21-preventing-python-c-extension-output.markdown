---
layout: post
title:  "preventing python c extension output"
date:   2017-07-21 18:14:00 -0700
categories: python c output redirection iot
---

Recently I ran into an issue while I was integrating an IoT python library into my application.

In a nutshell, the library was outputting it's own Info logs to standard out and I needed to squelch it. On first glance, this seemed like an easy problem. Just redirect standard out into an OS agnostic null device...something like

```python
import os
import sys

original_stdout = sys.stdout
devnull = open(os.devnull, 'w')
sys.stdout = devnull
```

and then restore `sys.stdout` when you are done right?  

Well, normally yes...except in this case, the python library I was squelching was a python C extension using printf for logging! The pattern above was not help.

Why? Because C libraries use file [descriptors](https://en.wikipedia.org/wiki/File_descriptor) 0 through 2 for input/output/error. Sheesh...
In order to overcome this we need to access the file descriptors (fd) directly then use dup and dup2 system call to copy the original fd and then replace it with null respectively. 

```python
orig_stdout_fno = os.dup(sys.stdout.fileno())
devnull = open(os.devnull, 'w')

# replace stdout with devnull
os.dup2(devnull.fileno(), 1)

... # do stuff
... # make noise

# restore original
os.dup2(orig_stdout_fno.fileno(), 1)
devnull.close()
```

Awesome it worked! But thats a pretty verbose wrapper (particularly with try/except/finaly) if you need to use this in multiple places.

What can we do to make things more concise? Why not a python contextmanager pattern?


```python
import os
import sys
import contextlib

@contextlib.contextmanager
def block_stdout():
    devnull = open(os.devnull, 'w')
    orig_stdout_fno = os.dup(sys.stdout.fileno())
    os.dup2(devnull.fileno(), 1)
    try:
        yield
    finally:
        os.dup2(orig_stdout_fno, 1)
        devnull.close()
```

**Nice!** This is absolutely compact and reusable. The contextmanager will handle the setup then cleanup in the finally block. Now we can use it like this everywhere we need to

```python
def Foo(...):
    ...
    with block_stdout():
        noisyFunction(...)
    doStuff()
```
