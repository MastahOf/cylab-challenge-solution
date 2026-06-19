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


[$\color{blue}{\text{*}}$] Arch  : i386-32-little<br>
    RELRO : $\color{yellow}{\text{Partial RELRO}}$<br>
    Stack : $\color{red}{\text{No Canary Found}}$<br>
    NX    : $\color{green}{\text{NX Enabled}}$<br>
    PIE   : $\coloe{red}{\text{No PIE}}$<br>
