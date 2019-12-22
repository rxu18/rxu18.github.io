---
title: "X-MAS CTF Writeup"
permalink: "ctf/xmas"
---
## X-MAS CTF
I competed under team `two idiots`. This is the first CTF my team has done, so everything is new. Still, we managed to solve a few awesome challenges. Below are challenges I solved and found interesting.

### Emu 2.0
We receive a ROM drive and a pdf describing how the operating system works, so the task is clearly to write an emulator. Below is some C++ code for my emulator.
```C++
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <unistd.h>

int main() {
    FILE *f = fopen("rom", "rb");
    fseek(f , 0 , SEEK_END);
    size_t size = ftell(f);
    rewind(f);
    unsigned char ram[1 << 12];
    memset(ram, 0, 1 << 12);
    if (fread(ram + 0x100, 1, size, f) != size) {
        printf("Hmm strange");
    } else {
        bool usable[1 << 12];
        for (int i = 0; i < 1 << 12; ++i) usable[i] = true;
        unsigned char a = 0;
        int pc = 0x100;
        while (true) {
            unsigned char i1 = ram[pc], i2 = ram[pc + 1];
            unsigned char b1 = i1 / 16, b2 = i1 % 16, b3 = i2 / 16, b4 = i2 % 16;
            int ref = ((int) b2 << 8) + i2;
            // printf("rip = %d rax = %x instr = %x %x ref = %x\n", pc, a, i1, i2, ref);
            pc += 2;
            switch(b1) {
                case 0:
                    switch(b2) {
                        case 0: a += i2; break;
                        case 1: a = i2; break;
                        case 2: a = a ^ i2;break;
                        case 3: a = a | i2;break;
                        case 4: a = a & i2;break;
                        default: a--;
                    }
                    break;
                case 1:
                    if (b2 == 3 && i2 == 0x37) {
                        printf("%c", a);
                        fflush(stdout);
                    } else a--;
                    break;
                case 2:
                    pc = ref;
                    break;
                case 3:
                    if (a == 0) pc = ref;
                    break;
                case 4:
                    if (a == 1) pc = ref;
                    break;
                case 5:
                    if (a == 0xff) pc = ref;
                    break;
                case 6:
                    if (b2 == 0) {
                        if (a == i2) a = 0;
                        else if (a < i2) a = 1;
                        else a = 0xff;
                    } else a--;
                    break;
                case 7: {unsigned char comp = ram[ref];
                    if (a == comp) a = 0;
                    else if (a < comp) a = 1;
                    else a = 0xff;}
                    break;
                case 8:
                    a = ram[ref];
                    break;
                case 9:
                    usable[ref] = false;
                    break;
                case 0xa:
                    usable[ref] = true;
                    break;
                case 0xb:
                    if (b2 == 0xe && i2 == 0xef) {pc = 0x100; a = 0x42;}
                    else a--;
                    break;
                case 0xc:
                    if (usable[ref]) ram[ref] = ram[ref] ^ 0x42;
                    break;
                case 0xd:
                    if (usable[ref]) ram[ref] = ram[ref] ^ a;
                    break;
                case 0xe:
                    if (b2 == 0xe && i2 == 0xee);
                    else a--;
                    break;
                case 0xf:
                    if (usable[ref]) ram[ref] = a;
                    break;
                default:
                    printf("Should not get here!");
            }
        }
    }
}
```
Running this code gives the key `X-MAS{S4nt4_U5e5_An_Emu_2.0_M4ch1n3}`.

### Dox the Grinch
This is an OSINT challenge, which involves visiting different sites in the internet to find relevant information about a (fictional) person. Here is how we did it.

The challenge starts with a [notabug post](https://notabug.io/t/whatever/comments/44530e6b7740f22940db9c176b621900d0bce697/i-hate-xmas), with the goal of finding out information about `domay1986`. The list of posts contains 2 that are relevant:
1. `Is HackerNews any good?` suggests a visit to Hackernews. A quick visit finds [this profile](https://news.ycombinator.com/user?id=Domay1986), with his first name: `Eugene`.
2. `In defense of the "selfish" baby boomers` points toward Facebook. Searching up `Eugene 1986` on FaceBook finds this [profile](https://www.facebook.com/eugene.clarke.56232). The Romanian references tell us this is what we are looking for.
3. The FaceBook post contains a link back to the xmas ctf website. Using the standard SQL injection `' or true or '`, we get access to a fictional medical record with his address, blood type and height.
4. A search with the address reveals that Domay lives in Scranton (where the Office was filmed). The blood type was 0-.
5. Back on the FaceBook page, the xkcd screenshot contains unclosed tabs. One of them has title `Matrimoniale`, and we find a Romanian dating website with [Domay's profile](https://www.matrimoniale.ro/Domay1986), revealing his favorite color: magenta.

The key was `X-MAS{eugene_clarke_scranton_magenta_0-_162}`.

### Pythagoreic Pancakes
This challenge requires you to input different Pythagorean triples. Not knowing how to use a circuit, we perform the inputs by hand. Below is some python code to generate Pythagorean triples.

```python
from math import gcd

a = []
m = 3000
for i in range(1,m):
    for j in range(i+1,m,2):
        if gcd(i,j) == 1:
            b = [j * j - i * i, 2 * i * j, i * i + j * j]
            b.sort()
            a.append(b)

a.sort(key=lambda x:(x[2], x[1]))
print("ready for query")
while True:
    s = input()
    print(str(a[int(s) - 1])[1:-1])
```

I did not record the key, but can confirm that the code above works.