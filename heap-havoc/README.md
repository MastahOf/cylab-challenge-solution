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
This is the code, after I examine it for a while, I know where is the vulnerable. Let us just focus on the 'malloc` section, when the struct was initiated as `i1, i2, and i3`.<br>
first one, the struct with label __i1__ it was initiated first into the heap. But inside the struct __i1__, __i1->name__ was also initiated to heap, so it would make new area of heap for __i1->name__ in the amount of 8 bytes.<br>
second one, same, the strut
### Heap Memory
-> heap for the i1 with size 20 bytes
   -> metadata (prev_size = 4 byte, size + AMP flag = 4 byte) = 8 byte
   -> i1 priority = 4 byte (type data is `int` that's why 4 byte)
   -> i1 name = 4 byte (because the name was also initiated into heap, 4 bytes for the address of name heap)
   -> i1 callback = 4 byte (because this was a function pointer, so callback hold an address)
