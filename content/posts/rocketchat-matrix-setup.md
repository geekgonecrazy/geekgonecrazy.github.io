+++
comments = false
date = 2022-05-30T00:00:00Z
layout = "post"
publish = true
tags = []
title = "Rocket.Chat and the Matrix Protocol"

+++

Rocket.Chat [now has federation support via the Matrix protocol](https://rocket.chat/press-releases/rocket-chat-leverages-matrix-protocol-for-decentralized-and-interoperable-communications).

This announcement is an exciting one to me.  I've been working with Rocket.Chat since I found it almost 7 years ago.  One of the things I recall from early on is the vision of being as ubiquitious as email.  One of the beauties of email is not everyone is on the same domain or let alone even the same kind of mail server.  Just all speaking the same language.

Rocket.Chat being able to talk the Matrix Protocol to me is a start of it achieving that.

It does this by registering as an AppService with a Matrix homeserver. This means Rocket.Chat is able to create users on the homeserver and then manage them.  So when you opt to federate a room and invite a remote user using Matrix.  Rocket.Chat creates your user on the homeserver and controls it as if they are one and the same user.

tl;dr You can now talk to anyone speaking the Matrix Protocol from Rocket.Chat.

# How to setup Rocket.Chat with a Matrix homeserver

- **Matrix support in Rocket.Chat is currently considered alpha.  Expect things to break**
- **This guide also uses Dendrite which is beta**

Video guide should you prefer that format:
{{< youtube id="oQhIH8kql9I" >}}

## Compute Requirements

If your Rocket.Chat workspace will only support a few users you can generally get away with a machine like:

- 2GB of RAM
- 2CPU
- 20GB of Storage

If you start using it more heavily you'll probably want to increase to a couple of CPU and more ram.  See the [Rocket.Chat Documentation](https://docs.rocket.chat) for more details.

In this case i've selected this digitalocean droplet:
<img width="841" alt="image" src="https://user-images.githubusercontent.com/51996/170799991-4bb5b9fd-645d-4f1a-835d-79bd358f96bb.png">

I've also selected Ubuntu 20.04 as the distro.

## Domain Requirements

You'll need a domain you control. In this case i'm going to use geekgone.dev.

For federation generally the idea is that you are accessed much like you are by email.  
So if I was to be reached at aaron [at] geekgone.dev then I would also be reached at @aaron:geekgone.dev.

Either way you need to decide what domain will you be reached at?  We'll call this the federated domain.  Then you'll also need to expose matrix and Rocket.Chat on their perspective domains.

In my case i'm going to use:
- Federated Domain: geekgone.dev
- Rocket.Chat Domain: chat.geekgone.dev
- Matrix Domain: matrix.geekgone.dev

Go ahead and point those all at the machine you've provisioned.

![image](https://user-images.githubusercontent.com/51996/170783824-d8afd20a-2aae-4dd2-8ce6-9b1173eb45d8.png)

## Install some pre-reqs

We need a reverse proxy.  I'm going to use nginx:

```
sudo apt update && sudo apt install -y nginx 
```

Now we need docker:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

Now for ease we'll add our self to the docker group and assume it:
```
sudo usermod -a -G docker ubuntu
newgrp docker
```

Finally we need certbot.  This is one of the quickest and easiest way to install it currently:
```
sudo snap install --classic certbot
```

## Configure Lets Encrypt Certificates

```
certbot certonly --nginx -d geekgone.dev,chat.geekgone.dev,matrix.geekgone.dev
```

## Setup Nginx Rules

Edit /etc/nginx/sites-available/default

```
server {
    listen 80 ;
    listen [::]:80 ;

    if ($host = geekgone.dev) {
        return 301 https://$host$request_uri;
    } 

    if ($host = chat.geekgone.dev) {
        return 301 https://$host$request_uri;
    } 
    
    if ($host = matrix.geekgone.dev) {
        return 301 https://$host$request_uri;
    } 
        
    server_name matrix.geekgone.dev chat.geekgone.dev geekgone.dev;
    return 404; 
}

server {
    listen 443 ssl; 
    ssl_certificate /etc/letsencrypt/live/geekgone.dev/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/geekgone.dev/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 

    root /var/www/html;

    server_name geekgone.dev; 

    location /.well-known/matrix/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:8008;
        proxy_read_timeout 90;
    }
}

server {
    listen 443 ssl; 
    ssl_certificate /etc/letsencrypt/live/geekgone.dev/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/geekgone.dev/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 

    root /var/www/html;

    server_name chat.geekgone.dev; 

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:3000;
        proxy_read_timeout 90;
    }
}

server {
    listen 443 ssl; 
    ssl_certificate /etc/letsencrypt/live/geekgone.dev/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/geekgone.dev/privkey.pem; 
    include /etc/letsencrypt/options-ssl-nginx.conf; 
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 

    root /var/www/html;

    server_name matrix.geekgone.dev; 

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:8008;
        proxy_read_timeout 90;
    }
}
```

Do a `nginx -t` just to make sure everything is ok.

Then:
```
systemctl reload nginx
```

## Install Rocket.Chat and Dendrite

I've commited a docker-compose and config file in a public repo.  So will start off with grabbing that:

```
git clone https://github.com/geekgonecrazy/rocketchat-matrix-federation-example.git matrix
cd matrix
```

Lets bring Rocket.Chat up:
```
docker compose up -d rocketchat mongo mongo-init-replica
```

Goto your Rocket.Chat install and finish the initial setup.  In my case it was available at https://chat.geekgone.dev

Now goto: Admin->Settings->Federation and expand the Matrix Bridge section

In the Homeserver URL section use:
```
http://dendrite:8008
```

In the Homeserver Domain use your federation domain.  In my case:
```
geekgone.dev
```

Now flip to enabled and click save changes.

Grab the Homeserver Token and the AppService Token and put them some place safe for a moment.

Rocket.Chat is ready.  Now lets go back to the terminal and look at dendrite.

We need to edit the dendrite config file.  Edit: `config/dendrite.yaml`

Replace geekgone.dev in this section with your federation domain:
```
global:
  # The domain name of this homeserver.
  server_name: geekgone.dev
```

Replace matrix.geekgone.dev in this section with your matrix domain:
```
  # The server name to delegate server-server communications to, with optional port
  # e.g. localhost:443
  well_known_server_name: "matrix.geekgone.dev:443"
```

Now we need to edit: `config/rocketchat-registration.yaml`
Replace: `hs_token` value with the Homeserver Token you got from Rocket.Chat
Replace: `as_token` value with the AppService Token you got from Rocket.Chat

Finally now we can start matrix up:
```
docker compose up -d dendrite
```

## Lets try talking to another Matrix Homeserver

Now that we have dendrite up and Rocket.Chat up.  Lets go into Rocket.Chat and try sending a message out to a user on another server out there speaking the Matrix Protocol.  Can be another Rocket.Chat server, dendrite, synapse etc.

In this case i'm going to use an account from matrix.org [@geekgonedev-demo:matrix.org](https://matrix.to/#/@geekgonedev-demo:matrix.org)

Create a channel in Rocket.Chat and then do: `/bridge invite @geekgonedev-demo:matrix.org`

The client you have connected on the other server you will receive an invite.  Once accepted you can now type a message on either side and if all went well you should receive the message on the other side!

_**Note: if you invite another Rocket.Chat server it currently will auto accept**_

I'll leave the server up for a couple of weeks if you want to try reaching me at [@aaron:geekgone.dev](https://matrix.to/#/@aaron:geekgone.dev)

Happy chatting!
