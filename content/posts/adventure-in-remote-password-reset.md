+++
title = "Adventures of the Remote Geek: Password reset"
date = 2022-06-16T00:00:00Z
layout = "post"
publish = true
comments = false
tags = []

+++

A few years back I wrote about [Formating a machine over ssh](https://geekgonecrazy.com/2012/08/20/format-machine-over-ssh/).  That was kind of crazy.  But with ssh access to a linux box anything is possible.

Time to take it to the next level. Time to earn the name Geek Gone Crazy.  Lets set the stage:

I have an elderly family member that lives a little over 8 hours away.  I've become the defacto IT guy.

Anyways so the story begins.  I was contacted with the case of "My password stopped working and I can't login to my laptop to get some work done".

We tried all the typical passwords and variations of them and nothing worked.

So then I went into research mode. How does a person reset a password on windows 10 now days?  I don't use windows at all now days and haven't for a few years.

Found that there are several approaches:
### Password reset via Security questions 

Apparently we didn't have those set. Looks like introduced on Windows 10 Version 1803.  Not sure if we originally installed before then or what.. but didn't have this.. so wasn't an option.

### Password Reset via Password Reset Disk or USB

We hadn't prepared one.  Awesome.

### Reset Password with Microsoft Account

This was a local account.. not an option

### Reset Password in Safe Mode

I honestly don't know what happened here.  I didn't have windows 10 in front of me and tried to navigate doing this based on screenshots on the internet and description of how things were reacting with my remote eyes and hands.

It didn't work.

### Several other "Password reset utilities"

But honestly never used any of them.  And if I was in person sitting in front of the computer I probably would have tried.  But with remote eyes and hands.. It was a no go.

### Offline Windows Password & Registry Editor, Bootdisk / CD

More info about it can be found here: https://pogostick.net/~pnh/ntpasswd/

This one had a lot of potential. Looked like it would do exactly what I needed.

So had a flash drive inserted to another laptop in the house.  And used teamviewer to connect.  Which literally only has their computers on it.  They fire it up any time they need me to help.

From there I downloaded a super lightweight tool called [Rufus](https://rufus.ie/) and put the ISO on the flash drive.

I then attempted to walk through the process of plugging in an hitting F12 to boot from USB.  After a few tries we did but the kernel kept crashing.

Identified via [reddit post](https://www.reddit.com/r/techsupport/comments/7j4z5t/pogostick_ntpasswd_error_kernel_panic_not_syncing/) that others had this and changing boot options had worked.  We tried every combo.  Then we went back to the other computer and flashed an older version hoping that would work.

Just to point out here.. this took a long time. Trying to relay information being discovered on the internet and translating that to the situation.. and trying to get remote hands to do the same.  If you're reading this.. you probably already know.

In the end.. Remote eyes and remote hands. If only I could just ssh in and do it my self...

## Use my own eyes and hands

I don't do this as my day job.  I can imagine if I worked in a support desk i'd probably have a procedure documented on exactly how to walk them through it step by step and get it done.  But thats not what I do. 

What I really needed was to be able play to my strengths.  I'm a problem solver. If I can see it with my own eyes and use my own hands.. I can try, process the information and iterate a **lot** faster!

So what we need is an ssh connection directly to that machine with the power of linux at my disposal.

### Exposing SSH?

How would we do that?  Turns out with ssh you can forward local ports remotely.  As well as remote ports locally.

So if we establish a connection out to a remote vps on the internet and expose our ssh port from the laptop... We should be able to SSH back from the vps to the laptop.

```
ssh -R 4022:localhost:22 -N user@my-vps
```

Great!  But.. do we really want to ask them to type a password or to use password based ssh?  Nah..

So we generate an ssh key and put the public key on the vps and have the private key for the laptop

```
ssh -i ~/path-to-ssh-key -R 4022:localhost:22 -N user@my-vps
```

### Wait.. Are we going to ask for this all to be typed?

Of course not.  That would be crazy.

In the past (like 10 years ago) I had created a livecd that I had actually shipped to them.  But then I made the mistake of losing my copy and all original files that I used to make it up.  

So I went on the duckduckgo path of trying to figure out now days what the easiest way to package up a livecd was.

There are a lot of tools for creating images to install in automated way on machines.  Packer etc.. 

But not a lot for making livecds. There are a few gui options.. but I wanted to be able to just quickly put it together in a more repeatable way so this was more future proof.

I then found this guide: https://slai.github.io/posts/customising-ubuntu-live-isos-with-docker/

Perfect!

### Creating livecd

The important aspects of the livecd would be:
1. Light weight.  So need to use the server iso as the base
2. Automatically connect to the internet
3. Automatically run the ssh tunnel
4. Reconnect if disconnected
5. Have some preinstalled tools like chntpw that I discovered thanks to the first ISO tried.  Also things like openssh-server ntfs-3g would be good to have
6. An ssh key baked in that could be used for connecting up.. but also that could be used to connect from VPS back

Nice to have:
1. Don't start up the installer it just adds confusion
2. Customize grub entry so it doesn't say its installing
3. Include some text above the login prompt to communicate to the user

From that we end up with: https://github.com/geekgonecrazy/remote-recover-livecd

### Lets go!

During this time of build, try in vm, iterate... I ran into an apparently common problem.  Teamviewer decided I was using it for commercial use.  Meaning sessions lasted ~1 minute in length and would randomly "reset".  Then it would claim I had to wait 1 minute (it provided the timestamp of when) before I could try again.  I'd wait 2 minutes (just in case of clock not being sync'd) try again and it'd say I had to wait another minute.  After about 5-6 minutes I could finally get connected and then quickly do something for 1 more minute.

**Extremely frustrating**

If I was using this for commercial purposes and I had a bunch of clients I was providing services via teamviewer.. I would have succumed to just paying.  How ever since I wasn't making a dime off of any usage from it and fitting in the "non-commercial usage".. I went shopping for another tool.  Even if I had to pay for it.

Ended up thanks to [@amolith@nixnet.social](https://nixnet.social/objects/c34e67c2-aa8f-45bc-916d-1b66f76b84f8) discovering an open source option called: [rustdesk](https://rustdesk.com)

Which I painfully managed to install over the course of 30 minutes or so... Thanks to constant disconnects and waiting 5 minutes between 1 minute bursts.

It works flawlessly.  10/10 recommend.  Plus you can self host the server portion if you want.

From there used Rufus to put on the USB and then had plug ethernet into laptop along with the fresh USB.

Then from vps:

```
ssh -i pre-shared-key -p 4022 root@localhost
```

Presto!  I had ssh access to the laptop!


## Finally Resetting password

From there mounted the hard drive with a quick

```
mkdir /mnt/laptop
mount /dev/sda2 /mnt/laptop
```

Using `fdisk -l` to determine the drive and partition.

Then:

```
cd /mnt/laptop/Windows/System32/config
```

Here is where all of the registry hive files are for windows.

Now we use the chntpwd util we pre-installed
```
chntpw –l SAM
```

From here you follow the prompts to reset the password.  Or in my case I went the path of just blanking the password out.
