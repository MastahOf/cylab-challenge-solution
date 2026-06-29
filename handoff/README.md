# BINARY EXPLOITATION CHALLENGE
---
### Name: Handoff
### Category: Binary Exploitation
### Difficulity: Hard
---
### 1. Examine the ELF file
As usual, I use `pwntools` to check the metadata of file. <br>
<br>
<img width="591" height="187" alt="image" src="https://github.com/user-attachments/assets/d7b3b02d-10d5-4066-bbda-043c2449e3f7" /><br>
<br>
From the image above we can see that there's no protection at all. so it will make our life easier
### 2. Review the code
After we know the metadata of file, it's a right time to review the code to check where is the vulnerability. <br>
<br>
C```

```
<br>
