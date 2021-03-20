---
title: 'Virtualbox with static and dynamic IP'
date: Sat, 07 Apr 2018 14:49:50 +0000
draft: false
tags: ['docker', 'fixedIP', 'ubuntu', 'Uncategorized', 'virtualbox']
---

While installing docker on my windows 10 machine I found that Docker for [Windows requires 64-bit Windows 10 Pro with Hyper-V available](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install), what? omg, I was expecting something simple... but NISBEIP, let's move on. Tried to use the Docker Toolbox that didn't work as smooth as I expected. After some hours banging my head against the wall I decided to create a Linux Virtual Machine using Virtualbox and use docker from there. But now here it comes the new challenge: How do I access the internet from the VM or even from the docker inside the VM and use a name to connect to it? I started to think about some crazy ideas

*   Updatable Local DNS, Then I would need to setup my DNS use it, and keep track of a list of name and IP, and the VM would need to update me with the new IP, way too much work, no!
*   Setup the router DHCP to reserve an IP for that specific VM, not gonna work I will have lots of machines.
*   Access the machines with the IP address that I would need to check and write it down somewhere... NO!
*   Update hosts file everytime a new VM starts or is rebooted, OMG NO.

Then [I saw a video](https://www.youtube.com/watch?v=S7jD6nnYJy0&t=329s) from [Corey Schaffer](https://www.youtube.com/channel/UCCezIgC97PvUuR4_gbFUs5g) talking about a network of machines with VirtualBox, then the Aha moment happened: KEEP TWO NETWORK INTERFACES IN THE VM!! ![](/images/2018/04/virtualbox-static-and-dynamic-ip.jpg) Here are the oversimplified steps to do it:

*   Create the new VM and install [Ubuntu Server](https://www.ubuntu.com/download/server), or clone an existing one.
*   At the VM Settings/Network keep TWO Adapters, first one Attached to Bridged Adapter, second one attached to Host-only Adapter, the second one will have the fixed IP
*   Log in to the machine and install OpenSSH - you will need to log in from the windows machine to your virtual machine, I find this easier than login using the VirtualBox screen, copy and paste there is a nightmare.
*   Setup the fixed IP for the Host-only network interface

Setting up the fixed IP
-----------------------

Make sure the Host-only network interface doesn't have DHCP server activated and there is a fixed IP assigned to the host machine, check this at Virtual box menu, Host Network Manager, yes this one is tricky it's not a Virtual Machine setting, is a Virtualbox setting. ![](/images/2018/04/virtualbox_host_network_manager.png) Now that your Host-only network adapter is pretty much useless, let's make it usable by assigning an IP to the 192.168.56.X network. Using the **ifconfig -a** identify the interface name that doesn't have an IP assigned, mine was **enp0s8.** edit the file **/etc/network/interfaces** and add the following lines at the end

```
auto enp0s8
iface enp0s8 inet static
address 192.168.56.100
netmask 255.255.255.0
```

restart your network service by using:

```
sudo service networkìng restart
```

And now your Virtual machine is accessible thru the IP 192.168.56.100, add this to the hosts file at the windows machine and you can ssh to the machine that will have a fixed AND a dynamic IP from the Bridged network interface.