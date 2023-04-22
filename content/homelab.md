---
title: "Homelab"
date: 2023-04-18T01:25:53-08:00
draft: false
---

# How it started

I initially started my homelab with the intention to get a network-wide
adblock set up for my network. I didn't really like having to look at ads
everywhere I went on the internet. I decided to install [pi-hole](https://pi-hole.net/) on an old
raspberry pi. This is what fuled my need for MORE! From there I found the /r/homelab subreddit and
/r/selfhosted subreddit. These forums helped me figure out what to do with my homelab.
Since I am a cybersecurity student, I obviously wanted to use my homelab to improve my skills in
cybersecurity.

# How it's going

I continued down the path to obtain more and more server hardware until I felt I had something I
could use as a training lab of sorts. So I'll just explain how I've set up the homelab. 
All of my computability is contained within 2 server boxes:

### Server #1 (pwn)
This is my pwn box. This is the box that is always available to me with all of the necessary
tools to practice on. All of the necessary tools for CTF challenges are on this box, so I can
log into this server from anywhere in the world in order to have the same environment in every
scenario. This is also exposed to the internet through specific ports in order to host test
instances for CTF challenges and training endpoints for practice.

### Server #2 (Proxmox)
This is my virtualization server that I can use to spin up different OS's as needed. I currently
only have two virtual machines running on this server:
- Machine 1 is running Windows server 2016. I only use this when I have a windows-specific task
(like running an exe or windows forensics)
- Machine 2 is running my Web Production environment. That's how this website is being hosted. I
also have a few other public servers running on my production environment. These are mostly monitoring
tools like uptime-kuma and a local instance of netdata.

### Other hardware
My router is an old gaming PC that I've repurposed. I got sick of the how slow my router was anytime I 
wanted to do anything. It got to the point where I had to reboot my router every day to retain any sense
of network capabilities. So I grabbed some old hardware and happened to have a 10G NIC which was perfect
for what I wanted. I ended up installing [OPNSense](https://opnsense.org/) on it and have been in love
with it ever since. I did have one hiccup once but I'll explain that in the "What went wrong" section.
My old Netgear router is now acting as my wireless access point. Great use for a $100 router... 
I also still have an old raspberry pi, but it currently is sitting on my network awaiting my next command.
At one point I was running 2 old dell r710 servers which were amazing at everything I asked of them, but
I couldn't have them in my new apartment, so I've since retired them to storage.

### Cloud Infrastructure
I currently have a single AWS instance running as my network entrypoint. Anything that want's to access my
network has to do it through this server. Since this is my network's entrypoint, it has the highest security
with 24/7 monitoring using netdata. I also plan to install Wazuh agent and run an elastic stack on my network
to practice endpoint monitoring and analyze attack patterns. This server also runs [Nginx Proxy Manager](https://nginxproxymanager.com/) To act as the reverse proxy for the 3 domain names that I manage. I have a few subdomains
pointing to different servers on my network and all of that can be easily managed my NPM.

### Tailscale
All of this would be much harder without tailscale. I previously had basic wireguard set up on all of these
servers and it was such a pain any time I wanted to add a new host to the network. This is where Tailscale
comes in. All I have to do is run the install script on any host and link it to my network with the token. 
I now have a new host on the network. I can also change the access controls through the admin console
preventing certain services from being accessed by certain hosts. This can be useful when running challenges
for CTF's that are not very secure.

Tailscale is essentially how I get anything done these days. I don't have to worry about opening holes in
my home network anymore. I just use the hole-punching technique that tailscale devs creates to have a p2p
vpn network without open ports. Then anything I want accessable to the public just gets routed through the
reverse proxy that I have set up in AWS.

# What Went Wrong
So there are tons of stories that I could talk about from my experience with my homelab but two stories.
Stick out like a sore thumb.

#### The VM router???
My original instance of OPNSense was running in a proxmox VM. Yes, A VM. So all of my routing was being routed
through the virtual network. This wasn't much of a problem for a while and it actually had great performance. 
Then the power went out... Since Proxmox has a web interface, the only way to access it was using the ip given
by the DHCP server running in the router vm. This became a problem because the filesystem had corrupted when
the power went out and the router vm wouldn't start. This meant that I didn't have any internet, and I couldn't 
access the host that was running the vm in order to fix the filesystem. This was a loop of of absolute terror. 
I ended up having to completely wipe the bare metal host and install the router OS the manual way (luckily I had 
an old copy of the OPNSense iso on a flashdrive). I was back up and running within the day, but that was a day of 
pure panic trying to get my network back online.

I learned a lot from this though. Two major points are:
- Always have you're essential services on a UPS backup because a power surge/outtage can definitely corrupt your filesystem.
- NEVER EVER put your router OS in a VM. Especially a vm that is headless/web accessable.

#### What does hot swap mean???
Just my the title of this, you might know what happened. I was using ESXI to host numerous virtual machines on my 
r710 servers back when I was using those still. I had multiple VM's that I was using for some classes and they took 
a little while to set up. I had just gotten a new NAS box and I was excited to test it out and start storing some 
stuff on it, but I didn't have any hard drives to put in. Then I thought "hey my r710's are not using all of 
its storage and The drive bays are hotswappable. Why not just take a couple out." This is when I learned about 
how RAID *actually* works. I decided to pull one of the drives WHILE IT WAS RUNNING and realized what I had done 
after it was too late. I had completely corrupted the filesystem of the entire server and destroyed every VM in 
the process. I had to learn how to communicate with the ancient iDRAC interface in order to rebuild the filesystem 
just to get the server to boot again. Then I had to rebuild each of my class VM's all before my class which happened 
to be in the morning of the next day. Luckily this was early in the term and we had not done much work in these VM's. 
Otherwise I don't know what I would have done. This was just a simple mistake that easily could have been avoided 
with a quick google search. Just a fun horror story I like to tell about my homelab. 

# Next Steps
Well, the sky is the limit. I'm going to continue using this homelab for my cybersecurity research and training, 
but I'll likely start focussing on monitoring the network and making sure it's robust enough for my liking. 
For now, this is what it looks like. 

***Thanks for reading!***
