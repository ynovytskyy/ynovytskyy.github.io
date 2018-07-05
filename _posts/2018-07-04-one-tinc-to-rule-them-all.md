---
layout: post
title:  "One tinc to rule them all"
---

It's a pretty useful and common practice to allow low level communication
between the servers in a safe environment. Also referred to as `p2p` or `mesh`
network.

Use cases:
- Joining servers from different networks into one logical network
  (e.g. to form a cluster)
- Cross-cloud network
- Migrations
- Other ad-hoc network setups when cloud provider's virtual networking
comes short or is not applicable

One of the ad-hoc cases, that I found this technique useful for, is when one of
the servers does not have external IP address. Using this approach it will be
possible for such server to join this virtual network and for public server to
potentially route traffic from public internet to it though the VPN.

> Note, I know about port forwarding ;) Sometimes you don't control routes,
but you can still use this approach.

We will achieve this with `tinc` VPN https://www.tinc-vpn.org/

I found `tinc` very stable, fast, light on resources. It maintains connections
and self-heals.

We'll create a VPN network called `middle-earth` with a central server `erebor`.
It is going to be the server to which all the other servers that participate
in this VPN will connect to, in particular `hobbiton`

### Central server setup
This is going to be the server that all the other participating servers will
connect to. In my case it is running CentOS 6.

**Important:** central server is the only one that needs external IP address
(or other IP address that makes it accessible to other servers). And port `655`
should be open for TCP traffic for it. Port can be changed in config, but this
is not covered here. Also `tinc` will detect if UDP traffic is possible and
will use it optionally - UDP traffic is not required.

So let's get going. On `erebor`:
```bash
sudo yum install tinc

sudo mkdir -p /etc/tinc/middle-earth/hosts

echo 'Name = erebor
AddressFamily = ipv4
Interface = tun0' | sudo tee /etc/tinc/middle-earth/tinc.conf

echo 'Address = erebor.public.ip.or.domain.address
Subnet = 10.0.0.1/32' | sudo tee /etc/tinc/middle-earth/hosts/erebor

sudo tincd -n middle-earth -K4096

echo '#!/bin/sh
ifconfig $INTERFACE 10.0.0.1 netmask 255.255.255.0' | sudo tee /etc/tinc/middle-earth/tinc-up

echo '#!/bin/sh
ifconfig $INTERFACE down' | sudo tee /etc/tinc/middle-earth/tinc-down

sudo chmod 755 /etc/tinc/middle-earth/tinc-*
```

### Other participating node (Ubuntu 18 in my case)
On `hobbiton`:
```bash
sudo apt install tinc

sudo mkdir -p /etc/tinc/middle-earth/hosts

echo 'Name = hobbiton
AddressFamily = ipv4
Interface = tun0
ConnectTo = erebor' | sudo tee /etc/tinc/middle-earth/tinc.conf

echo 'Subnet = 10.0.0.3/32' | sudo tee /etc/tinc/middle-earth/hosts/hobbiton

sudo tincd -n middle-earth -K4096

echo '#!/bin/sh
ifconfig $INTERFACE 10.0.0.3 netmask 255.255.255.0' | sudo tee /etc/tinc/middle-earth/tinc-up

echo '#!/bin/sh
ifconfig $INTERFACE down' | sudo tee /etc/tinc/middle-earth/tinc-down

sudo chmod 755 /etc/tinc/middle-earth/tinc-*
```

> Note that on this stage none of the servers is actually running `tinc` yet

## Exchange keys
Keys need to be exchanged between each participating server (e.g. `hobbiton`)
and the central server `erebor`.

#### From `hobbiton` to `erebor`
On `hobbiton`:
```bash
scp /etc/tinc/middle-earth/hosts/hobbiton user@erebor.public.ip.or.domain.address:/tmp
```
On `erebor`:
```bash
sudo mv /tmp/hobbiton /etc/tinc/middle-earth/hosts/
```

#### Vice-versa from `erebor` to `hobbiton`
On `erebor`:
```bash
scp /etc/tinc/middle-earth/hosts/erebor user@hobbiton.public.ip.or.domain.address:/tmp
```
On `hobbiton`:
```bash
sudo mv /tmp/erebor /etc/tinc/middle-earth/hosts/
```

## Test it
We'll start `tinc` on `erebor` first. The following command is going to run it
in the terminal in the foreground and show debug info so that we can make sure
that all works fine. Run the following on `erebor`:
```bash
sudo tincd -n middle-earth -D -d3
```
After it's started on the central server, let's run the same on `hobbiton`:
```bash
sudo tincd -n middle-earth -D -d3
```
After both are running we should be able to ping each other on `10.0.0.*` network.
You need a separate ssh or terminal session for this as you don't want to stop
current `tinc` sessions running in the foreground on both servers.

So on `erebor` run `ping 10.0.0.3` to ping `hobbiton` via VPN and on `hobbiton`
run `ping 10.0.0.1` to ping `erebor` via VPN. Both should work successfully at
this point.

After the successful test, press `Ctrl+\` in the `tinc` foreground sessions to
stop them. We are now going to configure `tinc` as a service and start it at
the boot time.

## Start `tinc` on boot
Central server `erebor` is running CentOS 6 (yeah, old) in this example
and therefore needs extra work to configure `tincd` as a service on startup:
```bash
echo 'tincd -n middle-earth' | sudo tee --append /etc/rc.d/rc.local

#and just to start tinc just this time without restarting the server:
sudo tincd -n middle-earth
```
On `hobbiton` which is Ubuntu 18 it's much simpler out of the box:
```bash
sudo systemctl enable tinc@middle-earth
sudo systemctl start tinc@middle-earth
```

## Happy virtual networking!
