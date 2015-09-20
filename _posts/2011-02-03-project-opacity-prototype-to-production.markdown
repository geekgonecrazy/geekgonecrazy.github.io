---
layout: post
title:  "Project Opacity: Prototype to Production"
date:   2011-02-03
comments: true
tags: [hardware]
---
About a month ago at work we started a project internally known as Project Opacity.

> Opacity by definition: is the quality of being opaque to a degree; the degree to which something reduces the passage of light

<!--excerpt-->

Anyways, on to what Project Opacity actually is.  Through out the library there are computer stations from which patrons can access the library catalog.  Which we call Opac's or kiosks.  Come to think of it, this probably is where the name comes from.  

An Old Opac Station
[![An Old Opac Station][1]][2]

This was one of the old Opac's.  They were becoming extremely unreliable.  Half the time they weren't working.  The graphics on the web page wouldn't show.  Pages took a long time to load.  Most of this is due to the browser being something along the lines of IE4(Ok maybe not. Honestly not sure).  It was winCE based.

This is the cabinet space below where the thin client was kept.
[![][3]][4]

[![][5]][6]

They were using old WYSE thin clients.  I'm sure these could have been updated or something.  But I can imagine licensing would have been a pain.  As well has who knows might not have supported this hardware.  Personally really glad to not have to deal with winCE, being a linux guy.

So my boss hunted around looking into upgrading these.  Custom clients vs updated WYSE type thin clients.  Custom clients won!  Thankfully!  I believe they were same price, if not cheaper with twice the hardware specs.  Operating system:
Linux.  WOOT!  Needless to say,  I got excited about this project.

So low and behold we have a prototype!!

[![][7]][8]

[![][9]][10]

Kind of a sexy little machine.  Using CF card for hard drive, Atom processor, and 2 gigs of ram.  Decent setup really.  

I then spent the better part of a week designing a custom linux install, based on [linuxmint][11].  Went through the process of actually making my own install CD.  Very cool stuff!  In the near future I plan on writing a post on doing this.  But for now sticking with Project Opacity, the Hardware side.  Believe me, plenty enough to write about!

So tested it got everything working.  Then, a shocking thing happened.  My boss opened the case, looking for a better way to mount the CF to SATA adapter. *BOOM!* *POP* *POP*  Ok, so maybe an exaggeration, but the power supply blew.
  
Long story short, verified it was the power supply, and ordered new one.  Around the same time also ordering parts, for the eight needed to replace all the Opac's, in both branches of the library.

[![][12]][13]

Cart full of parts for all eight Opac's.

Then we started building them.  Here are some pictures of the parts and putting them together.

[![][14]][15]

8gb CF card.

[![][16]][17]

The CF to SATA converter.

[![][18]][19]

2GB Ram.  Yeah I know just ram.  But I took a pic.. 

[![][20]][21]

Motherboard with Atom processor on it.

Now laying the parts out and actually building them.  Yeah I know, some of you are saying, "So what?  Been there done that."  If you're that person, go read something else.  Others that don't get the chance to build computers, carry on.  

[![][22]][23]

Laid them out.  Ain't it so purdy?  

[![][24]][25]

GAh! Wow that was hard.  We went and removed the cases from the box!  :-O  
Actually in this same step we inventoried them.  I guess that is the significance.

[![][26]][27]

Open case.

[![][28]][29]

The motherboard in, and power supply in place.

[![][30]][31]

Everything hooked up and ready.  Well almost..

[![][32]][33]

CF to SATA adapter mounted. Using power tools as an IT Assistant?? Who would have thought?

To mount the adapter, we drilled three holes through the top of the CD/Hard-drive bay. Then we used motherboard standoffs, nuts, and screws to raise it away from the metal, preventing shorting.

[![][34]][35]

Done?  No, not exactly.  One thing we forgot about completely, was how it would be fit into the cabinet.  Turns out, it didn't fit like we thought it would. So for the ones mounted on the poles(4 of them), we had to make some modifications.

[![][36]][37]

Two holes drilled where the legs normally would be glued.

[![][38]][39]

Nuts & Bolts through strips of copper

[![][40]][41]

View from the inside.  Yes had to remove the motherboard's.
Was kind of annoying.  But you do what you got to do.

[![][42]][43]

Then we hung these bad boys!

A lot of hacking, but we got them in.  Next I had to flash/clone the CF cards.  

[![][44]][45]

Prototype, being prototype?

[![][46]][47]

First attempt.

As with anything in this project, cloning the cards was a learning experience. At first we planned on using the CD I created and installing on each.  Which would have worked fine.  After turning it on, no interaction was needed for installation.  But still that's a lot of work, and lets face it.  CD drives are fast... But they are still so, slow!

So the cloning route seemed more appealing.  As seen in the screenshot, I booted from a cd, created a disc image, compressed it, then uncompressed and DD(the disk copy command in linux) to card.  Not sure why this didn't work.  Next just copied directly from one disc to another.

[![][48]][49]

Direct copying from sda to sdb.
Worked great!  Done using:  

```
dd if=/dev/sda bs=256M of=/dev/sdb
```

For those that don't know how dd works.  if=/dev/sda  "if" stands for input
file.  Everything in linux is a file.  Topic for another time.  "/dev/sda"
is the path or file where the source disc that you want to copy from is.  
"bs=256M"  aka byte size equals 256 megabytes.  The size of the blocks you want copied.  Seems the bigger byte size the faster it transfers.  It transfers
it in 256mb chunks.  Finally "of=/dev/sdb". Like the first parameter "of" is output file, and "/dev/sdb" is the output file or destination disc.  

If you have only two discs first disc being in SATA1 and second in SATA2 this is probably correct for you.  Check first!!! You get these backwards and you're gonna wipe out your source
disc.  Be careful!

Think before you do, and you'll be fine.  Now we just simply installed the CF cards.


[![][50]][51]

CF card being put in.
Finally!  Power them up!  

[![][52]][53]

Opac interface Software fully operational.  Still making some little tweaks to make the experience better. 

A couple of the key things that make this solution ideal:

Its linux!  The sky is the limit.  With just a few clicks was able to fully lock this down.  Can't close the window, log out, or do malicious things to the system.

All and all, I think we did an excellent job!  

Next, figuring out how to update them all at once.  [Puppet][54] maybe? Been dying to try it out

[1]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtyudbUbaI/AAAAAAAAAbA/9uBkOxQ8fQU/s320/2011-01-04+10.45.14.jpg
[2]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtyudbUbaI/AAAAAAAAAbA/9uBkOxQ8fQU/s1600/2011-01-04+10.45.14.jpg
[3]: http://3.bp.blogspot.com/_BMKBVRf6mio/TUt07O6PkbI/AAAAAAAAAcY/2aH_4lCzdC0/s320/2011-01-14+16.33.45.jpg
[4]: http://3.bp.blogspot.com/_BMKBVRf6mio/TUt07O6PkbI/AAAAAAAAAcY/2aH_4lCzdC0/s1600/2011-01-14+16.33.45.jpg
[5]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt1BoiuV6I/AAAAAAAAAcc/g3zITIoJPPM/s320/2011-01-14+14.14.27.jpg
[6]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt1BoiuV6I/AAAAAAAAAcc/g3zITIoJPPM/s1600/2011-01-14+14.14.27.jpg
[7]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt4z3AGryI/AAAAAAAAAdI/U_kUiATfDpE/s320/2010-12-20+17.28.40.jpg
[8]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt4z3AGryI/AAAAAAAAAdI/U_kUiATfDpE/s1600/2010-12-20+17.28.40.jpg
[9]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt4r7LY36I/AAAAAAAAAdE/MVa8701kLnA/s320/2010-12-20+16.45.57.jpg
[10]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt4r7LY36I/AAAAAAAAAdE/MVa8701kLnA/s1600/2010-12-20+16.45.57.jpg
[11]: http://www.linuxmint.com/
[12]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtyU1WevII/AAAAAAAAAaw/5CKTthN9qgg/s320/2010-12-29+12.55.53.jpg
[13]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtyU1WevII/AAAAAAAAAaw/5CKTthN9qgg/s1600/2010-12-29+12.55.53.jpg
[14]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzYqZBsqI/AAAAAAAAAbY/-bCJlQxZP50/s320/2011-01-06+12.19.07.jpg
[15]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzYqZBsqI/AAAAAAAAAbY/-bCJlQxZP50/s1600/2011-01-06+12.19.07.jpg
[16]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzoyzIuSI/AAAAAAAAAbk/2ENjFGeInGU/s320/2011-01-06+12.18.52.jpg
[17]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzoyzIuSI/AAAAAAAAAbk/2ENjFGeInGU/s1600/2011-01-06+12.18.52.jpg
[18]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzgiX52MI/AAAAAAAAAbg/qMRMPVImJqE/s320/2011-01-06+12.19.00.jpg
[19]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzgiX52MI/AAAAAAAAAbg/qMRMPVImJqE/s1600/2011-01-06+12.19.00.jpg
[20]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtzPCPAWfI/AAAAAAAAAbU/3pWrGRJI19g/s320/2011-01-06+12.19.30.jpg
[21]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtzPCPAWfI/AAAAAAAAAbU/3pWrGRJI19g/s1600/2011-01-06+12.19.30.jpg
[22]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUty2xLkk-I/AAAAAAAAAbE/8EHQUSU_D1g/s320/2011-01-04+12.51.45.jpg
[23]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUty2xLkk-I/AAAAAAAAAbE/8EHQUSU_D1g/s1600/2011-01-04+12.51.45.jpg
[24]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUty-YVRM-I/AAAAAAAAAbI/QpsEkDAfuQk/s320/2011-01-06+11.58.04.jpg
[25]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUty-YVRM-I/AAAAAAAAAbI/QpsEkDAfuQk/s1600/2011-01-06+11.58.04.jpg
[26]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzGqnqu6I/AAAAAAAAAbM/4qGsU2rOEN4/s320/2011-01-06+12.18.47.jpg
[27]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzGqnqu6I/AAAAAAAAAbM/4qGsU2rOEN4/s1600/2011-01-06+12.18.47.jpg
[28]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUtz4k0huiI/AAAAAAAAAbw/PHvYHqteIRw/s320/2011-01-06+12.39.23.jpg
[29]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUtz4k0huiI/AAAAAAAAAbw/PHvYHqteIRw/s1600/2011-01-06+12.39.23.jpg
[30]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzwsal6qI/AAAAAAAAAbs/O0vjdicS1VY/s320/2011-01-06+12.39.16.jpg
[31]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUtzwsal6qI/AAAAAAAAAbs/O0vjdicS1VY/s1600/2011-01-06+12.39.16.jpg
[32]: http://3.bp.blogspot.com/_BMKBVRf6mio/TUt0BddxyJI/AAAAAAAAAb0/VVhcAmYjMmQ/s320/2011-01-06+14.34.58.jpg
[33]: http://3.bp.blogspot.com/_BMKBVRf6mio/TUt0BddxyJI/AAAAAAAAAb0/VVhcAmYjMmQ/s1600/2011-01-06+14.34.58.jpg
[34]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt0UeGloxI/AAAAAAAAAcA/0pO-7kBFyJA/s320/2011-01-06+15.21.32.jpg
[35]: http://4.bp.blogspot.com/_BMKBVRf6mio/TUt0UeGloxI/AAAAAAAAAcA/0pO-7kBFyJA/s1600/2011-01-06+15.21.32.jpg
[36]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtydKPai0I/AAAAAAAAAa0/BEMk6Z1k5TM/s320/2011-01-13+17.26.43.jpg
[37]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtydKPai0I/AAAAAAAAAa0/BEMk6Z1k5TM/s1600/2011-01-13+17.26.43.jpg
[38]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtylkLXiqI/AAAAAAAAAa4/i7IOKSjHDZI/s320/2011-01-13+17.37.59.jpg
[39]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtylkLXiqI/AAAAAAAAAa4/i7IOKSjHDZI/s1600/2011-01-13+17.37.59.jpg
[40]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0jkuSwuI/AAAAAAAAAcI/Kkr0k9_mNQ4/s320/2011-01-14+14.07.03.jpg
[41]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0jkuSwuI/AAAAAAAAAcI/Kkr0k9_mNQ4/s1600/2011-01-14+14.07.03.jpg
[42]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0y6bxc4I/AAAAAAAAAcU/6qI2jAFPpVk/s320/2011-01-14+17.04.58.jpg
[43]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0y6bxc4I/AAAAAAAAAcU/6qI2jAFPpVk/s1600/2011-01-14+17.04.58.jpg
[44]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0rrjvOoI/AAAAAAAAAcM/nFuaM450E04/s320/2011-01-07+13.45.57.jpg
[45]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt0rrjvOoI/AAAAAAAAAcM/nFuaM450E04/s1600/2011-01-07+13.45.57.jpg
[46]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1S5mX96I/AAAAAAAAAco/Jv7ixJ6FkTg/s320/2011-01-07+16.57.25.jpg
[47]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1S5mX96I/AAAAAAAAAco/Jv7ixJ6FkTg/s1600/2011-01-07+16.57.25.jpg
[48]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1IhYqK8I/AAAAAAAAAcg/FNczc8hWaJM/s320/2011-01-07+16.18.16.jpg
[49]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1IhYqK8I/AAAAAAAAAcg/FNczc8hWaJM/s1600/2011-01-07+16.18.16.jpg
[50]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1dAOuXqI/AAAAAAAAAcs/VdeUjzHcA0A/s320/2011-01-12+13.43.19.jpg
[51]: http://2.bp.blogspot.com/_BMKBVRf6mio/TUt1dAOuXqI/AAAAAAAAAcs/VdeUjzHcA0A/s1600/2011-01-12+13.43.19.jpg
[52]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtyAN1SkAI/AAAAAAAAAas/seTsepqF-84/s320/2011-01-18+14.34.02.jpg
[53]: http://1.bp.blogspot.com/_BMKBVRf6mio/TUtyAN1SkAI/AAAAAAAAAas/seTsepqF-84/s1600/2011-01-18+14.34.02.jpg
[54]: http://www.puppetlabs.com/
