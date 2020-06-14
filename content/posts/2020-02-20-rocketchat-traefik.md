---
layout: post
title:  "Rocket.Chat + Kubernetes + Traefik = :heart:"
date:   2020-02-20
comments: true
publish: true
tags: [traefik]
---

You could say I've been a docker / container fanboy for a while now...  I started working with containers in a couple of previous jobs. Primarily only as aids for the build process or for maintaining the same environment between dev and production. But at my current job I got the opportunity to really go all in. At Rocket.Chat we wanted to build a cloud offering to allow people to quickly and easily deploy Rocket.Chat.

<!--excerpt-->

We started this journey off with a couple of my collegues actually putting together a quick proof of concept using a wildcard cert, hipache and a solution that was at the time called tutum.  Tutum was later aquired by docker.. and shutdown.

This solution worked great as a quick proof of concept.  We were able to setup a page on our website and people could come in and try Rocket.Chat out. It would provision a docker container using tutum and proxy traffic through with hipache and the user would have a Rocket.Chat instance to play with in just a few minutes.

![screenshot](/images/2020-02-20/deploy.png)

The problem with this solution is it just wouldn't scale that well. Tutum wasn't really great to work with and had a lot of weird quirks. It seemed to perform worse the more containers we had running.  Lots of network issues, lower level docker issues etc.  Hipache while doing the job didn't really fit that nicely with this dynamic load.  We had to setup a redis cluster just for hipache and inject new records every time we wanted to handle traffic for a new subdomain.

We started to evaluate a few other options.  At the time the orchestration landscape was rather scarce. Kubernetes already existed and looked super promising, but it was a bit young.  We looked at adapting a few other solutions to fit our needs.. We checked out Juju charms and LXD containers.. But.. after some prototyping felt that it was going to take a lot of work to make work as we invisioned.

Finally we decided to just take a bet on Kubernetes. Much like a lot of other companies were doing at the time.  It of course helped to see companies both big and small on board.  We really loved that Kubernetes had a super declaritive way of defining the payload and that it had a super easy to use api. We also really loved the idea that we could remain vendor independent.

We were able to pretty quickly get a yaml file up and running.  It looked much like it does today:

<script src="https://gist.github.com/geekgonecrazy/02aacfbf1853b7689832343c5971823d.js"></script>

We started writing a tool that we call fleetcommand.  This tool handled all of the talking to the kubernetes clusters, the billing integrations, the customer self service options etc.  We wrote it in Go. Which actually allowed us a lot of benefits.  We were able to easily wrap the kubernetes client and use it directly instead of having to build our own mappings to the api. We were able to use it to directly inject the deployment, service and ingress objects directly into kubernetes.  Its evolved a lot.. but its still in use today. :smile:

When we started looking at our infrastructure vendor for our managed Kubernetes (GKE) we quickly realized we had a bit of a problem. If we just defined an ingress rule like we were in our minikube environment... While Google would handle it for us.. It would also give a new ip for each one. Which while great of them.. meant we would some how have to get DNS configured and then after all of that we would be pretty shackled into GCP.

## Introducing Traefik

We started looking around at various ingress controllers. At the time there weren't a lot. We found the default nginx controller.. I think is still used today as the example ingress controller and used in minikube.  Some how we stumbled across traefik.  Right off the bat I loved Traefik because of the gopher. 

![traefik mascot](https://github.com/containous/traefik/raw/master/docs/content/assets/img/traefik.logo.png)

But then we discovered that not only did it have a super cool mascot.  They also could act as an ingress controller. This would allow us to just programatically insert an ingress record into our Kubernetes cluster and traefik would scoop it up and handle the rest.  They also added a little icing on the cake by providing support for provisioning Letsencrypt certs.  Which we knew we would want to make use of for custom domains.  It also certainly didn't hurt that it was also written in Go.  

## Putting the pieces together

We were then able to put the pieces together. 

* A wildcard DNS record so no need to add on the fly DNS record creation.
* A Google Provided IP and TCP LoadBalancer
* A couple of Traefik instances sitting on nodes we tainted as edge nodes and Traefik had selectors for and had a tolerence for.
* Fleetcommand to talk to kubernetes api to inject the deployment, service and ingress records.
* Some worker nodes
* Some database nodes

## Now

A lot has changed! 
* We switched away from GCP. Proving our solution was actually able to be vendor agnostic.
* We introduced multiple regions - We now spin up DNS records
* We still use and love Traefik - Have gone from just 2 Traefik instances to now 10+
* We let our customers bring their own custom domains and provide Letsencrypt certificates for them
* We still love Golang for our infrastructure needs.  But thats a topic for another time. :smile:

We are still very thankful to have discovered 2 key pieces. Kubernetes and Traefik
