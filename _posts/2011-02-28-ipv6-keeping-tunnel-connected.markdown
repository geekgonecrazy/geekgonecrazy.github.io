---
layout: post
title:  "IPv6 keeping tunnel connected"
date:   2011-02-28
tags: [ipv6]
---
Put together a script to keep your ip updated for the tunnel.  This is for Hurricane Electric.  Just plug in your info from your tunnel info page.  

<!--excerpt-->

<script src="https://gist.github.com/geekgonecrazy/882540.js"></script>

If Ubuntu, copy this file to:  /etc/init.d/updateip  
Then, `sudo ln -s /etc/init.d/updateip /etc/rc2.d/S95updateip  `

Will now start up with the computer.  
Hope it helps someone.
