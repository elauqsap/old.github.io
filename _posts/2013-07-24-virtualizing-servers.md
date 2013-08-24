---
layout: post
title: "Virtualizing Servers"
description: ""
category: virtualization 
tags: [virtualization, sys admin]
---
{% include JB/setup %}
This week I decided to take the leap into virtualization and installed ESXi from VMware on my desktop I built in 2009. I wanted to create a workstation that I could easily run three VMs simultaneously. I think this is an amazing tool that will help me learn a lot about networking, security, and system administration. With the hypervisor we can allocate resources how we want them, which allows for greater control of the system. Anyways this post will address how I set up my VMs and what I plan to do with them. I won't go into installing the hypervisor because I didn't do anything thing custom, just installed and opened up the vSphere manager on a Windows host (gripe).

The three VMs that I will be running primarily are a NAS, Debian, and Untangle server. The idea is that the NAS will provide storage to some of the services running on the Debian server as well as some of the hosts on the network. Then the Debian VM will provide media streaming, mail routing, and any other services I wish to provide later. Leading to the Untangle VM which will provide routing for the network as well as a VPN, IPS, Firewall, Spam/Phish Blocker, and a reporting system. This post will be a main overview, but I can go into greater config detail in later posts.



**ESXi host:**
- Intel Core 2 Quad Q9400 @ 2.66 GHz
- 8GB DDR2
- 160GB HD
- 2 x 500GB @ 7200 rpm
- 1 x 500GB @ 5400 rpm
- NVIDIA GTS 250



**Untangle guest:**
- 1 core
- 2GB RAM
- 30GB vmdk of the 160GB drive


**FreeNAS guest:**
- 2 core
- 4GB RAM
- 4GB vmdk of the 160GB drive
- 2 x 500GB @ 7200 rpm ( for the ZFS mirror )
- 300GB of the 500GB drive



**Debian guest:**
- 2 core
- 2GB RAM
- 40GB vmdk of the 160GB drive



I needed to give the NAS guest at least 4GB of RAM to allow the ZFS mirror to perform at its best and then I split the remaining. I believe there is away for the host to reclaim allocated free RAM from the guest but I will worry about that if I add more guests to the host. I didn’t know this at the time and I still need to read into it more but giving more cores to a guest doesn’t mean it will run better. Each vCPU is mapped to one core so the more you have the harder it will be for the processes to run by the scheduler. So keep this in mind when building your system, I may have to review my configuration later on.  For storage I just gave each guest what I thought would be enough for it to run properly. The NAS will contain most of the storage, as I want to provide redundancy for some of my important data as well as provide Time Machine backups via the extra drive.

Since we are already talking about the NAS I would like to continue its role in my network. I will be using FreeNAS 9.1.0 to provide DynamicDNS, ZFS, NFS, and CIFS shares. I won’t be using the AFP share because backing up is already slow enough. I will just configure my Macbook to look for other drives for Time Machine. I created two zpools, which are my sharing and backup volumes. Normally it would be better to back up your system to the redundant zpool but I decided I already have a network and portable drive dedicated to Time Machine. So the share zpool is the ZFS mirror of the matching 500GB drives and the backup pool is the 300GB provisioned ZFS stripe. I then created two datasets in the share volume for personally and public use. The backup volume only has one dataset currently but I will add another in the future for log backups. On these zpools I enabled lz4 compression to get a little extra out of my drives. After that I created exports to my network so I could mount the drives. All I had to do was turn on the sharing services and enable my DynamicDNS.

The Debian guest currently has a mail server setup using exim4 and courier to provide SMTP and IMAPS. Besides mail the system will also be a syslog and media server. I am running a Plex Media Server currently and I have to say it is going to make home media streaming pretty neat. The share zpool is mounted on this system and feeds music, movies, and TV shows to the media server. 

Finally the brain of the network is the Untangle guest. Which provides most of the network functionality like DNS, DHCP, and is wired to both of the NICs. I have mine configured as a router so there is an external and an internal network. All of the guests as well as a wireless router for the physical clients are on the internal NIC. I had a little bit of trouble getting the networking going for this but I found that if you push the VMkernel for management onto the internal NIC you won’t have any problems getting a public IP to the Untangle guest on the external NIC. This means that the ESXi host gets an internal IP from the guest and is accessible via that network or externally by using the VPN service. After getting the networking figured out all you have to do is turn on the services you want and with some minor tweaks it is running in no time.

I will get more into configuration once I have more time to play with it but I think this a pretty cool little setup. If you have any suggestions or recommendations please send them my way!
