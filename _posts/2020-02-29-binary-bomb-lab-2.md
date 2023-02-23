---
title: "Binary Bomb Lab - phase 2"
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

this is the phase number 2. in the last writeup I have solver the first phase of the binary bomb lab, So now let's move to the second phase.

# Tools

same as before (just GDB)

# Debugging

I will start the debugger again with the same command used before. and set a break point on the call to ```phase_2``` function, using the command ```b phase_2``` then run.

[![1](/assets/images/reverseEngineerinig/phase2/1.png)](/assets/images/reverseEngineerinig/phase2/1.png)

it will ask for input for second phase. let's again just write ```omar``` for test. after the break point hits, now we are in the ```phase_2``` function. let's disassemble it first to take an overview of it 

[![2](/assets/images/reverseEngineerinig/phase2/2.png)](/assets/images/reverseEngineerinig/phase2/2.png)

first it initialize the stack and allocate 24 bytes in the stack  (ignore the moving value from the FS segment, it's just a way for checking of the stack. you can [read](https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value) this for more Information). then it calls ```read_six_numbers``` function with 2 parameters (our input (```rdi```) and value of the stack (```rsi```)). i can't see any definition for the variables allocated in the stack, so it seems that this function will allocate that variables. let's dive into this function and see what it do.

[![3](/assets/images/reverseEngineerinig/phase2/3.png)](/assets/images/reverseEngineerinig/phase2/3.png)

this is the disassembly of the ```read_six_numbers``` function. you can see that it calls ```sscanf``` function with many arguments. if you read the documentation of that function you will know that it accept a format and a string and try to match that string with the format. and also it can accept other pointer arguments to store the values of the result into it; so now the code make more sense. 

if you displayed the first argument passed you will notice that it's my input ```omar``` and the second argument is the format stored in address ```0x555555557303```. by examining it, it stores ```"%d %d %d %d %d %d"```. so it's just check if the input consists of six numbers then if that successed it will store that values at the addresses passed before as an argument (and that's actually the pointers to our variables allocated at the stack). then it checks if the return value > 5 (the condition appears in the assembly code is actually the opposite that has written in the source code. this is something related more about the compiler specifications). So now we done with that function. Let's continue with the ```phase_2``` function.

[![4](/assets/images/reverseEngineerinig/phase2/4.png)](/assets/images/reverseEngineerinig/phase2/4.png)

Immediately after the call it checks the value stored in the first variable is equals to one. and if not it will explode. then it goes to loop that inside it seems to be an ```if-else``` statement (when you see a conditional jump that followed by a code then an unconditional jump after them, you are probably looking at ```if-else``` statement ). so let's see what this loop do:

 - first at all before the loop, the address of the last variable in the stack is loaded to ```rbp``` register

 - then it jumps to the loop, that moves the value of the variable pointed by ```rbx``` (which is the first value that should be ```1```) then it doubles it and compare with the value of the variable that follows it, it they match it will go to the second iteration then test the second value by double it and compare with the next value and so on ...

 - and the loops breaks when the ```rbx``` reaches to the last value. so if you want to figure that moves :

   

   *High Address*

   | register                | variable   | address        |
   | ----------------------- | ---------- | -------------- |
   | **RBP** ===>            | variable 5 | **0X14(%RSP)** |
   |                         | variable 4 | **0X10(%RSP)** |
   |                         | variable 3 | **0X0C(%RSP)** |
   |                         | variable 2 | **0X08(%RSP)** |
   |                         | variable 1 | **0X04(%RSP)** |
   | **RSP**  & **RBX** ===> | variable 0 | **0X00(%RSP)** |

   *Low Address*

 - then if all condition successed. it will just return as usual. so now we know the values needed to solve it :

   - first the input has to consist of 6 numbers
   - the first number should be 1
   - and every number should be twice as the one before him
   - so the final answer would be : ```1 2 4 8 16 32```

   let's try it :

[![5](/assets/images/reverseEngineerinig/phase2/5.png)](/assets/images/reverseEngineerinig/phase2/5.png)

**I'M COMING TO YOU PHASE 3**

