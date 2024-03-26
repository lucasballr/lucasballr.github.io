---
title: "rev/parallel"
date: 2023-01-27t19:46:10-08:00
draft: false
---

This was basically a flag scrambler/checker that would run in 8 different
processes. The binary required there to be 8 processes in order to run correctly
because it would use inter-process communication.

Essentially it would take your input, and jumble it using a key and just swap bytes
in weird places. This was easily reversable. Then it would user openMPI to send your
jumbled input in 8 byte sections to the 8 different processes. Then these processes
went in a loop 10000 times sending a byte at a certain index to the next process. These
processes were essentially rotating the bytes in a circle between processes. So it looked
kinda like this:

```python
proc1 = [a,b,c,d,e]
         v
proc2 = [f,g,h,i,j]
         v
proc2 = [k,l,m,n,o]
...       
proc8 = [w,x,y,z,s]
         v
proc1 = [a,b,c,d,e]
```

At the end of the program, it would check the result against a string, so all we had to do
was create a program that did all this stuff backwards. This is what it looked like (code
assisted by chatgpt):

```c
#include <stdio.h>
#include <string.h>

#define NUM_BINS 8
#define NUM_CHARS 8
#define MAX_KEY_LEN 32

int key[MAX_KEY_LEN] = {0x1a, 0x20, 0xe, 0xb, 3, 1, 0x20, 0x18, 0xd, 0x11, 3, 0x11, 2, 0xd, 0x13, 6, 0xc, 0x16, 3, 0x1e, 10, 6, 8, 0x1a, 6, 0x16, 0xd, 1, 0x13, 1, 1, 0x1d};

char bins[NUM_BINS][NUM_CHARS] = {"m_ERpmfr", "NkekU4_4", "asI_Tra1", "e_4l_c4_", "GCDlryid", "S3{Ptsu9", "i}13Es4V", "73M4_ans"};

int num_workers = NUM_BINS;

int main(void)
{
    char to_arr[8];
    for (int i = 10000; i > -1; i--)
    {
        int offset = i % NUM_CHARS;
        for (int rank = 8; rank > 0; rank--)
        {
            int to_bin = (num_workers + (rank-1 - i) % num_workers) % num_workers;
            //int from_bin = (num_workers + (i + rank) % num_workers) % num_workers;
            to_arr[rank-1] = bins[to_bin][offset];
        }
        for (int i = 0; i < 8; i++){
            bins[i][offset] = to_arr[i];
        }
    }
    char string_thing[NUM_BINS * NUM_CHARS];
    for (int i = 0; i < NUM_BINS; i++)
    {
        strcpy(string_thing + i * NUM_CHARS, bins[i]);
    }

    printf("%s\n", string_thing);
    int string_thing_len = strlen(string_thing);
    for (int i = 31; i >= 0; i--)
    {
        int x = string_thing[i];
        string_thing[i] = string_thing[key[i] + 0x1f];
        string_thing[key[i] + 0x1f] = x;
    }
    printf("%s\n", string_thing);

    return 0;
}
```
