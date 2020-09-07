+++
comments = false
date = 2020-09-07T00:00:00Z
draft = true
layout = "post"
publish = false
tags = []
title = "OVH Serial over LAN (SOL)"

+++
OVH has a few different ways to access your servers console to interact with the machine directly.

From a Java applet (KVM)

Via your web browser (KVM)

***

Out of the box when you connect all you get is:

```
    Could not chdir to home directory /home/ipmi: No such file or directory
    
                            OVH
                IPMI Serial Over Lan gateway
    
                IPMI SOL on ns1000129.ip-51-81-107.us
    
    ipmiutil ver 2.93
    isol ver 2.93
    Opening lan connection to node 10.245.29.163 ...
    Connecting to node 10.245.29.163
    -- BMC version 2.00, IPMI version 2.0
    Opening lanplus connection to node 10.245.29.163 ...
    [SOL session is running, use '~.' to end, '~?' for help.]
```

Hitting enter here should drop you to a login prompt to login to the console. But nothing happens. Turns out this is because out of the box the linux kernel isn't spinning up a terminal on the serial port.

To forward the linux kernel output to the serial console you need to ssh into your server and modify your grub settings to instruct the kernel to open a tty on that serial port.  Also to tell it the baud rate to use.

From what I can tell this is typically /dev/ttyS1.  But if it doesn't work for you try /dev/ttyS0.

In the bios you can confirm the baud/bit rate.  But in my case it was: 115200 and looking around that seems to be the most common.

First hop into: /etc/default/grub

and add:
```
    GRUB_CMDLINE_LINUX_DEFAULT="nomodeset console=tty0 console=ttyS1,115200"
    GRUB_TERMINAL="console serial"
    GRUB_SERIAL_COMMAND="serial --unit=1 --speed=115200 --word=8 --parity=no --stop=1"
```

This will both set the default linux command args, but also will let you interract with grub over the serial console as well.

If you do want to be able to interact you will need to add to /boot/grub/menu.lst

```
serial --unit=0 --speed=9600
terminal --timeout=5 serial console
```

To give you a chance to interact over serial console to grub.

To make sure takes effect:

```
    sudo update-grub
```

If you want to verify take a peak in /boot/grub/file.. and see if the kernel options are there.

If you would like to also be able to interact with bios and boot process you can also hop into bios and set this setting:



I want to give credit and thank the author of this blog post: http://divideoverflow.com/2016/07/Configuring_IPMI_on_OVH_server/ Definitely set me on the right track