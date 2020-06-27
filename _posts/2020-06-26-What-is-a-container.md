---
layout: post
title: What is a Container?
categories: Container
tags: Container
---

* content
{:toc}



    

## Processes

Containers are just normal Linux Processes with additional 
configuration applied. Launch the following Redis container so we can 
see what is happening under the covers.

`docker run -d --name=db redis:alpine`

The Docker container launches a process called `redis-server`. From the host, we can view all the processes running, including those started by Docker.

`ps aux | grep redis-server`

Docker can help us identify information about the process including the PID (Process ID) and PPID (Parent Process ID) via `docker top db`

Who is the PPID? Use `ps aux | grep <ppid>` to find the parent process. Likely to be Containerd.

The command `pstree` will list all of the sub processes. See the Docker process tree using `pstree -c -p -A $(pgrep dockerd)`

As you can see, from the viewpoint of Linux, these are standard 
processes and have the same properties as other processes on our system.

## Process Directory

Linux is just a series of magic files and contents, this makes it fun
 to explore and navigate to see what is happening under the covers, and 
in some cases, change the contents to see the results.

The configuration for each process is defined within the `/proc` directory. If you know the process ID, then you can identify the configuration directory.

The command below will list all the contents of /proc, and store the Redis PID for future use.

`DBPID=$(pgrep redis-server)
echo Redis is $DBPID
ls /proc`

Each process has it's own configuration and security settings defined within different files.
`ls /proc/$DBPID`

For example, you can view and update the environment variables defined to that process.
`cat /proc/$DBPID/environ`

`docker exec -it db env`

          
        

## Namespaces

One of the fundamental parts of a container is namespaces. The 
concept of namespaces is to limit what processes can see and access 
certain parts of the system, such as other network interfaces or 
processes.

When a container is started, the container runtime, such as Docker, 
will create new namespaces to sandbox the process. By running a process 
in it's own Pid namespace, it will look like it's the only process on 
the system.

The available namespaces are:

Mount (mnt)

Process ID (pid)

Network (net)

Interprocess Communication (ipc)

UTS (hostnames)

User ID (user)

Control group (cgroup)

More information at [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)

## Unshare can launch "contained" processes.

Without using a runtime such as Docker, a process can still operate within it's own namespace. One tool to help is unshare.

`unshare --help`

With unshare it's possible to launch a process and have it create a 
new namespace, such as Pid. By unsharing the Pid namespace from the 
host, it looks like the bash prompt is the only process running on the 
machine.
`sudo unshare --fork --pid --mount-proc bash
ps
exit`

## What happens when we share a namespace?

Under the covers, Namespaces are inode locations on disk. This allows
 for processes to shared/reused the same namespace, allowing them to 
view and interact.

List all the namespaces with `ls -lha /proc/$DBPID/ns/`

Another tool, NSEnter is used to attach processes to existing Namespaces. Useful for debugging purposes.

`nsenter --help`

`nsenter --target $DBPID --mount --uts --ipc --net --pid ps aux`

With Docker, these namespaces can be shared using the syntax `container:<container-name>`. For example, the command below will connect nginx to the DB namespace.

`docker run -d --name=web --net=container:db nginx:alpine`

`WEBPID=$(pgrep nginx | tail -n1)`

`echo nginx is $WEBPID`

`cat /proc/$WEBPID/cgroup`

While the net has been shared, it will still be listed as a namespace.

`ls -lha /proc/$WEBPID/ns/`

However, the net namespace for both processes points to the same location.

`ls -lha /proc/$WEBPID/ns/ | grep net`

`ls -lha /proc/$DBPID/ns/ | grep net`

          
        

## Chroot

An important part of a container process is the ability to have 
different files that are independent of the host. This is how we can 
have different Docker Images based on different operating systems 
running on our system.

Chroot provides the ability for a process to start with a different 
root directory to the parent OS. This allows different files to appear 
in the root.

          
        

## Cgroups (Control Groups)

CGroups limit the amount of resources a process can consume. These 
cgroups are values defined in particular files within the /proc 
directory.

To see the mappings, run the command:

`cat /proc/$DBPID/cgroup`

These are mapped to other cgroup directories on disk at:

`ls /sys/fs/cgroup/`

## What are the CPU stats for a process?

The CPU stats and usage is stored within a file too!

`cat /sys/fs/cgroup/cpu,cpuacct/docker/$DBID/cpuacct.stat`

The CPU shares limit is also defined here.

`cat /sys/fs/cgroup/cpu,cpuacct/docker/$DBID/cpu.shares`

All the Docker cgroups for the container's memory configuration are stored within:

`ls /sys/fs/cgroup/memory/docker/`

Each of the directory is grouped based on the container ID assigned by Docker.

`DBID=$(docker ps --no-trunc | grep 'db' | awk '{print $1}')
WEBID=$(docker ps --no-trunc | grep 'nginx' | awk '{print $1}')
ls /sys/fs/cgroup/memory/docker/$DBID`

## How to configure cgroups?

One of the properties of Docker is the ability to control memory limits. This is done via a cgroup setting.

By default, containers have no limit on the memory. We can view this via docker stats command.

`docker stats db --no-stream`

The memory quotes are stored in a file called `memory.limit_in_bytes`.

By writing to the file, we can change the limit limits of a process.

`echo 8000000 > /sys/fs/cgroup/memory/docker/$DBID/memory.limit_in_bytes`

If you read the file back, you'll notice it's been converted to 7999488. 

`cat /sys/fs/cgroup/memory/docker/$DBID/memory.limit_in_bytes`

When checking Docker Stats again, the memory limit of the process is now 7.629M

`docker stats db --no-stream`

          
        

## Seccomp / AppArmor

All actions with Linux is done via syscalls. The kernel has 330 
system calls that perform operations such as read files, close handles 
and check access rights. All applications use a combination of these 
system calls to perform the required operations.

AppArmor is a application defined profile that describes which parts of the system a process can access.

It's possible to view the current AppArmor profile assigned to a process via `cat /proc/$DBPID/attr/current`

The default AppArmor profile for Docker is `docker-default (enforce)`.

Prior to Docker 1.13, it stored the AppArmor Profile in 
/etc/apparmor.d/docker-default (which was overwritten when Docker 
started, so users couldn't modify it. After v1.13, Docker now generates 
docker-default in tmpfs, uses apparmor_parser to load it into kernel, 
then deletes the file

The template can be found at [https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/profiles/apparmor/template.go](https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/profiles/apparmor/template.go)

Seccomp provides the ability to limit which system calls can be made,
 blocking aspects such as installing Kernel Modules or changing the file
 permissions.

The default allowed calls with Docker can be found at [https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/profiles/seccomp/default.json](https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/profiles/seccomp/default.json)

When assigned to a process it means the process will be limited to a 
subset of the ability system calls. If it attempts to call a blocked 
system call is will recieve the error "Operation Not Allowed".

The status of SecComp is also defined within a file.

`cat /proc/$DBPID/status`

`cat /proc/$DBPID/status | grep Seccomp`

The flag meaning are:
0: disabled
1: strict
2: filtering

          
        

## Capabilities

Capabilities are groupings about what a process or user has 
permission to do. These Capabilities might cover multiple system calls 
or actions, such as changing the system time or hostname.

The status file also containers the Capabilities flag. A process can 
drop as many Capabilities as possible to ensure it's secure.

`cat /proc/$DBPID/status | grep ^Cap`

The flags are stored as a bitmask that can be decoded with `capsh`

`capsh --decode=00000000a80425fb`


