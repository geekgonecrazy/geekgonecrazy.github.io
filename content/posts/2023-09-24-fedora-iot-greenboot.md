+++
title = "Fedora Iot, Greenboot, Systemd and of course DNS"
date = 2023-09-24T00:00:00Z
layout = "post"
publish = true
comments = false
tags = ["fedora", "systemd", "dns"]

+++

This post is mostly as a resource for future me.  But maybe it'll come in useful to someone else.

This weekend I was working on provisioning a new NAS/Media box setup after having to do a bit of hardware shuffling.  Long story.

I decided to try [Fedora iot](https://fedoraproject.org/iot/) on a whim. Fedora iot has an interesting project called [greenboot](https://github.com/fedora-iot/greenboot).  The objective of the project is to define some checks for the state of the machine.  When the machine is booting and the machine isn't configured or working as expected it reboots and tries to correct.  From what I understand it does this several times and then if unsuccessful it rolls back to the previous ostree commit.  In an edge deployment scenario where you can't always have your hands on the device and it almost assuredly will break... I can see ostree commits + this to being great to rollback when something gets broken.   Much easier than in my previous post: 

[Adventures of the geek: Remote Password Reset](https://geekgonecrazy.com/2022/06/16/adventures-of-the-geek-remote-password-reset/)

Anyways.. In this particular case i'm also installing [pi-hole](https://pi-hole.net/) on it.  This will act as the DNS server in my house.  Blocking domains related to tracking, ads etc. But also lets me do other typical internal dns stuff, block list of inappropriate domains etc.

So I started off by just tying to run pi-hole:

```
sudo podman run -d \
     --name=pihole \
     -e TZ=America/Chicago \
     -e DNSMASQ_LISTENING=all \
     -e WEBPASSWORD=password \
     -v ./etc-pihole:/etc/pihole:Z \
     -v ./etc-dnsmasq.d:/etc/dnsmasq.d:Z \
     -p 53:53/tcp \
     -p 53:53/udp \
     -p 80:80/tcp \
     --restart=unless-stopped \
     docker.io/pihole/pihole:latest
```

This started throwing erorrs about :53 already being used.

Turns out that systemd-resolved listens on :53 with a stub-listener and by default all queries goto 127.0.0.1:53 and then through systemd-resolved to upstream dns resolvers.


To disable I had to create: `/etc/systemd/resolvd.conf.d/stub-listener.conf`

With contents:

```
[Resolve]
DNSStubListener=no
```

Then of course restart systemd-resolved:

```
sudo systemctl restart systemd-resolved
```

This let me start up pi-hole and I went on about my business setting up volumes, moving data around etc.


As part of it a reboot was finally needed.  Oops... It wouldn't boot.. get to login screen and then promptly reboot again.  I realized this was greenboot.  I let it go a few times but it never recovered.

So I finally intervened and modified the boot arguments from grub and added:

```
init=/bin/bash
```

To drop into single user mode.

I poked around and finally found: `/usr/lib/greenboot/check/required.d/01_repository_dns_check.sh`

This made me realize what had happened.  pi-hole wasn't up at this point and systemd-resolved is no longer running the stub.  So this check was failing.

```
[aaron@super-terribly-named-box ~]$ dig google.com
;; communications error to ::1#53: connection refused
;; communications error to ::1#53: connection refused
;; communications error to ::1#53: connection refused
;; communications error to 127.0.0.1#53: connection refused

; <<>> DiG 9.18.17 <<>> google.com
;; global options: +cmd
;; no servers could be reached
```

I found this issue: https://github.com/fedora-iot/greenboot/issues/113#issuecomment-1689929196

The suggestion is simple.  Disable the service.  Problem was I couldn't even get logged in fast enough over ssh or console access to do this.

In single user mode systemd isn't running of course so finding the unit file to quickly mask it didn't work.  And at the time I didn't remember how to do it manually with ostree involved anyways.

For the record: `/usr/lib/systemd/system/greenboot-healthcheck.service` but still the problem of it being read only because of ostree and being in single user mode where rpm-ostree did not seem to work like I was used to.  This is a subject i'll be diving into at a later date.


I looked for a way via boot args to turn off greenboot.  They don't have any.  But then I found:

```
systemd.mask=greenboot-healthcheck.service
```

Can read a bit more [here](https://man.archlinux.org/man/systemd-debug-generator.8.en)

By adding this to the boot args it will mask that service from starting during that session.

From here then it was as simple as: 

```
sudo systemctl mask greenboot-healthcheck.service
```

Now to go on to evaluate greenboot further.  Seems like a cool idea.

`systemd.mask` bootarg