# [Binary Instrumentation 1](https://play.picoctf.org/practice/challenge/451?category=3&originalEvent=74&page=1)

Author: Venax

Difficulty: $$\color{orange}{Medium}$$

Description:
```
I have been learning to use the Windows API to do cool stuff!
Can you wake up my program to get the flag?
Download the exe here. Unzip the archive with the password picoctf
```

# Understanding the Task

- The name of the challenge "[Binary Instrumentation](https://www.seclab.cs.sunysb.edu/seclab/pubs/soumyakant_rpe.pdf)",
  implies a process of injecting new code into a program or modifying its existing code, either at its runtime or offline, **without altering the application's overall behaviour**.
  The main goal of the binary instrumentation process is to analyze, monitor or enhance the program's behaviour **without changing its intended logic or outcome**.

  It is important to emphasize the ending phrase"**without changing its intended logic or outcome**" because this part of the definition differentiates instrumentation from [patching](https://en.wikipedia.org/wiki/Patch_(computing)) or [cracking](https://en.wikipedia.org/wiki/Software_cracking).

# Analyzing the executable

- We now need a basic understanding of what the program does. 
  Launching the executable will reveal a console application that doesnt accept any inputs from the user and only prints the following 3 lines of output:

<img width="1477" height="757" alt="image" src="https://github.com/user-attachments/assets/968341dc-c4b9-47ce-8e98-2fd169e0b744" />

- If we look at the description of the challenge, the program needs to "wake up" before it can print our flag and we can also see that it uses the windows API
- should be extracted using a binary instrumentation toolkit. For this challenge, I will be using a dynamic instrumentation toolkit Frida
- Lets try to get some clues to the approximate technical processes on the backend.
- The logical question now is "What part of the program's behaviour do we have to change to get the flag?".
- To know which part of the program's behaviour we have to change ==> We need some basic understanding of the program's backend.
