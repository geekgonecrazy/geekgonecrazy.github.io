+++
comments = false
date = 2020-09-07T00:00:00Z
layout = "post"
publish = true
tags = []
title = "Tinkerbell or iPXE boot on OVH"

+++
Recently have had to work with a number of physical servers that need to be treated more cattle like than a bunch of pets.  I need to be able to quickly and easily provision servers to fit a specific role with out having to go through a complicated setup process.

For this type of thing PXE or iPXE comes to the rescue.  Allowing you to boot over the network.  Simply rebooting the machine and then having it pick up its instructions and be provisioned like needed.

## Tinkerbell

Because of this need a project called tinkerbell has come to my attention. Project website is: https://tinkerbell.org.  Seems packet uses this or a variation of this inside of their infrastructure.

Tinkerbell is made up of a few components:

### tink-server

This provides the state. It is made up of:

#### templates

Templates have a series of actions that are docker images to be ran.

#### hardware

This is a json representation of the machine, metadata, nics with their interface definition ip/mac address.

#### workflows

Workflows are essentially just a job.  A match up of hardware and template for it to run. The state is updated as the job runs.

### boots

This provides dhcp and the ipxe script to the server so it can boot.  When it receives a request from the server for an ip, it talks to tink-server to find a record of that hardware.  If it finds it then it returns it the ip given to it in the hardware definition and returns the ipxe script based on info from that hardware template.

### osie

When boots gives the server an image to boot it gives it the osie image.  When this image boots up.  It has tink-worker.  Tink worker talks back to tink-server to fetch any workflows meant for this device.  It then runs these workflows in series of docker containers.  So you can package all of your scripts/dependencies in a nice neat docker container.

### hegel

Hegel seems to mostly just be a tiny little service sitting over tink-server to return to the server its metadata.  Pretty much like you are probably familar with if you've used AWS. The metadata service.

## Setting up tinkerbell

The easiest way to get started on your local machine is to goto: [https://tinkerbell.org/docs/setup/local-with-vagrant/](https://tinkerbell.org/docs/setup/local-with-vagrant/  "https://tinkerbell.org/docs/setup/local-with-vagrant/") and get started with their vagrant stack.

Of course if you are going to be running in OVH.. This won't really be an option.

### Pre-reqs

So first off you need a couple of things:

* A place to run tinkerbell.  I personally had a proxmox setup for little VM's on a couple of baremetal machines.  But if I didn't I would run a sandbox VM on the public cloud.
* A machine you want to boot over the network
* A vRack.  You're going to want a private network to conduct your experiment on.
* Your machines connected to that vRack.  Make sure you don't turn on dhcp on that vRack doing public cloud.
* An ip range selected.  Personally I tend to over provision:  10.1.0.0/16

### Install tinkerbell machine

Their examples use ubuntu 18.04 so I also chose ubuntu 18.04.  20.04 seems to also work just fine.

Once you have that we need to ssh in and start configuring tinkerbell.

        git clone https://github.com/tinkerbell/sandbox.git

Now we need to hop in to the sandbox folder to start configuring:

        cd sandbox

Now we need to generate the environment variables used for all of the rest of the configuration.

Identify the interface attached to the private interface.  You can use `ip addr` and find the interface with no ip.  Mine was ens18

Then you need to run:

        ./generate-envrc.sh ens18 > envrc

Then you need to edit the file output unless your network choice is 192.168.1.1/24 in which case you can leave alone

If not you'll need to change the following lines to two ips you wish to use:

        # Decide on a subnet for provisioning. Tinkerbell should "own" this
        # network space. Its subnet should be just large enough to be able
        # to provision your hardware.
        export TINKERBELL_CIDR=16
        
        # Host IP is used by provisioner to expose different services such as
        # tink, boots, etc.
        #
        # The host IP should the first IP in the range, and the Nginx IP
        # should be the second address.
        export TINKERBELL_HOST_IP=10.0.5.1
        
        # NGINX IP is used by provisioner to serve files required for iPXE boot
        export TINKERBELL_NGINX_IP=10.0.5.2

Now you need to load the environment variables into the current shell so we can continue executing.

        source ./envrc

Now you need to run:

        ./setup.sh

Once that finishes you will then be able to startup the components needed:

        cd deploy
        docker-compose up -d

**Note:** If you happen to come back and want to interact with the docker-compose in the future make sure to first load the `envrc` file.  If in the deploy folder can just do: `source ../envrc`

Wala the stack is up and running!

### Tweak boots service

I hope to be able to replace this section.  But at the time of this writing.. the boots service uses a command in the ipxe script called "params" which can be found here: [https://github.com/tinkerbell/boots/blob/1e1d60ac32bac18d9b3f1c07611cd3b3613ecec7/ipxe/script.go#L34](https://github.com/tinkerbell/boots/blob/1e1d60ac32bac18d9b3f1c07611cd3b3613ecec7/ipxe/script.go#L34)

In the version of ipxe my servers are running the param command causes things to error out. From what I can tell this just means that ipxe was built with out [PARAM_CMD](https://ipxe.org/buildcfg/param_cmd) build option.  But since these are OVH servers there isn't really anything I can do to ensure its added.

You can track this issue here: https://github.com/tinkerbell/boots/issues/78

To make it work on OVH first follow pre-reqs listed here: https://github.com/tinkerbell/boots/tree/1e1d60ac32bac18d9b3f1c07611cd3b3613ecec7#local-setup you'll need go installed from package manager as well.

    cd ~
    git clone https://github.com/tinkerbell/boots.git
    curl https://clbin.com/12XhH -o /tmp/ovh-boots-ipxe.patch
    make
    docker build -t quay.io/tinkerbell/boots:local-ovh-params-fix .

Then modify: docker-compose.yaml in sandbox/deploy/docker-compose.yaml and change tag to `local-ovh-params-fix`

Then in the sandbox/deploy folder run: `source ../envrc; docker-compose up -d`

### Prepare OVH machine

In OVH by default they have an ipxe server listening on the public interface, and intercept all requests.  So we need to setup a script through them to bootstrap things so we can boot from our LAN.

You'll need API access from here: https://api.us.ovhcloud.com/console

Go down to: `POST /me/ipxeScript` and insert:

    #!ipxe
    
    ifclose net0
    dhcp net1
    set iface net1
    
    chain --autofree http://10.0.5.1/auto.ipxe || exit

![](/images/2020-09-:day/screen-shot-2020-09-07-at-22-18-28.png)

This script will:

* close net0 the public interface.
* get dhcp on net1
* set net1 as the iface
* chain and try to boot the ipxe script from boots.  If it gives 404 then exit.

Then grab the name of machine you wish to use and goto `GET`[`/dedicated/server/{serviceName}/boot`](https://api.us.ovhcloud.com/console/#/dedicated/server/{serviceName}/boot#GET)

Set bootType=ipxeCustomerScript

You'll get an array back.  If you aren't sure which one is which you can call: `GET`[`/dedicated/server/{serviceName}/boot/{bootId}`](https://api.us.ovhcloud.com/console/#/dedicated/server/{serviceName}/boot/{bootId}#GET)

This will tell you the name of the script so you can make sure it matches

Now we need to set our server up to boot this script: `PUT /dedicated/server/{serviceName}`

![](/images/2020-09-:day/screen-shot-2020-09-07-at-22-27-32.png)

### Add info to tinkerbell

Go back to your sandbox/deploy folder and run: `source ../envrc; docker-compose exec tink-cli sh`

Now you have the tink cli available to you.

You need to create and inject a couple of files

#### hardware.json

    { "id":"0fba0bf8-3772-4b4a-ab9f-6ebe93b90a94",
            "metadata":{
                    "facility":{
                            "facility_code":"ewr1",
                            "plan_slug":"c2.medium.x86",
                            "plan_version_slug":""
                    },
                    "instance":{},
                    "state":"provisioning"
            },
            "network":{
                    "interfaces":[
                            {
                                    "dhcp":{
                                            "arch":"x86_64",
                                            "ip":{
                                                    "address":"10.0.6.3",
                                                    "gateway":"10.0.0.1",
                                                    "netmask":"255.255.0.0"
                                            },
                                            "mac":"d0:50:99:d7:63:20",
                                            "uefi":true
                                    },
                                    "netboot":{
                                            "allow_pxe":true,
                                            "allow_workflow":true
                                    }
                            }
                    ]
            }
    }

You'll need to swap out the ip and mac address.

Add this hardware with: `tink hardware push --file hardware.json`

#### hello-world.tmpl

    version: "0.1"
    name: hello_world_workflow
    global_timeout: 600
    tasks:
      - name: "hello world"
        worker: "{{.device_1}}"
        actions:
          - name: "hello_world"
            image: hello-world
            timeout: 60

Add the template with: `tink template create -p hello-world.tmpl`

It'll spit out an id.

Now we need to create a job to run this template.  So we need to create a workflow: `tink workflow create -t {template_id} -r '{"device_1":"d0:50:99:d7:63:20"}'`

Swap out the mac address for the same address used in your hardware.json above.

### Try it

Now for the fun part.  Connect to KVM console for your machine and then reboot it.

If all went well you should see it boot up