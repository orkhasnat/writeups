# Forenscript
Category: #Forensic
Difficulty: Medium
>It's thundering outside and you are you at your desk having solved 4 forensics challenges so far. Just pray to god you solve this one. You might want to know that sometimes too much curiosity hides the flag.
---
In the challenge, a [file](a.bin) with a `.bin` extension was provided. Running the `file` command didn't yield anything useful. In any forensic problem, examining the hexdump of the file is crucial. So I looked into the hexdump.

### Hidden Details in the Hexdump
Running `xxd` on the file produced a peculiar hexdump:
```bash
➜➜ xxd a.bin | head
00000000: 474e 5089 0a1a 0a0d 0d00 0000 5244 4849  GNP.........RDHI
00000010: 460c 0000 a504 0000 0000 0608 3dab 1f00  F...........=...
00000020: 0000 007c 4752 7301 ceae 0042 0000 e91c  ...|GRs....B....
00000030: 4167 0400 0000 414d fc0b 8fb1 0000 0561  Ag....AM.......a
00000040: 4870 0900 0000 7359 0000 871d 8f01 871d  Hp....sY........
00000050: 0065 f1e5 49e3 ec00 7854 4144 3ffc ec5e  .e..I...xTAD?..^
00000060: 1add 6dab f7ea 6bd6 302a 48a8 a311 3032  ..m...k.0*H...02
00000070: c2c0 acc2 cd14 4558 3363 444c f09c 93a3  ......EX3cDL....
00000080: 06a6 2604 ccc0 7082 10b0 b0c8 8181 3f33  ..&...p.......?3
00000090: 8814 214a bba7 5be1 f6f6 eeed 8e6b 1bfd  ..!J..[......k..
```

Upon closer inspection, strings like `GNP` and `RDHI` appear, resembling `PNG` and `IHDR` found in PNG files. Notably, the byte order is reversed for every 4 bytes. I reversed the bytes in groups of 4 using the following python script:

```python
with open("a.bin", "rb") as ifile:
  with open("broken.png", "wb") as ofile:
    while True:
      chunk = ifile.read(4)
      if not chunk:
        break

      ofile.write(chunk[::-1])

```

The resulting PNG file is, ![broken.png](broken.png)

### Oh no, a red herring, or is it?
The image says "Fake Flag!!!", so I delved deeper. Analyzing the hexdump of the new PNG file in the hex editor, I found another PNG file hidden within it. I simply extracted the embedded PNG file using the following python script:

```python
# Hidden PNG from offset 60048 to 127482
start = 60048
end = 127482
with open("image1.png", "rb") as ifile:
  with open("extracted.png", "wb") as f:
    ifile.seek(start)
    cc = ifile.read(end - start)
    f.write(cc)
```
 
Alternatively, you can use `binwalk` to extract it.

## Flag
And there you have it—the flag: ![extracted](extracted.png)

> Flag: `flag{scr1pt1ng_r34lly_t0ugh_a4n't_1t??}`