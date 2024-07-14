---
title: BTCTF 2024 snakebyte rev challenge writeup
author: znan
pubDatetime: 2024-05-28T10:33:28+08:00
slug: btctf-2024-snakebyte
featured: true
draft: false
tags:
  - re
  - ctf
  - python
ogImage: ""
description: A writeup for the Python bytecode re chall in BTCTF 2024
canonicalURL: https://example.org/my-article-was-already-posted-here
---

I joined a random team for this CTF. We came in 3rd place.

## Solution

We were provided with the below python bytecode and were required to find the flag. I used the documentation for the Python library [dis](https://docs.python.org/3/library/dis.html) as reference for the bytecodes.

I noticed that there are a few constants defined by the bytecode.

```python title="snakebyte.pyc" wrap
  5           0 BUILD_LIST               0
              2 LOAD_CONST               1 (('_', '3', 'N', 'D', 'J', 'b', 'M', '3', 'n', 'D', 'f', 'S', '}'))
              4 LIST_EXTEND              1
              6 STORE_FAST               0 (chars)

  6           8 LOAD_CONST               2 ('btctf{py7h0nS_')
             10 STORE_FAST               1 (flag)

  7          12 LOAD_CONST               3 (']{1Jv]2f]G5{@]vL2f')
             14 STORE_FAST               2 (encrypted_part)

  8          16 LOAD_CONST               4 (2)
             18 STORE_FAST               3 (key)

  9          20 LOAD_GLOBAL              0 (len)
             22 LOAD_FAST                2 (encrypted_part)
             24 CALL_FUNCTION            1
             26 LOAD_CONST               5 (1)
             28 BINARY_SUBTRACT
             30 STORE_FAST               4 (i)
```

This is then easily converted into Python:

```python
chars = ['_', '3', 'N', 'D', 'J', 'b', 'M', '3', 'n', 'D', 'f', 'S', '}']
flag = 'btctf{py7h0nS_'
encrypted_part = ']{1Jv]2f]G5{@]vL2f'
key = 2
i = len(encrypted_part) - 1
```

I then examine the following code. `POP_JUMP_IF_FALSE` and `POP_JUMP_IF_TRUE` are used to control the flow of execution, thus indicating that there is a presence of a loop. Specifically, if `i` is not greater than `0`, the program will jump to offset `80`, which is the end of loop, and jumps back to the offset `40` if `i` remains greater than `0`.

- Any `LOAD_FAST` used essentially loads the variable onto the stack.
- `LOAD_GLOBAL`: The body of the loop loads the `chr` function and the `ord` onto the stack.
- `BINARY_SUBSCR`: pops the loaded `i` and `encrypted_part` from the stack, accesses `encrypted_part[i]`, and pushes the result onto the stack.
- `CALL_FUNCTION`: calls `ord` with the top stack element (the character from `encrypted_part[i]`) and pushes the result (an integer) onto the stack.
- `BINARY_XOR`: A XOR operation is then performed , and `chr` is then called on this result, and the resulting character is pushed onto the stack.

- `INPLACE_ADD` then pops the two stack elements (`flag` and result of `chr`), appends the character to `flag`, and pushes the updated `flag` back onto the stack.
- `STORE_FAST` Stores the updated `flag` from the stack back into the local variable `flag`.

The process is then repeated until `i > 0` is false, which then breaks out of the loop.

```python
 10          32 LOAD_FAST                4 (i)
             34 LOAD_CONST               6 (0)
             36 COMPARE_OP               4 (>)
             38 POP_JUMP_IF_FALSE       40 (to 80)

 11     >>   40 LOAD_FAST                1 (flag)
             42 LOAD_GLOBAL              1 (chr)
             44 LOAD_GLOBAL              2 (ord)
             46 LOAD_FAST                2 (encrypted_part)
             48 LOAD_FAST                4 (i)
             50 BINARY_SUBSCR
             52 CALL_FUNCTION            1
             54 LOAD_FAST                3 (key)
             56 BINARY_XOR
             58 CALL_FUNCTION            1
             60 INPLACE_ADD
             62 STORE_FAST               1 (flag)

 12          64 LOAD_FAST                4 (i)
             66 LOAD_CONST               5 (1)
             68 BINARY_SUBTRACT
             70 STORE_FAST               4 (i)

 10          72 LOAD_FAST                4 (i)
             74 LOAD_CONST               6 (0)
             76 COMPARE_OP               4 (>)
             78 POP_JUMP_IF_TRUE        20 (to 40)

 13     >>   80 LOAD_FAST                1 (flag)
             82 LOAD_FAST                0 (chars)
             84 LOAD_CONST               6 (0)
             86 BINARY_SUBSCR
             88 INPLACE_ADD
             90 STORE_FAST               1 (flag)
```

The rest of the bytecode is then repetitive - the program indexes a character from the list `chars`, and append them to the flag.

```python
 14          92 LOAD_FAST                1 (flag)
             94 LOAD_FAST                0 (chars)
             96 LOAD_CONST               7 (3)
             98 BINARY_SUBSCR
            100 INPLACE_ADD
            102 STORE_FAST               1 (flag)

 15         104 LOAD_FAST                1 (flag)
            106 LOAD_FAST                0 (chars)
            108 LOAD_CONST               8 (8)
            110 BINARY_SUBSCR
            112 INPLACE_ADD
            114 STORE_FAST               1 (flag)
        ...
```

With this information, I was then able to recreate the original code:

```python
chars = ['_', '3', 'N', 'D', 'J', 'b', 'M', '3', 'n', 'D', 'f', 'S', '}']
flag = 'btctf{py7h0nS_'
encrypted_part = ']{1Jv]2f]G5{@]vL2f'
key = 2

i = len(encrypted_part) - 1
while i > 0:
    flag += chr(ord(encrypted_part[i]) ^ key)
    i -= 1

flag += chars[0]
flag += chars[3]
flag += chars[8]
flag += chars[7]
flag += chars[10]
flag += chars[6]
flag += chars[2]
flag += chars[9]
flag += chars[4]
flag += chars[1]
flag += chars[5]
flag += chars[11]
flag += chars[12]

print(flag)
```

The flag is thus `btctf{py7h0nS_d0Nt_By7E_d0_tH3y_Dn3fMNDJ3bS}`.
