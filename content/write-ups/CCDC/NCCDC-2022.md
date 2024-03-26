---
title: "NCCDC 2022"
date: 2024-03-25t19:55:10-08:00
draft: false
---

# How did we get here?

It started with a random tryout with the OSUSEC club to see who might be eligible to compete in the competition. This ended with me getting my spot on the team learning anything I can about cyber defence. I had been working on binary exploitation at the time, so I had no idea what to expect for this. They told me to look up the best ways to secure a wordpress site as well as an nginx site. These were both valid entry points of an attacker, so I went to work and eventually did a presentation on the subject.

Soon after was the regional competition, PRCCDC. This was the competition that would decide if we went to nationals or not. Only the winnner of regionals gets to go to the nationals. Clearly by me writing this, we won it. There wasn't much to say about that since the infra was messed up and the red team was entirely unable to attack us as a result. Shortly following this, we realized we had to step up our game and really practice some concepts. We decided to do a mock competition with one of the other teams going to nationals along with the rest of their club teams. This was much more interactive than the PRCCDC and we were able to learn a lot about our actual security and what to expect during the competition.

This is where we were introduced to 3snake: a binary that runs on the root account that can listen to all ssh connections as well as sudo calls all in plaintext. We found that having this running on anything would completely compromise our passwords. For me, this was a clear indication that our passwords needed to be reset when we were not 100% sure the system was secured.

# Getting to the competition

This is one thing that surprised me. The sponsors of NCCDC paid for everyone who made it to fly to San Antonio and and stay in the hotel. This includes the flight there and back, the fancy hotel, and the meals there as well. I soon realized that this is a high profile event. These sponsors understand that the people who make it to NCCDC are some of the best security students in the nation making them massive targets for recruiting.

In the days leading up to the competition, we were studying regularly to hone our skills in defending systems. I built a practice range to practice running different scripts and changing configs to better secure our systems. This was pretty useful and I wish I utilized the practice range more to really dive into my systems.

# Day One

Okay I'm skipping a lot, but the gist of it is that we made it there without any hiccups and checked in to our hotel. We spent that night just relaxing before the stressful two days that we knew were ahead of us. One thing we made sure to do was pass out password sheets to everyone on the team. These passwords were multiple words long with special rules to string these words together and make them complex.

Waking up at 7:00AM (San Antonio time) was difficult, but we had no time to complain. We went to our designated breakfast location and listened to the briefing on the details of the competition. This is where we got our first look at the team packets which included: usernames, passwords, ip addresses, ip ranges, and more including the whole concept of the organization we were "working for". Let me go into further detail on the specifics of this.

We were introduced to the company "Capissen", a video game company that hosts video game servers and sells video game peripherals. We were in charge of keeping this game company working despite persistent attacks from the red team.  
The services included:
- An e-commerce website for selling video game products An internal ERP  
- An internal HRM system  
- 5 separate video game servers
- 2 SSH hosts (one internal and one in the cloud) 3 hosts for controlling Point-Of-Sale  
- A VOIP phone for customer service  
- A DNS server for resolving our hostname. Windows AD and DC
- Palo Alto firewalls on local and remote networks.  
This is as much as I can remember right now, but there was more
These were separated into 3 separate subnets:
- 10.70.70.0/24 was our local subnet hosting our website and internal services.  
- 172.16.70.0/24 was our "cloud" subnet which was just an esxi instance running the game servers and a couple other things.  
- 172.16.75.0/24 was our other "cloud" subnet in charge of the Point-of-Sale systems. Most of this was on windows, so I'm not really sure about it.

So we get started at 9:00AM. Every team is sitting ouside their team rooms ready to be let in and everyone gets let in at the same time. We walk in and see 8 workstation laptops and 2 other laptops that we weren't really sure about. Going around the room, we realized 6 of the laptops were windows and 2 were linux. CRAP. We had 3 linux guys and only 2 linux workstations. I elected to be the one on the windows workstation as I probably have the most experience on windows out of us 3. The other 2 laptops were an ESXI hypervisor (running our local virtualized servers) and a blank laptop with nothing installed on it. We were told that we could do anything we would like to this extra laptop.

At the start, we were told that one service (E-Commerce) was down and we had to fix it as fast as possible in order to lose the least amount of points. Each minute the site was down, meant more points lost for us. Logging into this host, I realized there was a problem with the SQL database connection on the website. I attempted to login to the database to find out that the login information was incorrect. This was bad because the only way to get into the database was to reset it. I realized I had to consult with Robert to get this done. I told robert the problem and he went to work resetting the database to a stable state. It took him a while, but he eventually got it fixed and we were in the green.

I realized that while robert was hard at work fixing the database, I had nothing pressing to do. I decided to start somewhere by logging into as many linux hosts as I could to change passwords fix any

configuation issues. I realized very quickly that entering in these passwords with the rules we set and the long length was extremely hard in this high-stress environment. This ended in me becoming extremely frustrated and I had to work extremely hard to keep myself together and focus on the task at hand.

About an hour into the competition was the first time we caught the red team in our linux VMs. They had logged in with default credentials to a user account. Soon after, they had gained root privileges on all of our VMs and began deploying their malware. While checking the user "monty" I noticed in their bash history that they had run a program called "thepoo". Seems like just some random binary, but it turned out to be extremely malicious. The red team proceeded to deploy the malware on the majority of our systems as there was something that allowed them to gain root access to the VMs.

At this point we had no idea what was happening. We just assumed there was something weird going on and I had to fight some other fires. At this point, we were having some issues with our local ESXI login. We decided to reset the password manually and realized this took longer than we anticipated. There were multiple services down on this hypervisor and we had no access to it. This lost us a ton of points because point losses were higher at the beginning of the competition.

Shortly after that, we had an issue with our "cloud" infrastructure. All of our services were down on the hypervisor and the system had slowed to nearly a halt. We spent a ton of valuable time trying to figure out the solution before realizing the problem. Turns out, one of the teammates had turned on the Palo Alto VM in the cloud esxi without configuring it. This meant that the VM did not have an IP assigned and started spamming DHCP requests to the hypervisor's network. This caused the network to slow down to a halt and prevent access to any of the VMs. We lost a ton of points because of this. Our entire infrastructure was broken at that point. To add to this, our E-Comm server was still broken.

Robert had realized that in order to fix the database, he had to completely reinstall mysql on the VM. This was problematic though because the repository for the machine was misconfigured forcing him to reconfigure the repositories to the point where he could reinstall mysql. After this happened, it was as simple as starting up the sql server and importing the .sql backup file. There was a problem with the backup though. We had a bad database and realized there was an inject requesting a ransom (points) to get our database back. Well that sucks. So overall this whole problem cost us A LOT of points.

Following this, we decided to start looking for what the heck the red team had done while we were busy fixing all these issues. I logged into one of the VMs and realized the red team was online. I decided to start a wall conversation with this guy:
```
RT: Heyyy wanna be friends?
Me: Yes! Only if you tell me your persistence method.
RT: I'll tell you on Saturday :)
RT: Will you go to prom with me? Y | N
Me: Y <3
RT: YES!!! We'll have so much fun and we can go as friends!
Me: Maybe we can be more than friends <3
RT: YES Of course!!
RT: Lets play a game together!
Me: Okay! lets build a house in minecraft

Me: Hey buddy

Me: Maybe we can put our beds next to each other <3

RT: OMG I would love that! Lets do it!
RT: You gotta turn it to offline mode for us to play though. I don't have an account.
Me: My team has informed me that I'm not allowed to do that :/

RT: Hmmm... Maybe we can play another game
```
At this point we had wasted enough of each other's time and I decided to end the conversation. 

Anyways...

We looked through the VMs and realized that there were a TON of misconfigurations that we knew we had not touched. First, there were ssh keys littered in tons of home user accounts as well as weird places like the /bin/.ssh folder which did not exist before. Next, there were tons of misonfigurations in the /etc/pam.d/su , /etc/ssh/sshd_config and others that allowed all users to login as root and do anything they liked. YIKES. We changed all these configs to the correct state and removed the ssh keys. A quick "ls" revealed that our problem was a lot worse than we thought. Almost immediately after removing these files and fixing configs, they were all back to their original state. This is about when day one was comming to an end. We did a little bit of investigating, but we realized we needed to mitigate the effects of this malware. Gabriel had the incredible idea of removing the services that the malware was using. Privilege escalation capabilities were now removed from the system and the only way to login to the VM was with root and password. This meant that the ssh keys were useless and the privilege escalation was impossible. We also added a line to the sshd_config that only allowed connections from our IP addresses. We obviously couldn't do that on the scored ssh VM's, but that severely lessened the attack surface of the SSH service.

Recap of our mitigations: 
- Remove privilege escalation capabilities.
- Configure ssh to only allow root login with password. 
- Restrict access to only certain source IP's.

Soon after this, the first day had ended and we all took a big sigh of relief. The competition was halfway over and it was time to relax. We had dinner and chatted with other teams before heading to one of our rooms to do a first-day-meeting with the team. We decided to change our password scheme to something much more simple as well as reset all current passwords. We also synced up all of our current passwords to make sure we didn't have to ask all the time what passwords were.

This was our plan going into day two: Finish deploying mitigations and change all current passwords to our new, updated scheme.

# Day Two

This day started almost exactly like the first. We got up and went to breakfast where they announced the active rankings of teams. We were barely on the board and only made ourselves known on the customer service side of things (we were 1st for Orange/White team).

First thing as we went into today was resetting everything we said we would and making sure all of our services were up and running. My next step was to configure the extra laptop to my liking. The day before, my teammates had installed debian on it to use as a workstation and a Wazuh host. I decided to install some tools to work on some reverse engineering injects that we had been given. Yes, even here we had reverse engineering. I can't escape it. I honestly couldn't really figure any of them out and the only one I successfully cracked was an instruction counting program that built a password based on our input and we had to get the password to match the original. Turns out the answer was "FISHTANK". I could have written a program that would do this for me, but we just kept trying things until it worked.

### Alex Levinston's Malware

Halfway through the day, Gabriel and I decided to start digging. We wanted to find the malware that was causing this insanely complex persistence on almost all of our linux VMs. We started by just looking at the running processes to see what was happening. We didn't see anything unusual, so we had to go to the files to see exactly how complex this really was. We wanted to figure out what process was replacing all these files, so we started by setting up an "inotifywait" command with "lsof" to monitor the processes. This ended up not working because the malware was too fast for the command to see it and we couldn't get any valid results. I did some googling (which was hard because we had restricted access to the internet) and found that there's a magical tool called "fatrace" that was exactly what we wanted AND it was in the debian repositories so we were able to install it with apt. Running this program would watch every single file on the system and list the process that was making changes to them. PERFECT. After running this with some of our tests, we found that there was a process (ipmievd) that was rewriting all of these files. We decided to delete this program and kill any existing process with the program only to realize another program was doing the same exact thing. So Gabriel and I went to work building a command that would delete all of the programs that we had names for and then we would watch fatrace to see what new program would take their place. We realized that these programs were rewriting themselves and each time we removed them, they would come right back. Each time we removed any of the malicious programs, the remaining programs replace them right in the same spot. After following functionality of the malware 10 binaries deep, we finally had exhausted all the names of the malicious binaries and we were able to remove it all in one command. Watching fatrace revealed that this had completely stopped the malware from running. By the end of this whole event, the competition was about to end, so we decided to go try to secure the rest of the VMs just for fun. We realized that these binary names were not the same on each of the VMs, so we were unable to secure all of them before the competition ended.

And that was it. That was the competition. We packed up everything in the team room and went to get ready for the networking night.

Networking Night

We were told to bring our resumes to this event, wear business casual, and talk to as many people as possible because everyone wants to hire us apparently. Afterwards, I realized that the vast majority of these organizations were government contractors or large corporations WOW.

We walked in and instantly I chose to go to the Scale.api booth. The first person we talked to turned out to be Alex Levinston, the red team that was going against us. He explained that it was him that wrote the malware. He had spent the past week straight writing 8000 liines of code to create a polymorphic, multi-binary malware. This thing was embedded into everything. The fact that we were able to eradicate it on one machine was impressive since the malware had a giant list of program names that very well could have been used for the malicious binary. After telling him about us finding the malware, he seemed impressed with our abilities. Kinda feel good about that. Also, he explained how frustrated he got when we deleted his privilage escalation technique.

I proceeded to go around talking to all the other organizations that were sponsoring this event. This lead to me realizing that they really want to hire us. We exchanged contact information and I handed out a good amount of resumes.

Panopoly

This was a bust. We spent all night the night before figuring out a good script for persistence in the machine and didn't even think about how we'd actually get in. Turns out the initial "getting in" part was the hard part and we never ended up succeeding.  
