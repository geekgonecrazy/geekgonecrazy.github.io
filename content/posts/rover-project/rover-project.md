+++
comments = false
date = 2021-07-04T00:00:00Z
layout = "post"
publish = true
tags = ["rover-project"]
title = "Rover Project"

+++

My family is very STEM oriented.  I grew up learning hands on at home. Taking things apart. Questioning how things worked and eventually learning how to actually fix those things I took apart. As much as possible we're trying to encourage this for our kids.  

They are pretty fortunate as they have good examples of this on both sides of the family.  So they come by their curiousity about the world around them very naturally.  Now days its not surprising to get asked questions about things even we don't know or remember the answer to.

Not going to pretend some times the constant questions doesn't get a little annoying.  But just have to stop and appreciate the curiousity and help them learn more about the things they are actually curious about.

Anyways we often watch Rocket Launches together.  Now days thanks to the likes of SpaceX, Rocket Labs and even the old school ULA and others.. we frequently get to watch something launch.

About a year ago on 07/30/2020 we watched the Mars Perseverance mission take off.  This kicked off a flurry of questions.

What is it launching?  What is the rover? What does it do?  How will it land?  How does it work?

Then after spending some time showing them and answering.  I was left with one statement:

> I wish kids like us could make robots

I like making things.. and I used to help a highschool robotics team build their robot for competition every year.  I really missed it.  So really this was a win-win-win:
* I get to teach my kids how to approach engineering problems as best I know how. 
* They are really excited
* I get to work on building something too

So off began the process!

## Brainstorming

So we started right off with brainstorming. During brainstorming we came up with a quick sketch of some that it could look like:

![](/images/2021-07-04/rover-sketch-1.png)

A couple of things to note:

1. There is an arm
2. In the left corner there is a tablet controlling 

We're considering these stretch goals.  We agreed that the first priority was getting a basic robot built.


But through brainstorming we identified the main pieces that will be needed:

* 2 motors to drive the wheels
* 2 wheels

* A base to act as chassis and to put the electronics on
* Transmitter / Receiver
* Bread boards - I didn’t have any more.
* Batteries
* A brain - some sort of microcontroller

We also identified that we would need some buttons and a way to control. So we needed some buttons.

We weren’t too certain on the construction of it. We figured maybe CD’s would make ok wheels. We’ve seen robots with CD wheels. Also we have plenty of cardboard. Might work as a prototype.

I tried to involve the kids as much as possible on the research behind the parts to choose. But this part quickly bored them. So I went off and spent a few hours that evening locating all of the parts.

Landed on using the Ardunio Nano. This is mostly because info for arduino is a lot easier to come by. I don’t want to waste too much time on the code portion. I want to be able to explain and the kids be able to follow along with the code and how it works.

For the batteries I’m a noob on batteries. I didn’t want to go off and order some rechargable battery and it be wrong for our circuit. From what I could tell for the Arduino nano everything seemed to be 1.5V compatible so just picked up battery pack to put AA batteries in.

**Motors… I’m writing this in retrospect… so I don’t really know why I ended up purchasing stepper motors and the controllers. Its of course not a waste… But it did delay the project.**

Parts Ordered:
* [Motors](https://www.amazon.com/gp/product/B087B5NWY4/ref=ppx_yo_dt_b_asin_title_o03_s03?ie=UTF8&psc=1)
* [Arduino Nano](https://www.amazon.com/gp/product/B07G99NNXL/ref=ppx_yo_dt_b_asin_title_o03_s01?ie=UTF8&psc=1)
* [433Mhz Transmitter / Receivers](https://www.amazon.com/gp/product/B086ZL8W1W/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
* [AA Battery box](https://www.amazon.com/gp/product/B07C6T6YCZ/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)

Now we waited on the parts to arrive.