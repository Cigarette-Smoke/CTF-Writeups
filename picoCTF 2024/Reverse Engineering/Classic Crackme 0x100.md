Link: https://play.picoctf.org/practice/challenge/409?category=3&originalEvent=73&page=1

# Classic Crackme 0x100

Author: Nandan Desai

Difficulty: Medium

Description:

```
A classic Crackme. Find the password, get the flag!
Binary can be downloaded here.
Crack the Binary file locally and recover the password. Use the same password on the server to get the flag!
Access the server using nc titan.picoctf.net XXXXX
```

#### Tools Used
- Ghidra

## Taking a look at the server program
<img width="630" height="598" alt="image" src="https://github.com/user-attachments/assets/d3ad28fd-5f64-4991-8523-678af7b24444" />

- Let's retrieve the password of this program by analysing its binary with a static analysis tool.

## Binary Analysis
- The decompiler reveals a strcpy() function that copies a hardcoded string to a variable named "output". I could tell that this variable contains the password because I noticed that at line 37 that same variable is compared to our input.
<img width="738" height="739" alt="image" src="https://github.com/user-attachments/assets/e84736b6-42da-4a2d-8637-fdf50574243e" />

- Entering the hardcoded string into the server program won't print us the flag because at line 34 we can see that our input is decrypted through a cipher.
<img width="649" height="273" alt="image" src="https://github.com/user-attachments/assets/211c3c3f-26a6-47e3-9a66-9f6a3fea1524" />

- Let's reverse this cipher to get a encrypted version of the password.

## Reversing the cipher

- In case you don't know what the operands inside of the algorithm do, I have attached notes explaining them.
```C
int i_1;
int i = 0;
sVar3 = strlen(output);

for (; i < 3; i = i + 1) {
  for (i_1 = 0; i_1 < (int)sVar3; i_1 = i_1 + 1) {
    uVar1 = (i_1 % 255 >> 1 & 85U) + (i_1 % 255 & 85U);
    uVar1 = ((int)uVar1 >> 2 & 51U) + (uVar1 & 51);
    iVar2 = ((int)uVar1 >> 4) + input[i_1] + -97 + (uVar1 & 15);
    input[i_1] = (char)iVar2 + (char)(iVar2 / 26) * -26 + 'a';
  }
}
```
![20251104_125015](https://github.com/user-attachments/assets/81390102-fe50-43a3-8da5-87d24b50e44a)

- Here is the code that I came up with:
```C
#include <stdio.h>
#include <string.h>

int main(void) {
    char input[51] = "mpknnphjngbhgzydttvkahppevhkmpwgdzxsykkokriepfnrdm";
    size_t sVar3 = strlen(input);
    int i_1;
    unsigned int uVar1;
    int shift, iVar2;

    for (i_1 = 0; i_1 < (int)sVar3; i_1++) {
        // compute uVar1 the same way
        uVar1 = (i_1 % 255 >> 1 & 85U) + (i_1 % 255 & 85U);
        uVar1 = ((int)uVar1 >> 2 & 51U) + (uVar1 & 51);

        // total shift (3 times per encryption)
        shift = ((uVar1 >> 4) + (uVar1 & 15)) * 3;

        // reverse modular addition
        iVar2 = (input[i_1] - 'a' - shift) % 26;
        if (iVar2 < 0) iVar2 += 26;

        input[i_1] = (char)(iVar2 + 'a');
    }

    printf("%s\n", input);
    return 0;
}
```

<img width="620" height="106" alt="image" src="https://github.com/user-attachments/assets/d558adce-387d-4905-ad55-d4499ce2c734" />

<img width="747" height="778" alt="image" src="https://github.com/user-attachments/assets/da658510-b34f-49de-9e77-23c07e096afb" />
