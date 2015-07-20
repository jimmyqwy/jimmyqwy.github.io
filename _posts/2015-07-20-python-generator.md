---
layout: post
title: "[Python] Generator"
date:   2015-07-20 16:19:11
tags:
- Python
- Generator
---

### Notes about Generator in Python

* Important difference from a list.
  * Does not construct a list.
  * Only useful purpose is iteration
  * Once consumed, can't be reused

```python  
>>> a = [1,2,3,4,5]
>>> b = [2*x for x in a]
>>> b
[2,4,6,8,10]
>>> c = (2*x for x in a)
<generator object at 0x.....>
```

* Generator Functions:
```
def foo():
    ...
    yield expression
    ...
```

* Generator Expressions

*NOTE the ()*
```
(expression for i in s if cond1
            for j in t if cond2
            ...
            if condfinal)
```
==>> Same as
```
for i in s:
    if cond1:
        for j in t:
            if cond2:
                ...
                if condfinal: yield expression
                ```

### Examples
```python
# use generators
wwwlog = open("access-log")
bytecolumn = (line.rsplit(None,1)[1] for line in wwwlog)
bytes = (int(x) for x in bytecolumn if x != '-')
```
* It could be a *PIPELINE*
  * Generators decouple iteration from the code that uses the results of the iteration  
  * We can plug any number of components together up front as long as they eventually produce a line sequence

### Trick

* Tuples to Dictionaries  
```python
colnames = ('host','referrer','user','datetime',
'method','request','proto','status','bytes')
log = (dict(zip(colnames,t)) for t in tuples)
#  =>>> { 'keys' : 'values'}
```

* Convert *FIELD* values (map)
```
log = field_map(log,"status", int)
log = field_map(log,"bytes",
        lambda s: int(s) if s !='-' else 0)
```

### Tailing a File
* use loop to look up the last line and yield when new line comes.
* other logics using generator could be easily plugged in.
* FOREVER run

### Applications
* TCP connections / UDP socket messaging

```python
import socket
def receive_connections(addr):
 s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
 s.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
 s.bind(addr)
 s.listen(5)
 while True:
 client = s.accept()
 yield client
####
for c,a in receive_connections(("",9000)):
 c.send("Hello World\n")
 c.close()
```
UDP
```python
import socket
def receive_messages(addr,maxsize):
 s = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
 s.bind(addr)
 while True:
 msg = s.recvfrom(maxsize)
 yield msg

for msg, addr in receive_messages(("",10000),1024):
 print msg, "from", addr
```

* I/O multiplexing
```python
# Generating I/O events on a set of sockets
import select
def gen_events(socks):
 while True:
 rdr,wrt,err = select.select(socks,socks,socks,0.1)
 for r in rdr:
    yield "read",r
 for w in wrt:
    yield "write",w
 for e in err:
    yield "error",e
```

```python
clientset = []
def acceptor(sockset,addr):
 for c,a in receive_connections(addr):
 sockset.append(c)
acc_thr = threading.Thread(target=acceptor,
 args=(clientset,("",12000))
acc_thr.setDaemon(True)
acc_thr.start()
for evt,s in gen_events(clientset):
 if evt == 'read':
 data = s.recv(1024)
 if not data:
 print "Closing", s
 s.close()
 clientset.remove(s)
 else:
 print s,data
```

* QUEUE
```python
def consume_queue(thequeue):
 while True:
 item = thequeue.get()
 if item is StopIteration: break
 yield item
####
import Queue, threading
def consumer(q):
 for item in consume_queue(q):
   print "Consumed", item
   print "Done"
in_q = Queue.Queue()
con_thr = threading.Thread(target=consumer,args=(in_q,))
con_thr.start()
for i in xrange(100):
  in_q.put(i)
in_q.put(StopIteration)
```

### Multiple processes
* Broadcasting
*Consume a gnerator and send items to a set of consumers*

```python
def broadcast(source, consumers):
 for item in source:
  for c in consumers:
   c.send(item)
```
* Network consumer
```python
import socket,pickle
class NetConsumer(object):
 def __init__(self,addr):
    self.s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    self.s.connect(addr)
 def send(self,item):
    pitem = pickle.dumps(item)
    self.s.sendall(pitem)
 def close(self):
    self.s.close()
```
Consumer Thread

```python
import Queue, threading
class ConsumerThread(threading.Thread):
 def __init__(self,target):
    threading.Thread.__init__(self)
    self.setDaemon(True)
    self.in_queue = Queue.Queue()
    self.target = target
 def send(self,item):
    self.in_queue.put(item)
 def generate(self):
    while True:
        item = self.in_queue.get()
        yield item
 def run(self):
    self.target(self.generate())  # use a generator as parameter
```

```python
# Usage
def find_404(log):
 for r in (r for r in log if r['status'] == 404):
 print r['status'],r['datetime'],r['request']
def bytes_transferred(log):
 total = 0
 for r in log:
 total += r['bytes']
 print "Total bytes", total
c1 = ConsumerThread(find_404)
c1.start()
c2 = ConsumerThread(bytes_transferred)
c2.start()
lines = follow(open("access-log")) # Follow a log
log = apache_log(lines) # Turn into records
broadcast(log,[c1,c2]) # Broadcast to consum
```

* **Multi source**  by using **Multiplexing Generators**
```python
def gen_multiplex(genlist):
    item_q = Queue.Queue()
    def run_one(source):
        for item in source: item_q.put(item)
    def run_all():
        thrlist = []
        for source in genlist:
            t = threading.Thread(target=run_one,args=(source,))
            t.start()
            thrlist.append(t)
        for t in thrlist: t.join()
        item_q.put(StopIteration)

    threading.Thread(target=run_all).start()
    while True:
        item = item_q.get()
        if item is StopIteration: return
        yield item
```

### Important tricks

* Single-argument function is easy to turn into a generator function
```
def generate(func):
 def gen_func(s):
    for item in s:
        yield func(item)
 return gen_func
```

* *Shutting down*
By using .close() <= shut down in the middle
and and GenratorExit will be raised in generator function

try: .... yield  
except GeneratorExit:
    print "Shut down"
=> using for resources clean up
=> separate threads can not call .close()
=> Cannot shutdown by using signals

The only way is set an instrument with a flag or check
```python
def follow(thefile,shutdown=None):
 thefile.seek(0,2)
 while True:
 if shutdown and shutdown.isSet(): break
 line = thefile.readline()
 if not line:
 time.sleep(0.1)
 continue
 yield line

import threading,signal
shutdown = threading.Event() ## <-That's the point
def sigusr1(signo,frame):
 print "Closing it down"
 shutdown.set()  ## <- That's the point
signal.signal(signal.SIGUSR1,sigusr1)
lines = follow(open("access-log"),shutdown)  # <- HERE
for line in lines:
 print line,
```


### Next part -> Co-routines / reverse-generator

* Think of them as "receivers" or "consumer"
(yield) / .send()  
* **Co-routines could be used into co-operative multitasking, concurrent programming without using threads !!**  
* Pay attention about debugging (using wrapper genertor function for yield what you want to print out)

=====

Please refer to [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
