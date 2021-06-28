# Caesar-Shift
Caesar Shift written in AT&amp;T x86 that my group and I wrote during my third semester at NJIT in Spring 2021.

## About
This project contains a text file of the code, the caesar program, the caesar.s, and caesar.o files for your reference. The code was written in the caesar.s file, everything else was compiled using `as -g -o caesar.o caesarText.s && ld -o caesar caesar.o`. You can make edits directly to the caesar.s file if you wish to play around with the code.

The caesarText.txt file is just a file identical to the caesar.s file. This is so the code is readable without using vim, vi, or another editor. I do not know if this will work on Windows, since it was only written and tested on a Linux Virtual Machine. My assumption is that the .s file won't be properly compilable by default on Windows since it is in AT&T x86 format instead of the Intel format. You can convert the code to Intel format using GDB or some other converter and then bring it over to Windows.
