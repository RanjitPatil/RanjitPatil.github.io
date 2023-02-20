---
title: "Binary Bomb Lab"
classes: wide
header:
teaser: /assets/images/malware-analysis/ksl0t-keylogger/12.png
ribbon: DodgerBlue
description: "this binary was in x86_64 assembly course from OpenSecurityTraining2. and consist of 6 phases every one needs a special password to be defused (passed) otherwise it will blown up (not passed)."
Sample MD5:  59b57bdabee2ce1fb566de51dd92ec94
categories:
  - Reverse Engineering
---
<!-- 
# Sample MD5: 59b57bdabee2ce1fb566de51dd92ec94
# layout: post
# title: Binary Bomb lab
# subtitle: reverse engineering OST2
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
# tags: [reverse]
// comments: true
-->
## Introduction
this binary was in x86_64 assembly course from OpenSecurityTraining2. and consist of 6 phases every one needs a special password to be defused (passed) otherwise it will blown up (not passed).

it has 2 binaries, one compiled for windows and other for linux, we will use the ELF file.

## Tools
we will use **GDB** debugger.

## Before start
first, I have used a file to load it to the debugger to execute some commands when the debugger start.

this file called gdbCfg, and contains:
```
display/10i $rip
display/x $rax
display/x $rbx
display/x $rdx
display/x $rcx
display/x $rsp
display/x $rbp
display/x $rdi
display/x $rsi
display/x $r8
display/x $r9
display/x $r12
display/x $r13
display/x $r14
display/10gx $rsp
start
```
first ```display/10i $rip``` to display 10 instructions starting from the instruction pointed to by rip.

and the rest to display the content of the registers.

```display/10gx $rsp``` to display the stack by size of 10 block below the rsp, every block (g) size (g = giant = 8 bytes).

and at last the ```start``` command to break at the main and start the debugging ( without parameters).

## Debugging
to start the debugger and load the binary file with gdbCfg file: 
```
 gdb bomb -q -x gdbCfg
 ```
 "bomb" > the name of the binary
 "-q" > to do not print the version number on startup.


after it starts, it will print out what we wrote in the gdbCfg file, but first I want to disassemble the main function, by writing ```disas``` to disassemble the function we are standing at :

[![Disas](/assets/images/reverseEngineerinig/1.png)](/assets/images/reverseEngineerinig/1.png)

### Reading the Second Argument
we see that it starts with comparison the value stored in ```rdi``` register if it equals to  1  or 2, if 1 it will just do call ```<initialize_bomb>``` function and continue, and if 2 it will read the second value pointed to by ```rsi```, we know that in system v call convention that the rdi stores the first argument ( which in our case is the ```argc``` argument) and the rsi stores the second argument (```argv```), so it seems that if I pass it an argument it will try to find that file and read it to it's input, but in our case now let's just ignore that.

 
### Phases Functions
from the picture above we see a pattern like call to function called ```<phase_1>``` then  after it call to ```<phase_defused>``` and also phase_2... etc. and before each on of them it calls ```<read_line>``` to read an input. so now we have an overview for how this program work in general. let's just reverse every function and see what it needs.

#### phase 1
so by stepping over to reach the call of  phase_1 function (use ```ni``` command in gdb), we noticed that after the call of the ```<puts>``` function (similar to ```printf``` function) twice it printed to us: 
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
```
then it calls ```read_line``` and wait for us to enter an input, let's just enter ```omar```...

[![before the call of phase_1](/assets/images/reverseEngineerinig/2.png)](/assets/images/reverseEngineerinig/2.png)

so it moves the returned value of ```read_line``` function to the ```rdi``` register then calls ```phase_1``` function, so it takes our input as an argument to phase_1 function, so let's step-into the function(use ```si``` command) and disassemble it: 

[![disassemble of phase_1 function](/assets/images/reverseEngineerinig/3.png)](/assets/images/reverseEngineerinig/3.png) 

we see that it moves a value to ```rsi``` then it calls ```<strings_not_equal>``` function, so it's the second argument of it. then it checks the return value, if it equals to 0 it will continue otherwise it will jump to call function ```<explode_bomb>``` which sounds it a bad choice to us. so we need to make ```strings_not_equal``` returns zero. but first let's examine what in the first and second argument to that function.

```x/s $rdi``` will print to us ```omar```value, which is our input, and second argument can be examined by: ```x/s 0x555555557150``` as the debugger lists(this is the address calculated after ```$rip-0x1b9a```) or by just execute the instruction of the load and examine ```rsi``` it self, it doesn't matter. we see that it contain : ```I am just a renegade hockey mom.```. that's an interested string, let's now disassemble the ```strings_not_equal``` function without stepping-into it; by just type : ```disas strings_not_equal``` in gdb, it will print :

[![disassemble of strings_not_equal function](assets/images/reverseEngineerinig/4.png)](assets/images/reverseEngineerinig/4.png)

here we notice many things, first it call ```string_length``` twice, on with argument of our input and the other with Harden-string as an argument to get their size, then compare the size of them, if not equaled it will return 1, and if equal will go to next test.

and here you can see that it identifies a loop, starting with the first byte of my input in the instruction : ```movzbl (%rbx), %edx``` (it's similar to ```movzx edx, BYTE PTR[rbx]``` in intel flavor, it just moves the first character of my input and put it in edx). then it enters the loop and compare every character of my input to every character in the Harden-string (with the same index of course). and if it hit the Null byte it will break and return zero and exit.

so it simply compares my input to the Harden-string (and it's just easy to know from the name of the function). so now if I completed the debugging with my input it will call ```explode_bomb  ``` :

[![execute with wrong answer](/assets/images/reverseEngineerinig/5.png)](/assets/images/reverseEngineerinig/5.png)

let's try now with the new answer : ```I am just a renegade hockey mom.``` :

[![answer of phase_1](/assets/images/reverseEngineerinig/6.png)](/assets/images/reverseEngineerinig/6.png)

now, let's move to the second phase.

#### phase 2
