# Talk notes

## Introduction
Hi guys. Today I'm going to tell you about python thin client for
Apache Ignite. Few weeks ago Apache Ignite community have released new
version 0.4.0 after almost two and a half years since previous release.
A brand new version contains a lot of improvements, bug fixes and new functionality. I believe that nowadays the python thin client is more robust and definitely is production ready.

### About me slide
By the way, few words about me. My name is Ivan Dashchinsky, I'm an apache ignite committer, the python thin client maintainer. Here you can see my contacts (e-mail, telegram and github profile)

### Whats new slide
Let's talk briefly words about what's new in the 0.4 version.
Firstly, we have made a lot of performance improvements. In some scenarios, operations are performed faster by several orders of magnitude. Few slides later I will demonstrate benchmark figures and charts. 

Secondly, we have implemented partition awareness support. In few words, partition awareness is simply redirecting ignite cache requests to a cluster's server node that contains primary copy of data (so-called affinity node) This request can eliminate unnecessary network hop between ignite server nodes.

Also, we have implemented asyncio support. The async client is fully functional and performs well.

And one more thing, we have implemented ability to change cluster state (activate-deactivate). This is very useful feature, especially for persistent cluster.

## Sync client example
Let's see simple example. I will show you briefly simple code snippet, and then I will switch to jupyter notebook and run some code interactively.

### Student definition
Let's define Student class.
### Simple example
Let's put some data and query it.
### Notebook example

## Benchmarks
### Benchmarks environment
### How have we achieved this?
Firstly, we optimize memory consumption by using BytesIO, memoryview and bytearrays instead of bytes string, that were used in previous version.
Also, a hashcode calculation was implemented in C, and this is the cause of huge performance improvement of BinaryObject's put.
This is because during serialization of BinaryObject, we must compute hashcode of resulting bytearray.

## Partition awareness.
As I have said earlier, partition awareness is redirecting ignite cache requests to a cluster's server node 
that contains primary copy of data (so-called affinity node) This request can eliminate unnecessary network
hop between ignite server nodes. You can see details on sequence diagram.

Firstly, client send request to random node.
During processing of the first cache response, client receives topology version.
Client compare this version to the cached one and requests new partition mappings corresponding to actual topology version.
By computing hashcode of a key, client determines the partition and routes the request to a specific node, that contains
primary copy of the partition to which key-value pair belongs to.

## Let's see some benchmark's figures and diagrams.

## Asyncio version
There is a well-known concurrency problem in CPython. I suppose every python developer knows about GIL.
When thread that executes python bytecode and works with python datastructures must acquire GIL. GIL is released
when thread do IO routines. GIL must be required in order to process incoming data from IO. So GIL is an obvious bottleneck 
for common applications and `multithreading` is mostly useless. 

Is there any solution?
If your workload is mostly IO bound (and the most web applications are), the solution exists -- the event-loop.
It is still a single thread and it utilizes non-blocking network IO facility of OS.
Traditional event loop is callback based.  It schedules IO tasks one by one
and executes callbacks when data arrives. This computational model can be effectively replaced with coroutines. 
The scheduler executes coroutines one by one, suspend the coroutine when IO task is scheduled, executes the next one
and resume the first when data is arrived.

Of course, this machinery is not cost free. There is a performance penalty. But if do many IO tasks concurrently with
a lot of network connections, python processing code is not CPU bound, and you don't block event-loop, you can effectively 
process a lot of request.

### After benchmarks
As you can see, partition awareness eliminates not only unnecessary network hops, but utilizes multiple network connections.
Requests can be done concurrently to many nodes so latency is better than in sync version.

