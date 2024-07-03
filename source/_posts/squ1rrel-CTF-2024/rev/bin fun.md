---
title: bin fun
author: bytebrew
layout: writeup
category: squ1rrel-CTF-2024
chall_description: 
points: 0
solves: 0
tags: rev
date: 2024-4-28
comments: false
---

I lost the description :(

---

# Background 
SquirelCTF challenge, called "bin fun", TL;DR at end (to avoid spoilers).

# Static Analysis 
It's not static but it is stripped, but finding main is not an issue as you can see it pass the arguments in libc_start_main.
 
![image](https://github.com/Boberttt/notes/assets/104478197/6ddc581d-0cb8-4e40-8590-faf575a5c019)
 
In main, we see that it mprotects a region of code twice the length of the page size with the permissions "7". 7 is read/write/execute (read | write | execute = 7).
 
![image](https://github.com/Boberttt/notes/assets/104478197/e6f91f6c-2e89-464a-b12d-d29680f91de4) 
If the above doesn't fail to execute, a mystery sub is executed, with argv\[1\] (the first argument of the program) as the first argument of the mystery sub. 
 
![image](https://github.com/Boberttt/notes/assets/104478197/1201892b-3af0-4d7a-b013-794426d44d8b)
 
In the mystery sub, the code appears to be xoring what will be executed after a loop exits with 0x91 (it unpacks the next function). 
![image](https://github.com/Boberttt/notes/assets/104478197/36e70aa8-eff1-43ff-97b6-bdced49693ef)
 
Armed with this information, we can start using dynamic analysis
# Dynamic Analysis
First we set a breakpoint before the unpack loop, run the program, hit the breakpoint, continue the code, wait for it to crash, and hit the back button (leading us back to the breakpoint and the unpacked code).
 
The most important part of the unpacked code is:
```asm
.text:000000000040123B loc_40123B:                             ; DATA XREF: sub_401210:loc_401222↑o
.text:000000000040123B mov     al, [rdi]
.text:000000000040123D
```
Remember that the first argument passed to this function was a pointer to our first argument (which is stored in rdi), but since we debugged this code without passing any arguments it is trying to grab data from a null pointer, leading to a crash. After this piece of code we see another unpacker function:
```asm
.text:0000000000401247 loc_401247:                             ; CODE XREF: .text:0000000000401254↓j
.text:0000000000401247 mov     bl, [rcx]
.text:0000000000401249 xor     bl, 0B4h
.text:0000000000401249 ; END OF FUNCTION CHUNK FOR sub_401210
.text:000000000040124C mov     [rcx], bl
.text:000000000040124E inc     rcx
.text:0000000000401251 cmp     rcx, rsi
.text:0000000000401254 jb      short loc_401247
```
So, let's pass a random argument to the program and perform the first step (set a breakpoint, run the program, hit the breakpoint, continue the code, wait for it to crash, and hit the back button (leading us back to the breakpoint and the unpacked code)). NOTE: Code isn't being displayed? Undefine existing stuff, then make code (spam u at bytes and press c when you're done). Here's the most important part of the unpacked code:
```asm
.text:0000000000401256 loc_401256:                             ; DATA XREF: sub_401210+2D↑o
.text:0000000000401256                 xor     al, 0E7h        ; ACTUALLY IMPORTANT 
.text:0000000000401258                 lea     rcx, loc_401271
.text:000000000040125F                 mov     rdx, rsi
.text:0000000000401262
.text:0000000000401262 loc_401262:                             ; CODE XREF: .text:000000000040126F↓j
.text:0000000000401262                 mov     bl, [rcx]
.text:0000000000401264                 xor     bl, 82h
.text:0000000000401267                 mov     [rcx], bl
.text:0000000000401269                 inc     rcx
.text:000000000040126C                 cmp     rcx, rsi
.text:000000000040126F                 jb      short loc_401262
.text:0000000000401271
.text:0000000000401271 loc_401271:                             ; DATA XREF: .text:0000000000401258↑o
.text:0000000000401271                 cmp     al, 94h         ; ACTUALLY IMPORTANT 
.text:0000000000401273                 jnz     short loc_40121E
```
Basically, this code xors our first byte with 0xe7 and then compares it with 0x94, if they are equal, the code continues, if they aren't, we return (and crash). This means that our first byte is 0xe7 ^ 0x94, which is 0x73, aka s, the first letter of the flag format. So if we set the first letter of our arg to s, we will pass this check. If you repeat the above steps (set a HARDWARE breakpoint (software actually modifies the code in memory, so when it gets xored it breaks everything) after the next unpack loop, run it, hit the breakpoint, continue, crash, hit the back button for the disassembly putting you back at the breakpoint, scroll down, xor the two values, append it to your first arg) you'll find it repeats the pattern, so you can use this to find the whole flag.
# The solve

    squ1rrel{n1ce_r3v_sk3ll5_34289}
