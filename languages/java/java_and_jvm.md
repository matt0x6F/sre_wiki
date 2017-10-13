Java and the JVM
=====

## Intro to the JVM
---
What does the JVM do? The Java Virtual Machine has two primary jobs:

* Executes Code
* Manages Memory

This includes allocating memory from  the OS, managing Java allocation including heap compaction and removal of garbaged objects

Besides the above, the JVM also does stuff like managing monitors.

## Java Theory & Garbage Collection
---

An object is created in the heap and is garbage-collected after there are no more references to it. Objects cannot be reclaimed or freed by explicit language directives. Objects become garbage when they’re no longer reachable from the root set (e.g static objects)

The sequence of the garbage collection process is as follows:

1. Root set tracing and figure out objects that are not referenced at all.

2. Put the garbage objects from above in finalizer Q

3. Run finalize() of each of these instances

4. Free memory

## Understanding Thread States
---
A thread can be in one of the five states. According to sun, there is only 4 states in thread life cycle in java new, runnable, non-runnable and terminated. There is no running state.

But for better understanding the threads, we are explaining it in the 5 states.
The life cycle of the thread in java is controlled by JVM. The java thread states are as follows:

* New
* Runnable
* Running
* Non-Runnable (Blocked)
* Terminated

[gimmick:yuml]([New]->[Runnable], [Runnable]<-[Non-Runnable], [Runnable]->[Running],[Running]->[Non-Runnable],[Running]->[Terminated])

**1. New**

The thread is in new state if you create an instance of Thread class but before the invocation of start() method.

**2. Runnable**

The thread is in runnable state after invocation of start() method, but the thread scheduler has not selected it to be the running thread.

**3. Running**

The thread is in running state if the thread scheduler has selected it.

**4. Non-Runnable (Blocked)**

This is the state when the thread is still alive, but is currently not eligible to run.

**5. Terminated**

A thread is in terminated or dead state when its run() method exits.

## Check Java process heapsize
---
In Java startup set heapsize, but we are not sure whether the entry into force. You can get the size heapsize by jstat, some small script below, PID for the process java process id

`jstat -gccapacity PID |  awk '{print ($3+$9+$14)/1024}'  | tail -1`

_Units are MB_
