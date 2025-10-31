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

#### Note: Windows OS only

## I/O Analysis

- I began with analyzing the I/O of the given program.

<img width="1477" height="757" alt="image" src="https://github.com/user-attachments/assets/968341dc-c4b9-47ce-8e98-2fd169e0b744" />

- Based on the implications in the program's output and the challenge's description, I came to a conclusion that the source code probably looks something like this:

```C
#include <stdio.h>
#include <windows.h>

int main(void) {

  printf("Hi I have the flag for you right here!\n");
  printf("I'll just take a quick nap before I print it for you, should only take me about a decade or so!\n");
  printf("zzzzzzz....\n");

  /* I assume that this is a Windows API function because of this line in the description:
  "I have been learning to use the Windows API to do cool stuff!" */
  SOME_KIND_OF_A_WAITING_WINDOWS_API_FUNCTION(int INFINITE);

  printf("picoCTF{flag}");

  return 0;
}
```

- I decided to go through the [Win32 API](https://learn.microsoft.com/en-us/windows/win32/api/_base/) to figure out which function makes our program "take a quick nap". The [Sleep()](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep) function seemed to be the closest guess.

- Before proceeding any further, I went through the [binary instrumentation basics](https://www.seclab.cs.sunysb.edu/seclab/pubs/soumyakant_rpe.pdf).

## Trying to Hook Sleep()

- After learning the basics, I installed a dynamic instrumentation toolkit. For this challenge I used [Frida](https://frida.re/docs/installation/).

- Additionally, I have learned how to use the toolkit with the help of [this guide](https://learnfrida.info/basic_usage/#dealing-with-strings-reading-and-allocation).

- Knowing that this program uses Windows API, I attached a function tracing tool to our running program.

<img width="1468" height="309" alt="showcase1" src="https://github.com/user-attachments/assets/4cc2463f-cf7b-4d50-bb7e-392dc93556d3" />

- ```frida-trace``` tool auto-generates handlers and allows us to monitor API calls.

- ```-i "Sleep"``` - only trace API calls named "Sleep" regardless of what module they are coming from 

- ```3308``` - process ID

- My tracing tool wasn't catching any calls because I generated handlers after Sleep() was called:

<img width="1801" height="842" alt="image" src="https://github.com/user-attachments/assets/e5bde45a-200e-4f16-842b-e5026fdb1ff2" />

- After reading some more, I found that ```frida CLI (Command Line Interface)``` can create a new process and immediately pause the execution of its main thread. In other words, I could now launch a program (so that the Windows OS registers it as a running process), immediately pause its execution, and auto-generate a handler for the "Sleep" call before any of the ```int main()``` code runs.

## Successfully Hooking and Monitoring Sleep()

#### 1. Launched the executable through the ```frida CLI``` tool with a ```--pause``` argument at the end. 

<img width="1468" height="750" alt="showcase2" src="https://github.com/user-attachments/assets/e49b4775-2a47-49fe-8682-3975a44a9019" />

#### 2. In a separate terminal, connected the ```frida-trace``` tool to the running process

<img width="957" height="352" alt="image" src="https://github.com/user-attachments/assets/52c83604-8216-471b-bd4b-1487db0a6991" />

#### 3. %resume'd the process and monitored event logs

<img width="1768" height="917" alt="image" src="https://github.com/user-attachments/assets/026984c8-432f-42fc-833e-c6c9a85e06bd" />

- Now that we have made sure that Sleep() is used by the program, we can finally get to the fun part.
- You see, the toolkit can be not only used to monitor functions but also to intercept and modify their values right before execution.

## Creating an Interceptor Script

- With the help of this [documentation](https://frida.re/docs/javascript-api/), I quickly made a Sleep() function interceptor script:

```Javascript
const sleepPointer = Process.findModuleByName("KERNEL32.DLL").getExportByName("Sleep"); // we need the function's adress in order to attach an interceptor to it

Interceptor.attach(sleepPointer, {

    onEnter(args) { // interceptor executes this code whenever Sleep() is called. FYI, this executes BEFORE the original function's code

        console.log("Sleep()", args[0]);

        args[0] = new NativePointer('1'); // This part changes the "DWORD dwMilliseconds" argument before execution, therefore changing the program's sleep duration from 10 years to 1 second.

        console.log("Sleep()", args[0]);

     }
});
```

## Interceptor Script Injection
- ```frida -f bininst1.exe -l interceptor.js``` creates a new process of the file and injects our code before the program executes its first line of code
<img width="1470" height="757" alt="image" src="https://github.com/user-attachments/assets/08a067a9-5aa2-43bb-a524-7268dd5dadb6" />

<img width="1468" height="749" alt="image" src="https://github.com/user-attachments/assets/b7b00c92-9f5b-4d90-92c6-5fe6b835b98b" />

