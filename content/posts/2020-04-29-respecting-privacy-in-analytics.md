---
layout: post
title:  "Respecting Privacy in Analytics"
date:   2020-04-29
publish: true
tags: [100DaysToOffload]
---

There is a reason why people tend to add analytics to sites. I'd say most people add for completely honest reasoning.  They just want to be able to see how their site is doing and if anyone actually shows up. 

<!--excerpt-->

But its really important to go down the path with caution.  If you dont carefully select and think through your analytics… You end up spying on your users likely without even realizing it.

Let's imagine you read the exciting sales pitch from AssociatedIn on how adding their analytics will let you see the types of users or businesses visit your site.  Seems innocent enough right?

Lets put on our privacy detective hat!

So you turn it on and they start giving you this data.. How are they able to give you this info? The only way they would know anything about your users if they send something to them selves right?  So of course they do this.  But the only way them sending the page to themselves would be useful if they included something about the user. So they have to some how identify you or to see if unique.

Maybe they see you are logged in to their service? Ouch.. But ok ones easy.

Maybe they fingerprint you by grabbing an assortment of clues the browser gives away?  Could be.. To make it even easier on themselves maybe they drop a cookie in the browser so next time that user visits a page they will know it'd the same user.  Oh it happens to also send to any other site using their analytics?

This just went from analytics straight on over in to tracking territory.  Now by getting some analytics about the type of business your users run.. You have traded it for your user being tracked.

The tracking company is then aggregating this data, building models about users, maybe selling it?  Or maybe they aren't so blatenly nefarious.. maybe they are just offering a paid plan offering advanced abilities.  Your users data that you are giving them now let's them make money.

Seemed innocent though right?

Super important to think about the type of information we collect about our users.  Is that bit of data worth allowing a tracker to invade your users privacy?


# Ok so what should I use?

Great question!  First off self hosting an open source analytics tool can be a great way to know exactly what data you collect and how its collected.  Also you ensure that data isn't then being aggregated to track your users across the web.

I'm still on the hunt for good options but will update this list as I find out about them. But here are a couple:

## Matomo

https://matomo.org is one recommended by many people in the open source community. Full featured alternative to the likes of Google Analytics.


## Fatham

Recently came across this one.  I'm actually using it for my site and inspired this topic. Feels great for use cases like mine.

Its kind of confusing.. But apparently their first version is/was open source.  Then they rewrote it and this version is not open source but available on their SaaS offering.

But the open source version can be found here: https://github.com/usefathom/fathom

What I like about this is I can see exactly what is transmitted and what is stored.

Transmitted: [here](https://github.com/usefathom/fathom/blob/8f7c6d2e45ebb28651208e2a7320e29948ecdb2c/assets/src/js/tracker.js#L162)
Saved: [here](https://github.com/usefathom/fathom/blob/8f7c6d2e45ebb28651208e2a7320e29948ecdb2c/pkg/api/collect.go#L54)

Here is a sample of my brief moment on hackernews yesterday.  But what you see there is it.  Super simple, but does the job.

![](https://paper-attachments.dropbox.com/s_3A770CD59D00AB3A931B3BF5D654149EDE7875E5917677E7CC0A064B28BBDD07_1588216208662_file.jpeg)



Think about privacy the next time you start adding analytics to your website.  Think about what you are making your users give up in exchange. Do the right thing 🙂
