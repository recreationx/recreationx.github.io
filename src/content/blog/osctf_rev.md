---
title: OSCTF 2024 rev challenges writeup
author: recreation
pubDatetime: 2024-07-14T12:00:00+08:00
slug: osctf-2024-re
featured: true
draft: false
tags:
  - re
  - ctf
ogImage: ""
description: A writeup for the re challenges in OSCTF 2024
---

## Table of Contents

## Gophers Language

### Challenge Description

I know go is not a popular language, so I decided of creating a reversing challenge out of it. I'm sure now go will overtake java!!

Author: @5h1kh4r

`main.exe` is provided

### Solution

Challenge is written in Go.

```sh
└─$ strings main.exe| grep "OSCTF"
...
OSCTF{Why_G0_S0_H4rd}
(truncated)
```

We have our flag.

## Another Python Game

### Challenge Description

You know, I love Pygame why don't you. Prove your love for Pygame by solving this challenge Note: It is necessary to keep the background.png file in the same place as the exe file so that the exe file runs properly

Author: @5h1kh4r

`source.exe` and `background.png` is provided.

### Solution

The exe file looks like a Python compiled binary. We run (pyinstxtractor)[https://github.com/extremecoders-re/pyinstxtractor] to decompile the file. We get `source.pyc`.

```sh
└─$ strings source.pyc | grep OSCTF
Tz"OSCTF{1_5W3ar_I_D1dn'7_BruT3f0rc3}
```

We have our flag.

## Avengers Assemble

### Challenge Description

The Avengers have assembled but for what? To solve this!? Why call Avengers for such a simple thing, when you can solve it yourself

FLAG FORMAT: `OSCTF{Inp1_Inp2_Inp3} (Integer values)`

Author: @Inv1s1bl3

```asm title="code.asm"
asm
extern printf
extern scanf

section .data
        fmt: db "%ld",0
        output: db "Correct",10,0
        out: db "Not Correct",10,0
        inp1: db "Input 1st number:",0
        inp2: db "Input 2nd number:",0
        inp3: db "Input 3rd number:",0

section .text
        global main

        main:
        push ebp
        mov ebp,esp
        sub esp,0x20

        push inp1
        call printf
        lea eax,[ebp-0x4]
        push eax
        push fmt
        call scanf

        push inp2
        call printf
        lea eax,[ebp-0xc]
        push eax
        push fmt
        call scanf

        push inp3
        call printf
        lea eax,[ebp-0x14]
        push eax
        push fmt
        call scanf

        mov ebx, DWORD[ebp-0xc]
        add ebx, DWORD[ebp-0x4]
        cmp ebx,0xdeadbeef
        jne N

        cmp DWORD[ebp-0x4], 0x6f56df65
        jg N

        cmp DWORD[ebp-0xc], 0x6f56df8d
        jg N
        cmp DWORD[ebp-0xc], 0x6f56df8d
        jl N

        mov ecx, DWORD[ebp-0x14]
        mov ebx, DWORD[ebp-0xc]
        xor ecx, ebx
        cmp ecx, 2103609845
        jne N
        jmp O

        N:
        push out
        call printf
        leave
        ret

        O:
        push output
        call printf

        leave
        ret
```

### Solution

We have to input 3 numbers to pass the conditional checks. The program asks for 3 inputs at the start.

```asm
        mov ebx, DWORD[ebp-0xc]
        add ebx, DWORD[ebp-0x4]
        cmp ebx,0xdeadbeef
        jne N
```

This moves `DWORD[ebp-0xc]` (second input) to `ebx` and adds the first input to `ebx`. The result should be `0xdeadbeef`

```asm
        cmp DWORD[ebp-0x4], 0x6f56df65
        jg N

        cmp DWORD[ebp-0xc], 0x6f56df8d
        jg N
        cmp DWORD[ebp-0xc], 0x6f56df8d
        jl N
```

Our first input should be less than 0x6f56df65, and the second input should be exactly 0x6f56df8d (jg and jl would jump to the label N if true). So our first input is thus `0xdeadbeef - 0x6f56df8d = 1867964258`

```asm
        mov ecx, DWORD[ebp-0x14]
        mov ebx, DWORD[ebp-0xc]
        xor ecx, ebx
        cmp ecx, 2103609845
        jne N
        jmp O
```

The second input is XOR'ed with the third input to get 2103609845. To get the third input, we xor the known second input with 2103609845. So `0x6f56df8d ^ 2103609845 = 305419896`

The flag is thus `OSCTF{1867964258_1867964301_305419896}`.

## The Broken Sword

### Challenge Description

The time for the reforging of the The Sword That Was Broken has come.. Elendil left a riddle, solving which will give the password, which is what you need to find :p..

Flag Format: `OSCTF{valueof'flag'variable_valueof'a'variable_valueof'v2'variable} / OSCTF{flag_a_v2}`

Author: @Inv1s1bl3

```py title="challenge.py"
from Crypto.Util.number import *
from secret import flag,a,v2,pi

z1 = a+flag
y = long_to_bytes(z1)
print("The message is",y)
s = ''
s += chr(ord('a')+23)
v = ord(s)
f = 5483762481^v
g = f*35


r = 14
l = g
surface_area= pi*r*l
w = surface_area//1
s = int(f)
v = s^34
for i in range(1,10,1):
    h = v2*30
    h ^= 34
    h *= pi
    h /= 4567234567342
a += g+v2+f
a *= 67
al=a
print("a1:",al)
print('h:',h)
```

```title="message.txt"
The message is b'\x0c\x07\x9e\x8e/\xc2'
a1 is: 899433952965498
h is: 0.0028203971921452278
```

### Solution

The for loop in the challenge is negligible as `v2` remains constant every iteration, thus the outcome of `h` is the same. We write as solve script to work backwards using simple math to obtain the original values. `pi` is taken to be 3.14 such that `h` remains a whole number.

```py title="solve.py"
h = 0.0028203971921452278

h *= 4567234567342
print(h)
h /= 3.14
h = int(h)
h ^= 34
h /= 30

print("v2:", h)

y = int.from_bytes(b"\x0c\x07\x9e\x8e/\xc2", "big")
print("y:", y)
s = chr(ord('a') + 23)
v = ord(s)
f = 5483762481^v
g = f*35

a1 = 899433952965498
a1 /= 67
a = a1 - (g+h+f)
print(a)
flag = y - a
print(flag)
```

The flag is thus `OSCTF{29260723_13226835162127_136745387}`.
