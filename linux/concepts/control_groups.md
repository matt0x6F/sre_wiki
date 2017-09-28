Control Groups (cgroups)
======

## Intro
---

Control Groups are another key component of Linux Containers. They implement resource accounting and limiting. They provide many useful metrics, but they also help ensure that each container gets its fair share of memory, CPU, disk I/O; and, more importantly, that a single container cannot bring the system down by exhausting one of those resources.

So while they do not play a role in preventing one container from accessing or affecting the data and processes of another container, they are essential to fend off some denial-of-service attacks. They are particularly important on multi-tenant platforms, like public and private PaaS, to guarantee a consistent uptime (and performance) even when some applications start to misbehave.

Control Groups have been around for a while as well: the code was started in 2006, and initially merged in kernel 2.6.24.

## Implementing on RedHat/CentOS/Fedora

Control groups (cgroups) are a Linux kernel feature that allows you to specify how the kernel should allocate specific resources to a group of processes. With cgroups you can specify how much CPU time, system memory, network bandwidth, or combinations of these resources can be used by the processes residing in a certain group.

The advantage of control groups over nice or cpulimit is that the limits are applied to a set of processes, rather than to just one. Also, nice or cpu limit only limit the CPU usage of a process, whereas cgroups can limit other process resources.

By judiciously using cgroups the resources of entire subsystems of a server can be controlled. For example in CoreOS, the minimal Linux distribution designed for massive server deployments, the upgrade processes are controlled by a cgroup. This means the downloading and installing of system updates doesn’t affect system performance.

To demonstrate cgroups, we will create two groups with different CPU resources allocated to each group. The groups will be called `cpulimited` and `lesscpulimited`.

The groups are created with the `cgcreate` command like this:

```
sudo cgcreate -g cpu:/cpulimited
sudo cgcreate -g cpu:/lesscpulimited
```

The `-g cpu` part of the command tell cgroups that the groups can place limits on the amount of CPU resources given to the processes in the group. Other contollers `includecpuset`, `memory`, and `blkio`. The `cpuset` controller is related to the cpu controller in that it allows the processes in a group to be bound to a specific CPU, or set of cores in a CPU.

The cpu controller has a property known as cpu.shares. It is used by the kernel to determine the share of CPU resources available to each process across the cgroups. The default value is 1024. By leaving one group (lesscpulimited) at the default of 1024 and setting the other (cpulimited) to 512, we are telling the kernel to split the CPU resources using a 2:1 ratio.

To set the cpu.shares to 512 in the cpulimited group, type:

`sudo cgset -r cpu.shares=512 cpulimited`

To start a task in a particular cgroup you can use the cgexec command. To test the two cgroups, start matho-primes in the cpulimited group, like this:

`sudo cgexec -g cpu:cpulimited /usr/local/bin/matho-primes 0 9999999999 > /dev/null &`

If you runtopyou will see that the process is taking all of the available CPU time.


This is because when a single process is running, it uses as much CPU as necessary, regardless of which cgroup it is placed in. The CPU limitation only comes into effect when two or more processes compete for CPU resources.

Now start a second matho-primes process, this time in the lesscpulimited group:

`sudo cgexec -g cpu:lesscpulimited /usr/local/bin/matho-primes 0 9999999999 > /dev/null &`

The top command shows us that the process in the cgroup with the greater cpu.shares value is getting more CPU time.


Now start another matho-primes process in the cpulimited group:

`sudo cgexec -g cpu:cpulimited /usr/local/bin/matho-primes 0 9999999999 > /dev/null &`

Observe how the CPU is still being proportioned in a 2:1 ratio. Now the twomatho-primestasks in the cpulimited group are sharing the CPU equally, while the process in the other group still gets more processor time.

You can read the full control groups documentation from Red Hat (which applies equally to CentOS 7).+
