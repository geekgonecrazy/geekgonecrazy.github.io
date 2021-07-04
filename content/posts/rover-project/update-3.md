+++
comments = false
date = 2021-07-04T00:03:00Z
layout = "post"
publish = true
tags = ["rover-project"]
title = "Rover Project: Update 3"

+++

Finally a more eventful update!

Last time I worked with building parts in CAD I was using solidworks or AutoCAD. I have no desire to buy either one of those right now plus I need something that will run on linux.  So started investing some time into learning my way around FreeCAD.

Creating first iteration of the wheel
![](/images/2021-07-04/wheel-1-freecad.png)

50mm radius for a 10cm/100mm diameter wheel

3mm thick

Based on the specs of the motor:
```
Features:
Rated voltage: DC 3V
Rated speed: 15000 RPM
Motor size: 38 x 20 x 15 mm/1.5" x 0.8" x 0.6"
Shaft diameter: 2 mm/0.08"
Shaft length: 10 mm/0.4"
Material: metal 
```

The hole is 1mm radius for a 2mm diameter hole

Print time was going to take 2 hours and 9 minutes.  So time to reduce some material because we don't want to wait that long

![](/images/2021-07-04/wheel-2-freecad.png)

This takes it down to 1 hour 21 minutes

After lots of wrestling with bed leveling and things not sticking.. 
![](/images/2021-07-04/wheel-2-printing.jpg)


While this printed we started looking at how we want the body to look.  Designing a caster for the 3 wheeled robot we realized was a little harder then we really wanted.  Plus we were inspired by a rover... so we are leaning towards a 4 wheel design with a box to house the brain in... and surface area on top for mounting things in the future

![](https://i.imgur.com/rz8tl0g.png)
[source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.iconfinder.com%2Ficons%2F3319875%2Fisolated_robot_robotic_rover_space_icon&psig=AOvVaw2DQonss07C0LenXgaGbjZn&ust=1619650814026000&source=images&cd=vfe&ved=0CAIQjRxqFwoTCMjnvs3Dn_ACFQAAAAAdAAAAABAN)

The wheel finished printing!

![](/images/2021-07-04/wheel-2-finished.jpg)


So... we had to try it!

![](/images/2021-07-04/wheel-spinning.gif)

Woot!

Queue the other wheels!

While we waited did a bit of soldering and put antennas on the 433mhz chips

![](/images/2021-07-04/soldering-433-antenna.jpg)
![](/images/2021-07-04/433-soldered.jpg)

Then set about figuring out basic programs and getting everything hooked up to breadboards.

Got the L239D H-Bridge working doing speed control.

Then after a long period of debugging... finally got the 433mhz transmitter and receiver talking.  Ended up getting it working with the virtualwire library.

![](/images/2021-07-04/workbench-with-basic-electronics-working.jpg)

Next step get the chassis designed and printed!