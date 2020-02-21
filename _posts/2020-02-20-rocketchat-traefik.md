---
layout: post
title:  "Rocket.Chat + Kubernetes + Traefik = :heart:"
date:   2020-02-20
comments: true
publish: false
tags: [traefik]
---

I've been a docker / container fan boy since the dotcloud days.  Managed to introduce them into a couple of previous jobs. Primarily only as aids for the build process or for maintaining the same environment between dev and production. But at my current job I got the opportunity to really go all in. At Rocket.Chat we wanted to build a cloud offering to allow people to quickly and easily deploy Rocket.Chat.

We started this journey off with a couple of my collegues actually putting together a quick proof of concept using a wildcard cert, hipache and a solution that was at the time called tutum.  Tutum was later aquired by docker.. and shutdown.

This solution worked great as a quick proof of concept.  We were able to setup a page on our website and people could come in and try Rocket.Chat out. It would provision using tutum and proxy traffic through with hipache and the user would have a Rocket.Chat instance to play with in just a few minutes.

[insert screenshot if we can find one]

The problem with this solution is it just wouldn't scale that well. Tutum wasn't really great to work with and introduced a lot of weird quirks into the system. It seemed to perform worse an worse the more containers we had running.  Hipache while doing the job didn't really fit that nicely with this dynamic load.  We had to setup a redis cluster just for hipache and inject new records every time we wanted to handle traffic for a new subdomain.

We started to evaluate a few other options.  At the time the orchestration landscape was actually rather scarce. Kubernetes already existed and looked super promising, but it was a bit young.  We looked at adapting a few other solutions to fit our needs.. We checked out Juju charms and LXD containers.. But.. after some prototyping felt that it was going to take a lot of work to make work as we invisioned.

Finally we decided to just take a bet on Kubernetes. Much like a lot of other companies were doing at the time.  It of course helped to see companies both big and small on board.  We really loved that Kubernetes had a super declaritive way of defining the payload and that it had a super easy to use api. We also really loved the idea that we could remain vendor independent.

We were able to pretty quickly get a yaml file up and running.  It looked much like it does today:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    DeploymentId: {someid}
  name: rc-{someid}
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      DeploymentId: {someid}
  template:
    metadata:
      labels:
        DeploymentId: {someid}
    spec:
      containers:
      - env:
        - name: INSTANCE_IP
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: status.podIP
        - name: MONGO_URL
          value: mongodb://db1.internal.domain,db2.internal.domain,db3.internal.domain/{some id}?replicaSet=rs01
        - name: MONGO_OPLOG_URL
          value: mongodb://db1.internal.domain,db2.internal.domain,db3.internal.domain/local?replicaSet=rs01
        - name: ROOT_URL
          value: https://open.rocket.chat
        image: rocketchat/rocket.chat:3.0.0
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 10
          httpGet:
            path: /api/info
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 15
          successThreshold: 1
          timeoutSeconds: 10
        name: rc-{someid}
        ports:
        - containerPort: 3000
          protocol: TCP
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /api/info
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        resources:
          requests:
            memory: 600Mi
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    traefik.backend.circuitbreaker: NetworkErrorRatio() > 0.5
    traefik.backend.loadbalancer.method: drr
    traefik.ingress.kubernetes.io/affinity: "true"
  labels:
    DeploymentId: {someid}
  name: rc-{someid}
  namespace: default
spec:
  ports:
  - name: http
    port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    DeploymentId: {someid}
  sessionAffinity: None
  type: ClusterIP

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: rc-{someid}
  namespace: default
spec:
  rules:
  - host: open.rocket.chat
    http:
      paths:
      - backend:
          serviceName: rc-{someid}
          servicePort: 3000

```

We started writing a tool that we call fleetcommand.  This tool handles all of the talking to the kubernetes clusters, the billing integrations, the customer self service options etc.  We wrote it in Go. Which actually allowed us a lot of benefits.  We were able to easily wrap the kubernetes client and use it directly instead of having to build our own mappings to the api.

When we started looking at our vendor for our managed Kubernetes (GKE) we quickly realized we had a bit of a problem. If we just defined an ingress rule like we were in our minikube environment... While Google would handle it for us.. It would also give a new ip for each one. Which while great of them.. meant we would some how have to get DNS configured and then after all of that we would be pretty shackled into GCP.

## Introducing Traefik

We started looking around at various ingress controllers. At the time there weren't a lot. We found the default nginx controller that I think is still used as the example ingress controller and used in minikube.  Some how we stumbled across traefik.  Right off the bat I loved Traefik because of the gopher. 

![traefik mascot](https://github.com/containous/traefik/raw/master/docs/content/assets/img/traefik.logo.png){: .center-image }

But then we discovered that not only did it have a super cool mascot.  They also could act as an ingress controller. This would allow us to just programatically insert an ingress record into our Kubernetes cluster and traefik would scoop it up and handle the rest.  They also added a little icing on the cake by providing support for provisioning Letsencrypt certs.  Which we knew we would want to make use of for custom domains.

## Putting the pieces together

So then we were able to put the pieces together. 

* A wildcard DNS record so no need to add on the fly DNS record creation.
* One Google Provided IP and TCP LoadBalancer
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

We are still very thankful to have discovered 2 key pieces. Kubernetes and Traefik
