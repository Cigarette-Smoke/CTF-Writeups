# Binary Instrumentation 1

Author: Venax

Difficulty: Medium

Description:
```
I have been learning to use the Windows API to do cool stuff!
Can you wake up my program to get the flag?
Download the exe here. Unzip the archive with the password picoctf
```

# Analyzing the executable
- I began with launching the executable and analyzing the input/output.

<img width="1477" height="757" alt="image" src="https://github.com/user-attachments/assets/968341dc-c4b9-47ce-8e98-2fd169e0b744" />

# Reverse Engineering

- It took a quick look at the output and the description to know that the source code probably looks something like this:
```C
#include <stdio.h>
#include <windows.h>

int main(void) {
  printf("Hi I have the flag for you right here!\n");
  printf("I'll just take a quick nap before I print it for you, should only take me about a decade or so!\n");
  printf("zzzzzzz....\n");

  /* I assume that this is a Windows API function
  because of this line in the description
  "I have been learning to use the Windows API to do cool stuff!" */
  SOME_KIND_OF_A_WAITING_WINDOWS_API_FUNCTION(int INFINITE);

  printf("picoCTF{flag}");

  return 0;
}

```
# 
- 
