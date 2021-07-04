+++
comments = false
date = 2021-07-04T00:01:00Z
layout = "post"
publish = true
tags = ["rover-project"]
title = "Rover Project: Update 1"

+++

Parts came in!

![](/images/2021-07-04/parts-1.jpg)
![](/images/2021-07-04/parts-2.jpg)

Finally got time to work on it again.  We wired everything up.

![](/images/2021-07-04/parts-hookup-1.jpg)

We gathered up some cardboard.  We found something to trace and cut out 4 wheels.  (you'll notice this is defintely a deviation from the original sketch. But this was insisted on once the additional motors were spotted).


![](/images/2021-07-04/parts-hookup-2.jpg)

We also put together the transmitter on another breadboard

![](/images/2021-07-04/parts-hookup-3.jpg)


Now if you do electronics seriously at all you'll probably notice straight away two different issues. I was tempted to omit it and just have the build log with final parts.. But this is part of the journey.  I think it was important for kids to go through as well.  Not everything works first time around.  In fact probably doesn't most of the time.

Issue 1: Those are stepper motors and drivers. 🙈 Yeah.. I don't even know how I did that.  I did place the order late.. but I even have some articles stashed away specifically for working with those drivers. So its not like I didn't research the parts.  We'll chock it up to lesson learned though.

Issue 2: Those 433Mhz Transmitter / Receivers don't have even their basic included antenna's soldered on.  Reading later.. with out an antenna range is like a few centemeters.

Needless to say we discovered both of these issues after several hours.  I pretty quickly wrote up the arduino code for both.  I wrote the code that would send a simple signal that would include a signal for both the left and the right side tank style.

My Plan was:

* Forward: 01 FF 01FF
* Backward: 02 FF 02 FF
* Forward Left: 01 FF 01 99
* Forward Right: 01 99 01 FF

Pattern being: {Left Motor Polarity} {Left Motor Speed} {Right Motor Polarity} {Right Motor Speed}

With the speed being 0-255 represented in hex. No CRC or anything at first.

This didn't work.  

So figured maybe I was being too clever and switch to transmitting strings and probably 10x variations.

So after still not even a hum from the motors.. I figured it must be just something wrong with the 433Mhz pair.  I wanted to at least validate the motors worked.  So I found an example program for those motors show them working.  It was here and getting this to work... I realized these aren't really the right motors for the wheels.

So we shelved it until I could go and order the right motors.

It wasn't until later I actually realized I some how completely spazzed on putting antenna's on and their range was only centimeters with out one.  