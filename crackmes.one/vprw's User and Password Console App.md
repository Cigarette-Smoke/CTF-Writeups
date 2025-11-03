Link: https://crackmes.one/crackme/68e54bb82d267f28f69b740f
# vprwv's Random user and pw protected console app
<img width="1215" height="294" alt="image" src="https://github.com/user-attachments/assets/d571aaef-c83f-49b6-9d0c-76b7cd8af369" />


### Tools used
- Ghidra

# String Analysis
<img width="1470" height="747" alt="image" src="https://github.com/user-attachments/assets/9258c1fa-0890-43f7-94fa-dc6f6d88ea23" />
- The initial analysis reveals the following 2 strings: "Enter your name:" and "User not recognized:".

<img width="1918" height="1018" alt="image" src="https://github.com/user-attachments/assets/8ce2a32d-1f3a-4a10-b629-39044b72144a" />

- These strings can be used to find the starting point for binary analysis. Using ghidra's string search tool find the strings in the disassembler.
<img width="1876" height="802" alt="image" src="https://github.com/user-attachments/assets/e9d4e19c-cec2-4231-a0d7-16655415eff3" />

# Binary Analysis
- There are no COUT/CIN calls nearby and that our decompiler is empty. That is because this is the location where strings are stored, not used.
- I follow the XREFs, aka cross references, because they point to the location in the file where our strings are used.

<img width="1076" height="367" alt="image" src="https://github.com/user-attachments/assets/30085496-8d73-4d0e-a0c9-76df5dac3bda" />

- We can finally see the logic that matches the behaviour of the program that we've seen during the String Analysis, which means that we are moving in the correct direction.

<img width="1215" height="757" alt="image" src="https://github.com/user-attachments/assets/97fcd029-d6c6-4226-8610-23d1da6a1a99" />

- The decompiler shows that bVar is in charge of the control flow.

<img width="877" height="337" alt="image" src="https://github.com/user-attachments/assets/77e5421a-85c2-4160-9839-86c6824f0237" />
<img width="1590" height="362" alt="image" src="https://github.com/user-attachments/assets/588d000b-8492-4d51-9547-a80d552f0581" />

- The disassembler shows that
