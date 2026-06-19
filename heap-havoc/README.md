# BINARY EXPLOITATION CHALLENGE
---
### Name        : Heap Havoc
### Category    : Binary Exploitation
### Difficulity : Hard
---

## THE STEP OF MINE TO SOLVE THIS CHALLENGE
### 1. Examine the code
First thing that I do to solve this challenge is examine the code.<br>
Examine the type of file by typing:
```bash
file ./vuln
pwn checksec ./vuln
```
from the examination we got the result:

```bash
[*] Arch  : i386-32-little
    RELRO : Partial RELRO<br>
    Stack : No Canary Found<br>
    NX    : NX Enabled<br>
    PIE   : No PIE<br>
