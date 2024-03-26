+++
authors = ["laozi"]
title = "misc/ebe"
date = "2023-02-12"
+++

### The challenge

This was just a pcap that had certain flags set for the incorrect
UDP packets. All you had to to was look at the RFC definition and filter by
that ip flag. `ip.flags.rb == 0`. Then you can just export bytes or read the
flag one character at a time.
