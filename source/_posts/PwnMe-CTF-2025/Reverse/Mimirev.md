---
title: Mimirev
author: bytebrew
layout: writeup
category: PwnMe-CTF-Quals-2024
chall_description:
points: 401
solves: 48
tags: Hard Compiler Crypto
date: 2025-03-02
comments: false
---

A new and obscure programming language, MimiLang, has surfaced. It runs on a peculiar interpreter, but something about it feels… off. Dive into its inner workings and figure out what's really going on. Maybe you'll uncover something unexpected.

---

# Triage
The binary is stripped, but golang, and very large (because: golang). Since it's golang we can just recover most of the information using a golang plugin (I use [this one](https://github.com/scmerrill/golang_1_18_restore_names)).
# Initial Analysis
Upon restoring symbols we can just go to the main_main function and see some strings such as:
```
f
Alias of -file
disassemble
Disassemble instead of executing
the specified file doesn\'t have the .mimi extension
```
Mostly, these are program arguments, if we follow control flow we see the following functions (the github repo is not public as of doing the challenge):
```go
github_com_Lexterl33t_mimicompiler_compiler_Lifter_Compile
github_com_Lexterl33t_mimicompiler_vm_NewVM
github_com_Lexterl33t_mimicompiler_compiler_Disassembler
github_com_Lexterl33t_mimicompiler_vm_VM_Run
```
This indicates that the .mimi file talked about in the string contains code that gets compiled to bytecode and interpreted by a VM. 
# The VM
Inside of "github_com_Lexterl33t_mimicompiler_vm_VM_Run" we see a large switch statement, within we see functions such as:
```go
github_com_Lexterl33t_mimicompiler_vm_VM_Push
github_com_Lexterl33t_mimicompiler_vm_VM_Ssload
github_com_Lexterl33t_mimicompiler_vm_VM_Sstore
github_com_Lexterl33t_mimicompiler_vm_VM_Add
github_com_Lexterl33t_mimicompiler_vm_VM_Sub
```
The most important being `github_com_Lexterl33t_mimicompiler_vm_VM_VerifyProof`, since it clearly isn't a standard instruction and its purpose will become pretty obvious when we take a look at it later.
# Calling VerifyProof
Using the custom language, how do we call VerifyProof? We can see in `Lifter_Compile` (prefix (github_com...) will be omitted from now on) that there is a function called `Parser_Parse`, within that we see `Parser_parse`, within *that* one we see `Parser_GetStatement`, and within that function we see `compiler_Parser_VerifyProofStatement`. In that function we see the string "verifyProof" so it can be assumed (and confirmed through testing) that we can call it like a standard function: `verifyProof()` in the .mimi file.
# VM_VerifyProof
In this function our suspicions are confirmed when we see the string `Error decrypting flag:`. The only check that appears to be occurring is:
```c
if (r8 + rdx_4 != 0x4cb2f) {
	void* rax_20 = runtime_newobject(&data_4cd3c0, arg3);
	*(rax_20 + 8) = 0x12;
	*rax_20 = "VerifyProof failed";
	return &data_50c700;
}

int64_t rdx_8 = rdx_4 * rdx_4 * rdx_4 + r8 * r8 - rdx_4 * r8;
if (rdx_8 % 0xffffd != 0x42b6e)
	return fmt_Errorf(0, 0, rdx_8 / 0xffffd * 0xffffd, nullptr, 
        "VerifyProof failed", 0x12, arg3);
```
If we are to use reverse slicing, we first need to find values that satisfy r8 and rdx_4. We can do this with a C script:
```c
#include <stdio.h>
int main () {
	for (long long rdx = 0; rdx <= 1000000; rdx++) {
	   for (long long r8 = 0; r8 <= 1000000; r8++) {
			long long temp = (rdx * rdx * rdx + r8 * r8 - rdx * r8);
			if (0x42b6e == (temp%0xffffd)) {
			    if ((r8 + rdx) == 0x4cb2f) {
			        printf("%llu %llu\n", rdx, r8);
				}
			}
		}
	}
}
```
This is very slow (takes 17 minutes, 57.836 seconds to run), here's an optimized version:
```c
#include <stdio.h>
int main () {
	for (long long rdx = 0; rdx <= 1000000; rdx++) {
        // (r8 + rdx) == 0x4cb2f
        // r8 = 0x4cb2f - rdx
        // wow it's fast now
		long long temp = (rdx * rdx * rdx + (0x4cb2f-rdx) * (0x4cb2f-rdx) - rdx * (0x4cb2f-rdx));
		if (0x42b6e == (temp%0xffffd)) {
		    long long r8 = (0x4cb2f-rdx);
		    printf("%llu %llu\n", rdx, r8);
		}
	}
}

```
And now we have the output (took 5 milliseconds):
```
107447 206712
190703 123456
750421 18446744073709115354
```
Going back to our VerifyProof function we can see the following (note that only relevant lines are included):
```c
int64_t* var_10 = &var_80;
github_com_Lexterl33t_mimicompiler_vm_Stack_Pop(1, rsi, rdx, 1, rdx, &var_10, arg3);
var_10 = &var_78;
github_com_Lexterl33t_mimicompiler_vm_Stack_Pop(1, rsi_1, arg2, 1, *arg2, &var_10, arg3);
int64_t rdx_4 = var_80;
int64_t r8 = var_78;
```
Following this the above check function occurs. So, based off context, we can guess function arguments are placed on the stack and pop'd off and checked. 
# Solve
All we have to do is place the following in a .mimi file:
```c
verifyProof(206712, 107447);
```
And run it with:
```
./mimicompiler -f source.mimi
```
Oops, wrong one I guess!
```c I like colors
VerifyProof passed successfully. Flag unlocked!
2025/03/02 01:28:08 Execution error: Error decrypting flag: failed to unpad plaintext: invalid padding
```
Let's try our other value:
```c
verifyProof(123456, 190703);
```
And...
```c
h@ada470e43019 /m/share> ./mimicompiler -f hi.mimi
[{123456} {190703}]
VerifyProof passed successfully. Flag unlocked!
Flag: PWNME{R3v3rS1ng_Compil0_C4n_B3_good}
```
