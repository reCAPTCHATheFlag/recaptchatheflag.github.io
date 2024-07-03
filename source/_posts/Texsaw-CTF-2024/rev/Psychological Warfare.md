---
title: Psychological Warfare
author: bytebrew
layout: writeup
category: Texsaw-CTF-2024
chall_description: 
points: 0
solves: 0
tags: rev
date: 2024-03-25
comments: false
---

Did you know that the mov instruction in x86 is Turing complete :D?

---

# The crackme's background
![image](https://github.com/Boberttt/notes/assets/104478197/7ab34977-6344-4412-b1ba-b12bddaccbd3)
 
In the past, I've seen Christopher Domas's work. I could tell from the challenge name it was going to be Christopher Domas related because of this awesome project he made: https://github.com/xoreaxeaxeax/REpsych. But I did not expect it to be movfuscated (pick a better name devs! Edit: after reading the official writeup I realized it was a hint, lol)
 
For those who do not know, x86 mov is Turing complete. This means that, only using the mov instruction, you can remake any program. Christopher's role in this was making a mov compiler: https://github.com/xoreaxeaxeax/movfuscator

# Initial analysis
It only uses mov, and it's compiled for Linux. The flag is the amount of instructions executed + the hidden message. It takes no input (not via stdin, a file, or anything), and has a flat control flow (no comparisons, even when demovuscated).

# The first part of the flag
![image](https://github.com/Boberttt/notes/assets/104478197/d2e11df9-7247-47d0-a2ee-8fe8ba40a7de)
 
Thankfully, my team-mate knew how to use perf. 

# The second part
All info that is given is: it's a hidden message ;-;
### Demov
Demovfuscating was annoying, but doable. Long ago, I tried compiling demovuscator (https://github.com/leetonidas/demovfuscator), but failed. So, this time, I searched for a docker container instead, and found it: https://hub.docker.com/r/iyzyi/demovfuscator
 
Does it have malware? Does it upload my binary to some random website? I don't know, but it works (kinda):
![image](https://github.com/Boberttt/notes/assets/104478197/e2a10fc1-3f07-4ae6-99bf-0145e1b89053)
### Finding the hidden message
After A LOT of trial and error, I finally found out how to obtain the flag:
1. I dumped the decompilation:
![image](https://github.com/Boberttt/notes/assets/104478197/d647a183-b187-4caf-957c-8d57b8f0016a)
2. I ran a diff checker against the sub functions, the only difference is the written address, the function name, and most importantly: the value written (THOSE LOOK LIKE CHAR VALUES!):
![image](https://github.com/Boberttt/notes/assets/104478197/9cc14bfa-0a50-4eb9-90c9-8aa7144d9bbd)
2.5. While debugging, make all exceptions pass to app (movfuscated binaries need exceptions in order to work):
![image](https://github.com/Boberttt/notes/assets/104478197/8ed72b20-3cfd-4236-a859-cd9da86cb3e8)
3. I traced the variable (R0) in IDA (yes I use IDA's debugger, fight me):
![image](https://github.com/Boberttt/notes/assets/104478197/7e1a66d3-9adc-4edf-8b53-ad51f7cdbc9c)
 
 ![image](https://github.com/Boberttt/notes/assets/104478197/c7756711-ec80-43d7-bfe7-a0d205163cb5)
4. Copy all the hex values and use a vim macro to extract the hex values (I'm scared of regex):
 
![image](https://github.com/Boberttt/notes/assets/104478197/b7a7c09c-8da7-4538-8516-79fce018a10b)
5. Use magical cyberchef (not to be confused with cyberchef's magic thing) for hex decoding:
![image](https://github.com/Boberttt/notes/assets/104478197/13bab195-9836-40a4-b689-ee86da90ce3c)
6. Based off of eyeballing, each char is repeated each time except e, and the stuff written at the beginning and the end appear to be junk:
 
![image](https://github.com/Boberttt/notes/assets/104478197/db0a2ea3-86a2-4a26-9ca3-fa73d05954d7)

# All together

	texsaw{387711_miles_to_my_home}
