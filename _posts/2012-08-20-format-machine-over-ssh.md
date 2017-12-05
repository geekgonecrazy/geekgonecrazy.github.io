---
layout: post
title:  "Format Machine over ssh"
date:   2012-08-20
comments: true
publish: true
tags: [tips,linux]
---
Lets say you have data on a remote linux machine, and for what ever reason you need to format the drive.  Like for instance you decide to switch VPS hosts.  If you just let them delete it, you have no way of knowing what really happens to your data.  So providing you have ssh access here is a way to eliminate your data.  

<!--excerpt-->

One of the beauties of linux is that everything is represented as a file or folder.  So if you need to store something in the RAM you can do so by copying it to: `/dev/shm`.  

The `/proc` directory holds information about your system, kernel, configuration parameters, processes, etc.  We need access to this.  Once we have formatted the system, we will no longer be able to use our normal methods of shutting down.

So first off you will want to become root.  If you don't know how to do this.. You probably shouldn't be trying to do this.  

Then we need to mount `/proc` in `/dev/shm` so we will have access after we have formatted the drive.

```Bash
mkdir /dev/shm/proc
mount /proc /dev/shm/proc -t proc
```

If you need any other tools like ls copy them from /bin into `/dev/shm` because they will be gone.  

Now you need to cd into `/dev/shm`.  Again once you have formatted the drive you won't be able to change into it, because / doesn't exist.  

Now you can choose your method of destruction.  Some you could use:  

* rm -rf --no-preserve-root /
* dd if=/dev/urandom of=/dev/sda
* shred -n1 -z -v  /dev/sda (you can replace the number with how ever many passes you wish to run)

I personally used shred.

Once this is done its time to put the machine to rest.

```Bash
echo 1 > proc/sys/kernel/sysrq
echo o > proc/sysrq-trigger
```

And next time it starts up you will have nada!  Everything is gone.  You might want to fire up a VM and give this a go before you go and try it.
