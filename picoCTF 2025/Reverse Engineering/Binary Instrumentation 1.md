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

- I began with launching the executable and analyzing the I/O.

<img width="1477" height="757" alt="image" src="https://github.com/user-attachments/assets/968341dc-c4b9-47ce-8e98-2fd169e0b744" />

# Visual Analysis

- I theorized that the source code looks probably something like this based on the implications inside of program's output and the challenge's description:
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

# Binary Analysis

- I used Frida, a dynamic instrumentation toolkit to solve this challenge, you can install it here: https://frida.re/docs/installation/
- I read this website to learn Frida from scratch: https://learnfrida.info/basic_usage/#dealing-with-strings-reading-and-allocation

- When I started going through the documentation, I stumbled upon the ```frida-trace``` tool, which allows us to monitor API calls of an existing process, so I immediately hooked up the tracing tool to my running program.
  
<img width="1468" height="309" alt="showcase1" src="https://github.com/user-attachments/assets/4cc2463f-cf7b-4d50-bb7e-392dc93556d3" />

- ```-i "Sleep"``` - only trace API calls named "Sleep" regardless of what module they are coming from 

- ```3308``` - process id

- Upon opening the tool's Web UI I realized it wasn't catching any calls because I auto-generated handlers after the sleep function was called.

<img width="1919" height="977" alt="image" src="https://github.com/user-attachments/assets/3dd50b00-2f4a-4eca-8254-616f1367a434" />

- The solution that I came up with is as follows:
- According to learnfrida.info ```frida CLI (Command Line Interface)``` is capable of creating new processes and immediately pausing execution of their main threads. In other words, we can launch a program (so that the Windows OS registers it as a running process), pause the execution of all threads and auto generate a handler for the "Sleep" call before any of the code inside of ```int main()``` runs.

## 1. I launched the executable through the ```frida CLI``` tool with a ```--pause``` argument at the end. 

<img width="1468" height="750" alt="showcase2" src="https://github.com/user-attachments/assets/e49b4775-2a47-49fe-8682-3975a44a9019" />

## 2. In a separate terminal, connected the ```frida-trace``` tool to the running process

<img width="957" height="352" alt="image" src="https://github.com/user-attachments/assets/52c83604-8216-471b-bd4b-1487db0a6991" />

## 3. %resume'd the process and monitored event logs

<img width="1611" height="907" alt="image" src="https://github.com/user-attachments/assets/ca9a5a33-cfac-405f-8a9a-0a69f8f604b2" />
- The results above proved that the program does indeed use sleep. I was now ready to start the instrumentation process.

# Binary Instrumentation
### Weaponization
- When I initially started reading the learnfrida.info 
I discovered that the toolkit is capable of injecting new code  
and modifying existing code of running processes without changing the overall functionality of programs.
- I used this knowledge, as well as this documentation (https://frida.re/docs/javascript-api/), to develop the following function interceptor:
```Javascript
const sleepPointer = Process.findModuleByName("KERNEL32.DLL").getExportByName("Sleep"); // we need the function's adress in order to attach an interceptor to it

Interceptor.attach(sleepPointer, { // the interceptor executes the following code whenever the API call is made within the program. FIY, this executes BEFORE the original function's code
    onEnter(args) {
        console.log("Sleep()", args[0]);
        args[0] = new NativePointer('1'); // This part changes the "DWORD dwMilliseconds" argument before execution, therefore changing the program's sleep duration from 10 years to 1 second.
        console.log("Sleep()", args[0]);
     }
});
```
### Injection
<img width="1470" height="757" alt="image" src="https://github.com/user-attachments/assets/08a067a9-5aa2-43bb-a524-7268dd5dadb6" />

<img width="1468" height="749" alt="image" src="https://github.com/user-attachments/assets/b7b00c92-9f5b-4d90-92c6-5fe6b835b98b" />

