Link: https://crackmes.one/crackme/68e54bb82d267f28f69b740f
# vprwv's Random user and pw protected console app
<img width="1215" height="294" alt="image" src="https://github.com/user-attachments/assets/d571aaef-c83f-49b6-9d0c-76b7cd8af369" />


### Tools used
- Ghidra

# Launching the executable
<img width="1470" height="747" alt="image" src="https://github.com/user-attachments/assets/9258c1fa-0890-43f7-94fa-dc6f6d88ea23" />
- The program outputs these 2 strings: "Enter your name:" and "User not recognized:".

<img width="1918" height="1018" alt="image" src="https://github.com/user-attachments/assets/8ce2a32d-1f3a-4a10-b629-39044b72144a" />

- These strings can be used to find the starting point for our binary analysis. I used ghidra's string search tool to find the said strings in the disassembler.
<img width="1876" height="802" alt="image" src="https://github.com/user-attachments/assets/e9d4e19c-cec2-4231-a0d7-16655415eff3" />

# Binary Analysis
- There are no COUT/CIN calls nearby and our decompiler is empty. That is because we are looking at a location where our strings are stored, not used.
- I follow the XREFs, aka cross references, because they point to the location in the file where our strings are used.

<img width="1076" height="367" alt="image" src="https://github.com/user-attachments/assets/30085496-8d73-4d0e-a0c9-76df5dac3bda" />

- I assume that bVar is the boolean result of strcmp(name, input) because it is used to control the flow between "Enter your name:" and "User not recognized".

<img width="663" height="360" alt="image" src="https://github.com/user-attachments/assets/36ef0458-390b-429e-b4eb-c8d45f71d3c5" />

- In the disassembly window I can see a conditional jump ```JZ```, meaning the program jumps over the "User not recognized" section only if the strings are equal.

<img width="1590" height="362" alt="image" src="https://github.com/user-attachments/assets/588d000b-8492-4d51-9547-a80d552f0581" />

- Lets patch the instruction to change it to an unconditional jump, making the program always jump over the "User not recognized" section.

<img width="884" height="156" alt="image" src="https://github.com/user-attachments/assets/08f54713-6ae5-443d-831c-cc30d397e694" />
<img width="888" height="414" alt="image" src="https://github.com/user-attachments/assets/55fb7340-8932-40f9-ba4e-509556db8e43" />

- Now we have to pass the password check, the code for password checking is the same, so we can repeat the same steps.

<img width="872" height="214" alt="image" src="https://github.com/user-attachments/assets/e45c80ee-6ac3-4249-8c0d-2bf2e6544638" />
<img width="892" height="192" alt="image" src="https://github.com/user-attachments/assets/5de05176-78bc-4b09-b82c-0ef2774d5370" />

- Export the program and check see if it works
<img width="1472" height="745" alt="image" src="https://github.com/user-attachments/assets/3100274e-4a99-416e-8bf4-bf5cc2df4df4" />

