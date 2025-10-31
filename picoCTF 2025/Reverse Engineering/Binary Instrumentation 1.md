Link: https://play.picoctf.org/practice/challenge/451?category=3&page=1
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

I decided to go through the Win32 API to find what function could match the description: https://learn.microsoft.com/en-us/windows/win32/api/_base/
And found the Sleep function: https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep

# Binary Instrumentation
- I used Frida, a dynamic instrumentation toolkit to solve this challenge, you can install it here: https://frida.re/docs/installation/
- I read this website to learn Frida from scratch: https://learnfrida.info/basic_usage/#dealing-with-strings-reading-and-allocation

- The first thing I learned is that the ```frida-trace``` tool allows to monitor API calls of a running process, so I immediately hooked up the tracing tool to my running program.
  
<img width="1468" height="309" alt="showcase1" src="https://github.com/user-attachments/assets/4cc2463f-cf7b-4d50-bb7e-392dc93556d3" />

```-i "Sleep"``` - only trace API calls named "Sleep" regardless of what module they are coming from 

```3308``` - process id

- When I opened the web UI I realized that the tool wasn't catching any calls because I auto-generated handlers AFTER the sleep function blocked execution of threads.
<img width="1919" height="977" alt="image" src="https://github.com/user-attachments/assets/3dd50b00-2f4a-4eca-8254-616f1367a434" />

- The solution that I came up with is as follows: ```frida CLI``` can create a new process and immediately pause execution of the main thread. In other words, we can launch a program (so that the Windows OS registers it as a running process) and pause it before any of the code inside of ```int main()``` runs. 
