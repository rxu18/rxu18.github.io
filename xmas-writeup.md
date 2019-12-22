---
layout: page
title: "X-MAS CTF Writeup"
date: YYYY-MM-DD hh:mm:ss -0000
permalink: "ctf/xmas"
layout: default
categories: CTF
---
## X-MAS CTF
I competed under team `two idiots`. This is the first CTF my team has done, so everything is new. Still, we managed to solve a few awesome challenges. Below are challenges I solved and found interesting.

### Emu 2.0

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
This challenge requires you to input different Pythagorean triples. Not knowing how to use a circuit, we perform the inputs by hand. The code to crack the SHA256 lock and the Pythagorean code will be attached soon.