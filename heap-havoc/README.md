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
[<kbd>$\color{blue}{\text{*}}$<kbd>] Arch  : i386-32-little
    RELRO : $\color{yellow}{\text{Partial RELRO}}$
    Stack : $\color{red}{\text{No Canary Found}}$
    NX    : $\color{green}{\text{NX Enabled}}$
    PIE   : $\coloe{red}{\text{No PIE}}$
