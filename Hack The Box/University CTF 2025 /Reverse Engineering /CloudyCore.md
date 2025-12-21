# CloudyCore
**Category:** Reverse Engineering

**Difficulty:** Medium

**Description:**

```
Twillie, the memory-minder, was rewinding one of her snowglobes when she overheard a villainous whisper.
The scoundrel was boasting about hiding the Starshard's true memory inside this tiny memory core (.tflite).
He was so overconfident, laughing that no one would ever think to reverse-engineer a 'boring' ML file.
He said he 'left a little challenge for anyone who did,' scrambling the final piece with a simple XOR just for fun.
Find the key, reverse the laughably simple XOR, and restore the memory
```

## Challenge Overview

The challenge provides a TensorFlow Lite model: `snownet_stronger.tflite`

The description hints that the flag is hidden inside the model and is "scrambled with a simple XOR”.

## File Analysis

I opened this model in Netron to reveal its contents:
<img width="764" height="655" alt="image" src="https://github.com/user-attachments/assets/372aada3-c0b8-47f8-8355-3777cb54d644" />

### `in_payload`

* Shape: `[9, 1]`
* Type: `float32`
* Size: `9 × 4 = 36 bytes`

### `in_meta`

* Shape: `[1, 4]`
* Type: `float32`
* Values: (`~5.87e-39`)

## Step 1: Inspecting `in_meta`

I packed the `in_meta` floats as raw bytes with ```struct.pack("<4f", meta_floats)``` to take a deeper look

Output:

```
6b 00 40 00
33 00 40 00
79 00 40 00
21 00 40 00
```

And after converting it to ASCII I got the XOR k3y!

<img width="362" height="278" alt="image" src="https://github.com/user-attachments/assets/40731513-a204-49cd-b7ff-d5f2ff0e9201" />


## Step 2: Extract the Encrypted Payload

```python
import numpy as np

floats = np.array([
    6.158801631661256e-14,
    -4.444144483638194e+28,
    -5.419893129309592e-34,
    2.494911638184448e-33,
    3.1142920831693307e-27,
    4.140769657710109e-32,
    9.041799440998328e-17,
    4.870966301205419e-14,
    3.1809475140173348e-43
], dtype=np.float32)

as_uint32 = floats.view(np.uint32)
byte_data = as_uint32.tobytes()
print(byte_data)
```

Output: ```b'\x13\xaf\x8a)\x1a\x99\x0f\xefZ\x1b4\x88\xe7DO\tY\xbdv\x13E\x00W\x0b]}\xd0$k^[)\xe3\x00\x00\x00'```

or
```
13 af 8a 29 1a 99 0f ef 5a 1b 34 88 e7 44 4f 09
59 bd 76 13 45 00 57 0b 5d 7d d0 24 6b 5e 5b 29
e3
```

(36 bytes total)

## Step 3: XORing the encrypted payload with k3y!

```python
cipher = bytes.fromhex("13af8a291a990fef5a1b3488e7444f0959bd76134500570b5d7dd0246b5e5b29e3")
meta_key = b'k3y!'

flag = bytes(
    cipher[i] ^ meta_key[i % 4]
    for i in range(len(cipher))
)

print(flag)
```

Output: ```b'x\x9c\xf3\x08q\xaav\xce1(M\xa9\x8cw6(2\x8e\x0f2.3.*6N\xa9\x05\x00m"\x08\x88'```

Now, I did get stuck here for a while because I was expecting the flag instead of blob and I eventually had to ask ChatGPT for help.

Upon taking a look at my output, it immediately noticed that the first two bytes are:
```
78 9c
```
Which indicates that its a zlib header (how was I supposed to know that)

For those who don't know, zlib is a data-compression library that is designed to be used on any computer hardware and operating system, so at that point it was pretty obvious that I simply had to decompress the data to get the flag.

## Step 4: zlib Decompression
```shell
> >>> import zlib
> ... 
> ... cipher = bytes.fromhex(
> ...     "13af8a291a990fef5a1b3488e7444f0959bd76134500570b5d7dd0246b5e5b29e3"
> ... )
> ... 
> ... meta_key = b'k3y!'
> ... 
> ... compressed = bytes(
> ...     cipher[i] ^ meta_key[i % 4]
> ...     for i in range(len(cipher))
> ... )
> ... 
> ... print("compressed:", compressed)
> ... 
> ... flag = zlib.decompress(compressed)
> ... print("FLAG:", flag)
> ... 
> compressed: b'x\x9c\xf3\x08q\xaav\xce1(M\xa9\x8cw6(2\x8e\x0f2.3.*6N\xa9\x05\x00m"\x08\x88'
> FLAG: b'HTB{Cl0udy_C0r3_R3v3rs3d}'
> >>>
```
