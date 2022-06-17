+++
title = "Adventures of the Geek: Remote Password Reset"
date = 2022-06-16T00:00:00Z
layout = "post"
publish = true
comments = false
tags = []

+++

Our adventure begins on a Monday evening.  I received a call from a family member that lives a bit over 8 hours away.

> I powered on my laptop today and it won't take my password

Unfortunately the laptop runs Windows 10.  I have made it a point to not use windows and haven't for last several years. Primarily using linux or osx.  Leaving me at a bit of a disadvantage.

So then I did what any self respecting computer guy does when need to find out the options. I duckduckgo'd it!

## Options to do Password Reset

Found a few options
### Password Reset via Security Questions 

Apparently these weren't set. Looks like it was introduced on Windows 10 Version 1803.  Not sure if we originally installed before then or exactly what happened.. but didn't have this.. so wasn't an option.

### Password Reset via Password Reset Disk or USB

We hadn't prepared one.  Awesome... After this is solved, making one is a must.

### Password Reset via Microsoft Account

This was a local account.. not an option.  

### Password Reset via Safe Mode

Tried this.. 

I honestly don't know what happened here.  I tried to navigate doing this based on screenshots on the internet.  Relying on my remote eyes and hands to describe what was happening.

It didn't work.


### Password Reset via Offline Windows Password & Registry Editor

More info about it can be found here: https://pogostick.net/~pnh/ntpasswd/

This one had a lot of potential. Looked like it would do exactly what I needed.

Requested my helpful remote hands find a flash drive and put in an older laptop in the house.  Used teamviewer to connect.  It was already installed this isn't the first time i've needed to help out with something.

From there I downloaded a super lightweight tool called [Rufus](https://rufus.ie/) and put the ISO on the flash drive.

With a little luck and a little trial and error.  We managed to boot from the USB.  But kept getting "[Kernel panic not syncing](https://www.reddit.com/r/techsupport/comments/7j4z5t/pogostick_ntpasswd_error_kernel_panic_not_syncing/)".  Even after trying all suggested work arounds.

This took several hours of verbally instructing these options to try, rebooting trying again..  Flashing an older version.

All with the same results.

We had spent several hours by this point.  The idea was even suggested that would make unscheduled trip to visit us.  Given the price of gas though and how rough it was for them to travel.. I really didn't want it to come to this.

We decided to call it a night.  Went to bed thinking how much easier it would be if I could just use my own eyes and hands.   I'm a problem solver. I knew that if I was able to see it with my own eyes and use my own hands.. The whole try, process and iterate can happen **a lot faster**!

## Being able to use my own eyes and hands

Sounds a bit cliché.. But it hit me in the shower Tuesday morning.

I've made livecd's before... What if I could have a livecd like the one we had tried.. but based on a more versital distro like Ubuntu.  Ubuntu typically boots up on just works about any hardware. 

While i'm at it, what if it connected back to me so I could run commands my self?  With linux at my hands.. I knew we could get it resolved.

### Connect back to me?

Turns out this is actually extremely easy.

With ssh you can expose a local port remotely with a command like this:

```
ssh -i ~/path-to-ssh-key -R 4022:localhost:22 -N user@my-vps
```

It uses an ssh key and expose port 22 on the remote host on port 4022.

### Creating your own livecd?

I'll save you from some of the discovery.  There are many ways to create linux livecds.  A few of even with a GUI.

But, I wanted a way to do it quickly and repeatable with out having to use a GUI. 

I've made some before manually but not very repeatable as it was always manually editing the squashfs and chrooting in and poking around.

I found this guide great example of loading a livecd as a docker image and then using docker to modify it

https://slai.github.io/posts/customising-ubuntu-live-isos-with-docker/

Perfect!

### Lets make it!

The important aspects of the livecd would be:
1. Lighter weight - Ubuntu Server would be the base.
2. Automatically connect to the internet
3. Automatically run the ssh tunnel back to me
4. Automatically reconnect the ssh tunnel if disconnected
5. Have some preinstalled tools
    * ntfs - i'd need to mount that partition
    * ssh server - of course
    * chntpw - most importantly in this case the tool we discovered thanks to the previous day that could modify the registry
7. Baked ssh keys for bi-directional communciation

Nice to have:
1. Don't start up the installer it would be confusing
2. Customize boot menu so it doesn't say "Install Ubuntu Server"
3. Include some text above the login prompt to communicate to the user

For more technical details see this github repo: https://github.com/geekgonecrazy/remote-recover-livecd

I set about building after my work day ended.

### Attack of the remote desktop tooling

It came time to load the iso up on the USB drive.

Teamviewer showed its true colors. 

They apparently decided since I needed to remote in two days in a row.. that I was exhibiting signs of commercial use.

Meaning apparently they start being extremely aggressive.  My first session of the evening was barely open and then abruptly ended.  I tried connecting again and it said something like:  You can't reconnect again so soon try again at: 17:55.

I waited until then and it showed another saying try again at 17:56.  I guess a clock is off somewhere or the error time frame was wrong?  It ended up being at least 5 minutes before could connect.

Here begun that pattern of wait 5 minutes.. Scramble to work for 1 minute before getting the session ripped away from me. 

**Extremely frustrating!**

I did debate paying them.  But appears they don't sell monthly.. so I'd have to shell out $400+ just to ocassionally use it to help out family.  Nope.. Sorry.

I went shopping for another tool.  Even if I had to pay for it surely there is something better out there.

I [posted about this frustration on mastodon](https://fosstodon.org/@geekgonecrazy/108478535293237696)

Big thanks to @amolith@nixnet.social for the suggestion

![](https://user-images.githubusercontent.com/51996/174218187-0463babe-38af-4f3f-bf24-5eea9cfd5b30.png)

[Rustdesk](https://rustdesk.com) was perfect!

30 minutes or so later... Thanks teamviewer.. I had rustdesk installed and was able to move forward.

If you are using teamviewer I highly recommend switching to RustDesk.  You can even self host the server portion if you want.

From there used Rufus to put on the USB and then had them plug ethernet into laptop along with the fresh USB.

We booted.. and it didn't connect back up.

![](https://media.giphy.com/media/xT5LMzIK1AdZJ4cYW4/giphy.gif)

We had to call it a night.  This ended night 2.

## One more try

Wednesday came and I quickly realized it was netplan.. I think because I had removed the installer squashfs not thinking much about it.. It must have been initalizing a netplan file.

At the end of the work day I made some changes and built the iso again.  Quick rustdesk and rufus later...

USB inserted and there it was!

I finally had access and the process of could finally begin of resetting the password!

10 minutes later the password was reset.

## That was a lot

Yes it was! 3 days to reset a password.  Just a bit crazy..

But now I have a flash drive on site that if this ever happens again.. I can spin up my vps and have them insert the flash drive.

So all in all.. Another tool for the tool belt.  Definitely might be closer to earning the Crazy in Geek Gone Crazy.  :smile:

## You left me hanging how did you reset?

Ok if you want the last 10 minutes.. they went something like this.

I mounted the hard drive with a quick:

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

![](https://media.giphy.com/media/PqjTdvXImZQfcmTYEO/giphy.gif)
