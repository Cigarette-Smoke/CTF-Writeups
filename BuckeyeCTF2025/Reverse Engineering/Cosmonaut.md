Link: https://pwnoh.io/challenges
# Cosmonaut
Author: mbund
```
This is a good rev challenge to start with. Use the help button to ask for a hint if you get stuck.

Cosmonauts run their programs everywhere and all at once. Assemble all the flag fragments to win.
```
# Tools used
- Kali VM (VirtualBox)
- Windows OS
- Ghidra

# Running the executable
<img width="503" height="227" alt="1" src="https://github.com/user-attachments/assets/5ff84988-5828-4faf-9626-a2002e5ec99e" />

- The string is not closed by a "}", so it seems that I only got a part of the flag?
- "Like on Linux!" Huh. What if I make it run "Like on Windows" for example?
<img width="717" height="121" alt="image" src="https://github.com/user-attachments/assets/8bdba38b-d365-4555-8cea-be51519e377b" />

- Okay, I think I know what is going on, our program here prints parts of the flag based on the user's OS. The only issue is that God knows how many operating systems I will have to emulate before getting the entire flag and I am NOT wasting my time on all that.
# Static Analysis

- I got some strings from the output, so if I manage to find them in the string search maybe I will be able to find something of interest in their cross references?

<img width="915" height="243" alt="image" src="https://github.com/user-attachments/assets/6a710aa7-8154-4a17-9efc-cad648a3f4a4" />
<img width="1040" height="450" alt="image" src="https://github.com/user-attachments/assets/323d58c4-1d20-4f52-befa-41c8cb8a986a" />

- Would you look at that, they all lead to the same function, which consists of 4 conditional blocks.
<img width="808" height="859" alt="image" src="https://github.com/user-attachments/assets/e05dabc1-0d54-4f16-8b24-602e3a4dccb1" />
<img width="601" height="641" alt="image" src="https://github.com/user-attachments/assets/4e82ddbb-9ca1-4ba1-9f8d-675765011339" />

- FreeBSD, Windows, Linux and an else case that only calls one function, whereas the rest call two. Logically, I can emulate FreeBSD and probably get the last piece of the flag, but I don't want to waste my precious time downloading massive ISO files. And what if the last case also prints a part of the flag?
- I think it will be easier if I just remove all of the conditional jumps and let all of the cases execute in a single block.
<img width="1526" height="769" alt="image" src="https://github.com/user-attachments/assets/5cc45a27-63bb-40d7-b3d0-06f2fc19cdf4" />
<img width="1032" height="333" alt="image" src="https://github.com/user-attachments/assets/e8826359-d404-448b-a862-48ce5d060553" />

- Tada! This is not rocket science, you don't have to be a genius to be a reverse engineer!
- You just have to be a social inept freak with no life that will spend most of their time working reading 40 year old documentations about the internals of C/C++ and embedded systmes, in hopes of getting a job in a field that is in fact so niche that its job listings can be only found at the NSA or anti-malware development companies (How many of those do you know? OH YEAH LIKE FOUR?). Oh yeah and did I mention that said companies only hire best of the best like once half a decade and you have to SOMEHOW have at least one year of prior experience in this field to even try applying for an entry level position?
