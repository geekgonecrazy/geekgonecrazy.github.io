---
layout: post
title:  "Getting public ip from CLI"
date:   2012-05-16
comments: true
publish: true
tags: [tips, linux]
---
I've found my self on occasion needing to find the public IP of a machine while in commandline(CLI).  Most of the time its just because i've spoiled my self with DNS entries, and haven't memorized it.  

<!--excerpt-->

You need to have curl installed.  If you don't have it, it should be in your package manager.  In Ubuntu its easily installed via:

```Bash
sudo apt-get install curl
```


Then just simply run:  

```Bash
curl ifconfig.me
```

I figured since I hadn't posted in a while, this is a useful enough tidbit to share.
