# BINARY EXPLOITATION CHALLENGE
---
### Name        : Heap Havoc
### Category    : Binary Exploitation
### Difficulity : Hard
---

## THE STEP OF MINE TO SOLVE THIS CHALLENGE
### 1. Examine the code
- First thing that I do to solve this challenge is examine the code.<br>
Examine the type of file by typing:
```bash
file ./vuln
pwn checksec ./vuln
```
from the examination we got the result:

```bash
[*] Arch  : i386-32-little
    RELRO : Partial RELRO
    Stack : No Canary Found
    NX    : NX Enabled
    PIE   : No PIE
```
We know that the file has no much protection, only NX, it means we can't execute the stack. <br>
But seeing from the title of challenge, seems we will not deal with the stack, instead we will deal with the heap. so it's fine, the NX protection will not be our obstacle
- Second thing that I do is trying to understand the logic behind the code
