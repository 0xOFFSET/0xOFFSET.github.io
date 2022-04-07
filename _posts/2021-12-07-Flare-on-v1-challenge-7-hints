---
title: Flare-on v1 challenge 7 hints
tags: RE
---

It was a cool experience, I enjoyed trying to solve these challenges and gaining new methodologies.

The Last challenge was slightly annoying, so I got hints from "Matt Graeber's" solution and it was really an awesome approach.

# Challenge 7 solution:
It's an PE file containing a series of 14 types of anti-debugging, anit-VM, network checks and other host-based checks.
Each check applies two multi-byte XOR operations on an embedded PE depending on which branch to take.

It's a time-consuming to defuse all these anti-debugging techniques, so the approach was as follow:
1. Extract the encoded embedded PE file.
2. Extract all XOR keys, every environment check has two keys, so we have 14 pairs of keys.
3. Calculate brute-force complexity: (2^14) - 1 = 16383 rounds , it seems simple.
4. Convert the number of each round to binary string and padding with zeros into 14 digits, which in result represents the current permutations of the environment checks.
( Each bit represents an environment check of the 14 in order, and whether bit is SET or UNSET, we decide to take each key of 2.
5. Taking only few bytes from the embedded PE to speed up the operations, then performing 14 xor operations for each permutation of 16383.
6. Comparing the PE header standard value with first bytes of the decoded PE.
7. Finally, we got the right permutation, then decoded the full embedded PE.
8. Later, it's a .NET executable with some encoded strings.
9. Another good approach to decode the strings, by letting the malware itself does so for us. By loading the .NET executable to "powershell" and dynamically invoke its decoder functions.
10. Finally, there're 3 emails, one of them is the flag.

Full write-up from fireeye: https://www.fireeye.com/blog/threat-research/2014/11/flare_on_challengep.html
