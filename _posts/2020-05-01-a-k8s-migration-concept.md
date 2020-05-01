---
layout: post
title:  "A k8s Migration Concept"
date:   2020-05-01
publish: false
tags: [100DaysToOffload]
redirect_from:
  - /2020/05/01/a-k8s-migration-story/
---

Surprisingly I don’t have a lot of kubernetes related content on my blog yet.  This might just because I’m too busy using it to talk about it.  But i’m a heavy user of it.  To me it just makes orchestrating work loads easier.

One thing in general I don’t see a lot of talk about is migrating between infrastructure.  Especially don’t see much for kubernetes.  

<!--excerpt-->

So I wanted to share a high level concept set of steps that i’ve followed a couple of times to migrate between a couple of kubernetes clusters with minimal to no downtime.


### Establishing a link between environments

The first step in this process was establishing a link between the old environment and the new environment.  Most major cloud providers have VPC peering so can set this up pretty easily.  So far all have been IPSEC.  A few tweaks to the firewall later you can talk across.


### Spin up new DB nodes

In my case it was some mongo nodes.  Once they were up, joined them to the replicaset on the other side.  One important thing here is they were set with 0 priority to prevent them from some how becoming a primary before we were ready


### Proxy Endpoint

Once the DB is ready and ready to migrate.  Established an endpoint from the new environment across to the old environments ingress provider.  On caveat here is did actually have to spin up a second internal ingress provider that would handle proxying to load inside the cluster.  But did not handle TLS termination.  This is what we pointed the endpoint to.


### Create ingress records

Now that the other side is ready to accept traffic over the peering and route the traffic to the appropriate place… Need to actually tell the new environment what routes to listen for and where to send them.  So all of them just point to the proxy endpoint.

At this point you can make a request like:

```
curl --verbose --header 'Host: www.example.com' 'http://10.1.1.36:8080/url'
```

### DNS

Now that requests are verified to go across you can swap over the DNS from old environment to the new environment.  Making sure to take note of the TTL on the record.  I actually decreased the TTL a couple of days prior to prevent having to wait as long.

So the traffic will flow into new environment and the proxy over to old environment


### Migrate

Once  give the time for the TTL time to have fully ran out.  Could start migrating.

For us this process looked like:


1. Deploy Deployment
2. Deploy SVC
3. Swap ingress to point to service
4. Verify traffic hits
5. Spin down resource in old environment


### DB Primary Switch

We waited until about half of the deployments had switched and then adjusted priority in our DB cluster so that only nodes in the new environment had priority and now all in old environment had none and then made our primary in the old environment step down.

Important to make sure your application can actually detect and handle this. Otherwise it won’t work.


### Finishing up

Once it was all over only thing left to do was to clean up.

To me the magic here is in being able to change the DNS and traffic still be working with some minimal latency increase.

Things can definitely go wrong. You have to plan for them.  This is also why giving your self a maintenance window is also extremely important.

Also if you know anything about working with the kubernetes api.. consider writing something to do this.

