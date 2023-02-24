---
title: "Binary Bomb Lab - phase 3"
classes: wide
tags: [BinaryBomb Phases]
header:
    teaser: /assets/images/reverseEngineerinig/7.png
description: "this binary was in x86_64 assembly course from OpenSecurityTraining2. and consist of 6 phases every one needs a special password to be defused (passed) otherwise it will blown up (not passed)."
Sample MD5:  59b57bdabee2ce1fb566de51dd92ec94
categories:
  - Reverse Engineering
toc: true
ribbon: red
---
# Introduction

nothing new, just phase 3.

# Before Start

Remember the [```Reading the Second Argument```](https://omarshehata11.github.io/reverse%20engineering/binary-bomb-lab-1/#reading-the-second-argument) section in the first phase write-up ? as i said before that in this code the program check if there's second argument passed to the program, and if so it will treat that argument as it's a name to file and try to get handle to it (using the ```fopen``` function), then it reads the content of that file to use every line as an input to the ```stdin``` it opens to accept the answer.

so in short we can make a file and write the answer of phases in separated lines, instead of running debugger every time and write the answer every time it asks for.

so i will make a file called answers, and write inside it :

```
I am just a renegade hockey mom.
1 2 4 8 16 32
```

then i will modify the ```gdbCfg``` to be:

```bash
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
start answers
```

> just modified **start** to pass answers file as an argument



# Debugging

so let's run the debugger, and set a breakpoint on```phase_3```.  before continue and enter a wrong answer for test, let's analyze the code at first and see what it wants :

[![1](/assets/images/reverseEngineerinig/phase3/1.png)](/assets/images/reverseEngineerinig/phase3/1.png)

It starts same as last phase, it calls```sscanf``` again to check the format of the input, if you examined the format parameter resides in ``` 0x55555555730f```, you will see that it stores: ```"%d %d"``` so it waits for two integers.

after that you will see something new, it's seems to be a ***jump table***, that's a way the compiler uses to implement the ***switch*** case in more better way; instead of just repeatedly use cmp and jmp.

[![2](/assets/images/reverseEngineerinig/phase3/2.png)](/assets/images/reverseEngineerinig/phase3/2.png)

It start with comparison to our first integer with value 7, so seems that that switch has only 7 cases. and you may notice that the conditional jump after that comparison is ```ja``` which indicates that we are dealing with ***unsigned*** integer, so it's a strong indicator that this is the number of cases of the switch.

then it do it's calculation to get the value of the address that it will jump to according to my first input. and in every case it zero's out the eax and do some mathematics operations on it. i think this is something weird because that every case will start from a specific point of that mathematics operation and continue without a jump; that's not like any normal switch case is implemented. so i can't really imagine this code  in high-level code, so if anyone know this is implemented in high-level, please mail me :)

so let's continue, after it did it's operation it will compare first my first integer if it bigger than 5, but now it treats it as a signed integer, and if so; it will explode. then it finally compares the value that was calculated before in eax and the second integer i have passed. so to solve it i just need to pass any value starting from 0-5 (not bigger because it will explode) and randomly choose the second integer, and see what would be the calculated value then try again with modify the second integer.

I will try with first value = 2, and see what will be the final value:

[![3](/assets/images/reverseEngineerinig/phase3/3.png)](/assets/images/reverseEngineerinig/phase3/3.png)

So the second value calculated as ```562``` . and every time you change the first value from its range, you will get another calculated one.

let's run again and enter the right password:

[![4](/assets/images/reverseEngineerinig/phase3/4.png)](/assets/images/reverseEngineerinig/phase3/4.png)

Don't forget to put that answer in the ```answers``` file !



***PHASE 4, DON'T BE AFRAID BUT WATCH OUT***







