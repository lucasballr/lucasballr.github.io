+++
authors = ["laozi"]
title = "misc/a-hackers-notes"
date = "2023-02-12"
+++

### The Challenge

You are given a luks encrypted drive that has a pretty weak password on it.
All you have to do to open it is run a bruteforce script on the drive:

```bash
bruteforce-luks -t 6 -f dictionary.txt -v 30 hackers-drive.dd
```

dictionary.txt had a bunch of variations of "hackerXXX" which is the format
we are told the password is in. The result is "hacker765" as the password.
Then you need to mount the drive to take a look at it:

```bash
sudo cryptsetup luksOpen hackers-drive.dd tmp
sudo mount /dev/mapper/tmp ./asdf
```

This turns out to be a joplin notebook that is encrypted with joplin's e2ee
encryption. The joplin sql database holds the master password in it, so you
can easily pull that password: Encryption master pw: "n72ROU9BqbjVOlXKH5Ju"

The next step was kind of difficult. Since the joplin-cli is not easy to work
with, I could not figure out how to actually open the joplin notebook. Instead,
I opened up the joplin app and set my "master-password" to the above string.
Then I chose to "sync with filesystem" and chose the directory of the hacker's
notebook. This ended up automatically decrypting the notebook in turn decrypting
the flag.
