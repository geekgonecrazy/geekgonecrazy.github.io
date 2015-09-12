---
layout: post
title:  "Dedicated Virtualbox Host"
date:   2010-11-16
tags: [virtualization]
---
A while back I posted about [building a quad core system][1].  In the post I discussed virtualization, and building a dedicated machine for hosting virtual machines.  Following that, I posted about actually building that machine.  In it I mentioned installing Xenserver.  I have no doubt that it is amazing.  Just wasn't exactly what I had in mind.  

Long story cut extremely short, I went with virtualbox.  Reasons being: Its fast, so far has supported everything I throw at it, stable, cross platform, live migration, snapshots make backup easy, easy to use, and I like it. 

Now for the exciting part.  How I set it up.

<!--excerpt-->

First you obviously need to pick your host operating system.  I picked Ubuntu Server.  This is just a personal preference.  I have had good experience with it.   Most of this should work on any linux distro.


I would recommend installing ssh on the server and going headless right off.  

```
sudo apt-get install ssh
```

Then ssh in from your desktop computer to do the rest.  Copy and paste works this way :D

![][2]

Not generally a good practice.  But lets start out by becoming root.

```
sudo su
```

Next install virtualbox.

```
echo 'deb http://download.virtualbox.org/virtualbox/debian maverick non-free' >> /etc/apt/sources.list

wget -q http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc -O- | apt-key add -

apt-get update 
apt-get install virtualbox-3.2 dkms
```

Modify to suit your distro.  Virtualbox.org if you just want to download and install it that way.  

Next we need to add the virtualbox drivers to our linux kernel.  Before you can do this you need to download your kernel headers.

```
apt-get install linux-headers-server
```

Pretty straight forward.  

Now we need to compile and load the driver into the kernel.  

```
/etc/init.d/vboxdrv setup
```

Now virtualbox is pretty much setup.  But if you're like me and running this server headless..  Virtualbox is useless at this point.  

So we need a web gui.  If you don't have it already install apache2 or your preferred web server package.  

```
apt-get install apache2 php5 libapache2-mod-php5
```

Now to install the interface..  


```
cd /var/www/  
wget http://phpvirtualbox.googlecode.com/files/phpvirtualbox-0.5.zip  
unzip phpvirtualbox-0.5.zip  
mv phpvirtualbox-0.5 phpvirtualbox
```

Now for even more fun.  In my setup I wanted all virtual machines to be ran as a user called vm-user.  So we need to create that user.  

```
adduser --ingroup vboxusers vm-user
```

Followed by a somewhat secure password.  Remember this password you will need it.  

Almost forgot.  Need to now add this user info to the phpvirtualbox config.  

```
nano /var/www/phpvirtualbox/config.php
```

Change the username & password variables to the user you just created.  Ctrl+X to exit. Make sure to hit enter to save.  

Now to create virtualbox as a service that will start up as this new user upon every boot.  

```
nano /etc/init.d/vm-manage
```

In this file put the contents of this [link][4]. Ctrl+X to exit.  Make sure it saves again.  

Then make executable, and add to startup.  

```
chmod +x /etc/init.d/vm-manage  
update-rc.d vm-manage defaults
```

Now give up your admin rights!  Do it!  No but seriously your done being super admin.  Might be a good idea to give them up.  

```
exit
```

Should return you to normal 'ol you!  

Now you should be able to access the web interface from the machines ip. In this case it is: http://10.10.1.180/phpvirtualbox  

![][5]

Now you can start setting up your virtual machines.  If you want to be able to route traffic to them like I am in my setup.  Make sure to set the network interface to bridged.  

As long as you don't have your router pointing at this ip you should be safe.  If you're paranoid setup password protection.   I will do this in a later post.  


[1]: /2010/03/24/building-first-computer
[2]: http://4.bp.blogspot.com/_BMKBVRf6mio/TQoBIQzXW5I/AAAAAAAAAY0/g6Tdv1MrmvY/s320/Screenshot.png
[4]: http://pastebin.com/fgMC20b8
[5]: http://3.bp.blogspot.com/_BMKBVRf6mio/TQoBXvaX-GI/AAAAAAAAAY4/Dw8D9wLEEE4/s320/phpvirtualbox.jpg
