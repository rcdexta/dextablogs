---
layout: post
title: "What's the deal with Ruby GC and Copy-on-write"
date: 2013-01-02 21:06
comments: true
categories:
---

This post aims at answering the following questions:

* What is Unix Copy-on-write (COW)
* Why is current version of ruby (1.x.x) not COW friendly
* How does the GC packaged with Ruby 2.0 fix that

## Holy COW

The fork functionality in Unix systems uses an optimization strategy where memory is shared between the parent and the child processes. The shared memory is maintained till either the parent or one of the children modify their copy of the resource. At that point, a true private copy is created to prevent the changes from being visible to other processes. The primary advantage is that if any of the processes do not make any modifications, no private copy needs to be ever created. This is called Copy-on-write (COW) technique.

{% codeblock fork_process_shared.rb %}
shared_array = [1,2,3,4,5]

if fork
  #will be executed by parent process
  parent_array = [6,7,8]
else
  #will be executed by child process
  child_array = [9,10,11]
end
{% endcodeblock %}

If you are not familiar with Ruby fork, a call to `fork` creates a new process. In the above example, the code inside the if block will be executed by the parent process and else block will be executed by the child process (In the child process fork returns nil). A snapshot of how resources will be shared in memory is shown below.

{% img left /images/2013-01-02/shared_pool.png %}

As can be seen above, the _shared_array_ is maintained as a common resource between parent and child processes. When _shared_array_ is modified by the child process, as copy-on-write implies, a private copy is created so that it is not visible to the other process. This is illustrated below.

{% codeblock fork_process_dirty.rb %}
shared_array = [1,2,3,4,5]

if fork
  #will be executed by parent process
  parent_array = [6,7,8]
else
  #will be executed by child process
  shared_array << 5
  child_array = [9,10,11]
end
{% endcodeblock %}

{% img left /images/2013-01-02/private_copy.png %}

## Mark and Sweep objects

The advantages of Copy-on-write cannot be leveraged between multiple ruby processes due to the inherent nature of the way the garbage collector works in Ruby. The garbage collector uses a simple [Mark and Sweep](http://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29#Na.C3.AFve_mark-and-sweep) algorithm to identify unused objects. In simplest terms

> Each object in memory has a flag (typically a single bit) reserved for the garbage collector. Starting from the root-set, all objects that can be accessed are recursively traversed and marked as being 'in-use'. At the end of the cycle, the GC sweeps all objects that have not been marked and restores free space for future objects.

The way ruby creates objects, the GC flag or reserved bit is stored in the object itself. So, as you would have guessed by now, when the GC runs in one of the processes, the GC flag would be modified in all the objects even if they are present in the shared pool. Now, the OS would sense this and trigger a copy-on-write making private copies of the objects in each child's memory space. 

This is the reason why Ruby 1.8 or 1.9 is not COW friendly :(

## GC Bit and objects should keep distance

The most sensible thing to do would be to pull out the GC flag (it's actually called FL_MARK bit) from objects and maintain them separately. And this is exactly what Ruby 2.0 claims to do. 

{% img left /images/2013-01-02/bitmap.png %}

For each heap allocated by ruby, there is a corresponding bitmap which is linked to the header of the heap. The bitmap can store 0 or 1 values effectively replacing the GC bit which was previously stored inside the objects present in the heap. This technique is called bitmap marking. So in effect, the _Mark step_ will not modify any live objects in the heap. Only the bitmap will be changed. Hence, the shared objects can remain that way until one of the processes actually modified them.

### References:

1. [Narihiro Nakamura's patch for Bitmap Marking GC](http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-core/41916)
2. [Pat Shaughnessy's excellent blog post](http://patshaughnessy.net/2012/3/23/why-you-should-be-excited-about-garbage-collection-in-ruby-2-0)