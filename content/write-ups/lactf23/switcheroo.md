+++
authors = ["laozi"]
title = "rev/switcheroo"
date = "2023-02-12"
+++

### The challenge

The was a switch statement written in assembly. I'm not amazing at REV,
so I was a little bit slow at this. Essentially the concept was to have
a region of memory that acted as a "key". Each character of the user input
would act as an index to an 8-byte region of this memory. Then it would take
the 8 bytes and do some math on it. I was a little hazy on the details, but
essentially it would to an 8-bit rotation of the memory and grab the last byte
of that value and compare it to the running index count. If it was larger than
the index, it would rotate again. If it was smaller than the index, it would
move on to the next input and that char would be considered correct. I used
this knowledge to create most of the flag, but there were some issues with it
since the program didn't necessarily do this exact algorithm. I was able to fill
in the rest of the flag by testing out different rotation counts for each step.
Here's my exploit code. It's pretty messy, but it got it done.

```python
#! /usr/bin/env python

x = [0x0700010203040506, 0x3e04f4fb783cfc7e,\
0x9d70813d04308703,     0x4a51a1f8093da74d,\
0x0c4950b2a88b02fc,     0x02d21d9262f0fd4b,\
0x88e781beaabe46a9,     0x77090f0e85bb1994,\
0x564eb824353f49fb,     0x7f0b7a9deb629b47,\
0x013fb130146e18d1,     0xe650e17e4dd465ed,\
0x81d36704803dabe7,     0x02ce80a373718c7a,\
0x822c510930254873,     0x570837475070684e,\
0x3bd734ec321014b5,     0x975e1ee37edd98b3,\
0x158144feb60d4cfd,     0x88407c6cac154291,\
0x0fae192b920dfc1b,     0xfc0c676e7c762680,\
0x4850b8d36d94ef27,     0x086466720d884da2,\
0xac7e0453c14f48dc,     0x7c2568ad2461dfd8,\
0x473b065dde911f08,     0x1eed1fd742e65871,\
0x6c81edc3c1b82548,     0x03169f71b61802ff,\
0xe4a7396792ae3fe0,     0x90cf08ac6b10b7ca,\
0x004ca10931f081a0,     0x00f53df18f89683b,\
0x004242bcb1de8f49,     0x007e5900ceac84ca,\
0x00706f0fe8110ea4,     0x002278b42062504e,\
0x002a05a95da52b9c,     0x004dc5d0b9df0e49,\
0x006c3641e06cf0c4,     0x007947a68e374a6e,\
0x00d8d2632b336802,     0x013856c42fb5022e,\
0x003b14db1a083918,     0x00f8673ab12fe1bb,\
0x00a108b014ed1224,     0x00b2892e6aef3b04,\
0x022c3cd3180024d6,     0x023911aad0fc5f34,\
0x020807c9714b4ee6,     0x071a331c09302327,\
0x0418062921a9d245,     0x051f16342b0f9253,\
0x013b19bfda0ae97c,     0x0712172535192f1e,\
0x03322a0bc910d295,     0x00c070fffa196394,\
0x009ab072a84999dd,     0x004e82e174a1aaac,\
0x00aa5205419584b9,     0x00678a2b8c1ee3d5,\
0x0045bcacc9d8e7d0,     0x003a19c384b0d4f9,\
0x008b1960a582c51b,     0x004ec4506c2a2acd,\
0x007473c8605786e1,     0x011316536bbab37f,\
0x000f0d247fdbdeef,     0x00adf608a9de1e76,\
0x009f540868cf82e3,     0x0014f5fc312b801d,\
0x00106f14c43224fe,     0x00b85bad5d8b37c5,\
0x00edce17848c990d,     0x00ccdc13df11bcc4,\
0x020c2d6776a0559b,     0x020a1b64ea7521f7,\
0x00391d24d990be82,     0x007500719dd73ca8,\
0x00ef2eadb2a82a37,     0x008ef10a40961460,\
0x00339a2cfe3f67a5,     0x00e071306365879d,\
0x006e6e3d2a75dcf0,     0x012e7254c18273f0,\
0x004808adcd9b667f,     0x0110801230ea6395,\
0x00fa98c7f1f1fc38,     0x010d400fe6fe7bd9,\
0x00479e150008b94d,     0x00f980cf38134895,\
0x002a0ffe7bcb5dac,     0x00750b4b0fce9b99,\
0x00bd6efe79b37758,     0x07200e2431153628,\
0x007caaf14032bc02,     0x023d011f618395bb,\
0x0053807834fe6f9e,     0x0102303de88670c9,\
0x00f50c3ac708d8f8, 0x00ff5ef8a99d75bc,\
0x02043a12100e517e, 0x00e82d19d049d706,\
0x02142631db624214, 0x00a91363b0ffa60b,\
0x00858ea1137db2f9, 0x006f4ffa06b018ec,\
0x010057e77680f034, 0x005a5ab88b8c7272,\
0x011d196834081d82, 0x006dee0756faeaac,\
0x001661e80c160078, 0x00cca67a1a58c313,\
0x01223683b81b13e8, 0x0065cac20345d68d,\
0x01031c1a6050ecdc, 0x013704ec972c1905,\
0x004ec9e48b6c0b25, 0x0055c40d12a5e1dd,\
0x00f0d4fcc420e40e, 0x009bb8c1998b27f3,\
0x00cac85d04041af6, 0x020541af898eba05,\
0x002cb0263d671095, 0x023e42ffe7ac3643,\
0x0019f3624e096087, 0x26f8d3995aa788ec,\
0x7472747368732e00, 0x65746f6e2e006261,\
0x6975622e756e672e, 0x742e0064692d646c,\
0x7461642e00747865]

def leftRotate(n, d):
    return ((n << d)|(n >> (64 - d))) & 0xFFFFFFFFFFFFFFFF

test_flag = b"l#ctf{##2#M#LY##W#7Ch###4#3##n#5_#r#########0#U###8####u+1#6#a}"
test_flag = b"l#ctf{##2#M#LYf5W#7Ch_##473r#n#5_#r#########0#U#3#8####u+1#6#a}"
test_flag = b"lactf{4223M8LY_5W17Ch_57473Mtn75_4r3_7h3_4850LU########u+1f60a}"
test_flag = b"lactf{4223M8LY_5W17Ch_57473Mtn75_4r3_7h3_4850LU+3_8357#u+1f60a}"
chars = b"+012345678CLMUWY_acfhlnrtu{}"

flag = b"#"*64
flag = flag.split(b'#')
cnt = 0
print(hex(leftRotate(x[0x61],8)))
r = leftRotate(x[0x61], 8)
r1 = leftRotate(r,8)
r2 = leftRotate(r1,8)

test_flag = list(test_flag)
for i in chars:
    rot1 = leftRotate(x[i], 8)
    rot2 = leftRotate(rot1,8)
    rot3 = leftRotate(rot2, 8)
    rot4 = leftRotate(rot3, 8)
    rot5 = leftRotate(rot4, 8)
    rot6 = leftRotate(rot5, 8)
    rot7 = leftRotate(rot6, 8)
    rot8 = leftRotate(rot7, 8)
    rot9 = leftRotate(rot8, 8)
    rot10 = leftRotate(rot9, 8)
    rot11 = leftRotate(rot10, 8)
    rot12 = leftRotate(rot11, 8)
    rot13 = leftRotate(rot12, 8)
    c = rot12 & 0xff
    if c < 0x40: # and (test_flag[c]== ord(b'#')):
        test_flag[c] = i

print('\n')
arr = ''
for i in test_flag:
    if i == b'':
        arr += '#'
    else:
        arr += chr(i)

print(arr)
```
