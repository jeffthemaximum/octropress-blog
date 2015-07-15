---
layout: post
title: "subprocess.Popen: Using python's subprocess module to pipe stdin and stdout between processes"
date: 2015-07-15 12:04:14 -0400
comments: true
categories: technical
---

Let's say you have two programs that need to communicate with eachother via stdin and stdout. [Pipes](http://www2.cs.uregina.ca/~hamilton/courses/330/notes/unix/pipes/pipes.html) is one way to accomplish this. A pipe is a connection between two processes such that the stdout of one program becomes the stdin of the other.

With python, this piping process is streamlined with the [subprocess module](https://docs.python.org/2/library/subprocess.html). 

Let's look at an example of how this plays out in python. Take our first program, foo.py: 

**foo.py:**
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>import time
import sys

for i in range(0,10):
    print &quot;Hello world &quot; + str(i)
    sys.stdout.flush()
    time.sleep(1)
</code></pre>

All foo.py does is print "Hello World" with an iterator number, and it does this every second. This is our sample output. Now, let's imagine a second program that is trying to catch the stdout from foo.py as stdin. Let's make that second program called bar.py, and it will be based on [python's subprocess module documentation](https://docs.python.org/2/library/subprocess.html):

**bar.py:**
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>import subprocess

proc = subprocess.Popen(['python','fake_chess_engine.py'],stdin=subprocess.PIPE)

proc.communicate()
</code></pre>

Running bar.py produces our desired result: it catches the stdout from foo.py, uses it as stdin, and prints "Hello world" plus an iterator to the console every second. 

Now, let's say you want to catch each line of output from foo.py as a variable in bar.py, and maybe save that output to a variable in bar.py. Let's write baz.py, based on [this post from stackoverflow](https://docs.python.org/2/library/subprocess.html), to accomplish that goal:

**baz.py:**
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>import subprocess

proc = subprocess.Popen(['python','fake_chess_engine.py'],stdout=subprocess.PIPE)

while True:
  line = proc.stdout.readline()
  if line != '':
    print &quot;test: &quot;, line.rstrip()
  else:
    break
</code></pre>

You'll notice two key differences between bar.py and baz.py. First, baz.py uses a "stdout" arg in it's call to subpress.Popen, where bar.py uses a "stdin" arg here. Second, baz.py replaces bar.py's "proc.communicate()" call with a while-block. This while-block in baz.py accomplishes our goal of collecting each line of output from foo.py in a variable. With baz.py, we just print that variable out. However, you could presumably append that line to a list and save all your line variables in this way.

Any thoughts, questions, comments? 