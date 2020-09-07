+++
comments = false
date = 2020-09-07T00:00:00Z
layout = "post"
publish = true
tags = []
title = "OVH Serial over LAN (SOL)"

+++
Its been a while since I've worked with physical servers and typically when something happened I could pretty quickly get physical access to the box if things got complicated.

But when you buy or lease servers from a provider like OVH.  Those servers might be sitting half way around the world.  So if something goes wrong you need to some how virtually be in front of the keyboard.

OVH has a few different ways to access your servers console to interact with the machine directly.

* From a Java applet (KVM)

This way while works across all of the servers.. requires Java Runtime.. Yuck.  Not to mention it isn't necessarily going to work on every machine.

* Via your web browser (KVM)

This way is by far the best.  But seems for some reason, i'm not completely sure why.. its not available for all of their servers. So its not really a consistent option

* Serial over LAN (SOL)

This way is probably the most easy to work with.  Can access over ssh from any device that can ssh.  Which is pretty much anywhere, so very appealing.

## The Problem

***

Out of the box when you connect all you get is:

        Could not chdir to home directory /home/ipmi: No such file or directory
        
                                OVH
                    IPMI Serial Over Lan gateway
        
                    IPMI SOL on [Omitted ;-)].us
        
        ipmiutil ver 2.93
        isol ver 2.93
        Opening lan connection to node 10.245.29.163 ...
        Connecting to node 10.245.29.163
        -- BMC version 2.00, IPMI version 2.0
        Opening lanplus connection to node 10.245.29.163 ...
        [SOL session is running, use '~.' to end, '~?' for help.]

Hitting enter here should drop you to a login prompt to login to the console. But nothing happens. Turns out this is because out of the box on OVH the linux kernel isn't spinning up a console on the serial port.

## Solution

To make the linux kernel output console output and open a tty on the serial port as needed by Serial Over Lan you need to ssh into your server and modify your grub settings to instruct the kernel to open a tty on that serial port.  Also to tell it the baud rate to use.  The [arch wiki](https://wiki.archlinux.org/index.php/Working_with_the_serial_console) is useful here.

From what I can tell this is typically /dev/ttyS1.  But if it doesn't work for you try /dev/ttyS0.

In the bios you can confirm the baud/bit rate.  But in my case it was: 115200 and looking around that seems to be the most common.

First hop into: /etc/default/grub

and add:

        GRUB_CMDLINE_LINUX_DEFAULT="nomodeset console=tty0 console=ttyS1,115200"
        GRUB_TERMINAL="console serial"
        GRUB_SERIAL_COMMAND="serial --unit=1 --speed=115200 --word=8 --parity=no --stop=1"

This will both set the default linux command args, but also will let you interract with grub over the serial console as well. 

If you do want to be able to interact you will need to add to /boot/grub/menu.lst

    serial --unit=1 --speed=115200
    terminal --timeout=5 serial console

To give you a chance to interact over serial console to grub.

To make sure takes effect:

        sudo update-grub

If you want to verify take a peak in /boot/grub/grub.cfg and see if the kernel options are there.

If you would like to also be able to interact with bios and everything after POST you can also hop into bios via one of the KVM solutions and set this setting to enabled:

![](/images/2020-09-:day/screen-shot-2020-09-07-at-14-28-07.png)

![](/images/2020-09-:day/screen-shot-2020-09-07-at-14-29-20.png)

I want to thank the author of this blog post for setting me on the right track: http://divideoverflow.com/2016/07/Configuring_IPMI_on_OVH_server/ 