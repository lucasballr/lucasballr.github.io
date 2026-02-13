---
title: "PRCCDC 2022"
date: 2024-03-25T19:55:10-08:00
draft: false
---

Okay so I'm writing this two years after the fact, but I figured I'd talk about my experience. The 2024 version is much more in-depth than this one.

### Why would I do this?
I essentially was trying to look into ways to get back into the whole hacker / computer science space and decided to look into OSUSEC again. I had originally joined the discord server in 2017 because I had an interest in cyber, but I didn't really have the guidance/motivation to participate. Fast-forward to 2021, I started showing up to meetings and getting to know everyone. I started joining in on CTF meetings and got introduced to the idea of the cyber defense competitions. I decided to try out for the CDC team and managed to get on it. We were scheduled to compete in the 2022 PRCCDC, so we got to practicing.

### Training
I was designated to the Linux side (which is what I primarily enjoy). I didn't really know what I was doing though. I was pretty new to the field and it got overwhelming pretty fast. My teammates were absultely amazing though and I have them to thank for the majority of the things I know today. Writing this two years later, I'm realizing that this was likely the tipping point for me to go all in on cybersecurity. I started learning about every aspect of the linux operating system all the way from the kernel to the specific services that are commonly deployed through Linux. My primary teammates Gabriel and Robert Were hugely instrumental in my learning. Both are near genius level when it comes to the Linux operating system and they were kind enough to teach me anything and everything I wanted to know. FYI, both have since taken very prestegious positions at various organizations. 

Anyways, our training consisted of practicing basic linux terminal commands and strategies to lock down the operating system. These included mostly basic commands like `ufw` and `kill -9`. Basic commands that any proficient linux user would find easy, but were mostly new to me. 

### The event
PRCCDC was remote this year. COVID rules were still in place, so the competition organizers made the decision to make the competition remote with the stipulation that we needed to be live on camera the entire time. This allowed us to set up an incredible war room with tons of monitors and plenty of computers to do anything and everything we wanted. 

### t0
We are given an openvpn config file and a list of hosts with respective IP addresses in our team packets and the competition starts. Almost immediately, we realize there is nothing online. The only thing we were able to really find was a basic wordpress service working on one of the linux servers. This was subsequently secured very quickly mainly by changing the password and verifying the proper version. An issue we found, though, was that all of the IP addresses for the content on the site were incorrect. Simply changing the config to fix the primary URL was not sufficient, so I had to change every link on the site. Luckily, it wasn't vulnerable so I wasn't bothered by the red team at all. 

### t1 
This was 2 years ago, so there's a lot I'm forgetting about this event. What I can remember is getting shells on every single linux VM in a different window. My teammates had previously explained multiple strategies for monitoring the activity on the box. I monitored the wordpress logs and connected users for any activity and noticed a couple things happening. Red team activity was pretty obvious since the scoring system made the same request at a regular interval. Any time we observed anything happening, we investigated everything and created an incident response report for it. I think the hardest part about the rest of the competition was the injects that had us deploying new services or adding new content to the wordpress site.

### Reflection
I think it's entirely possible that this event sparked something in me to go all in on cybersecurity. The camaraderie and excitement of this intense competition made me extremely excited to be a part of this new community. It made me excited for the future of my career in cybersecurity.  

