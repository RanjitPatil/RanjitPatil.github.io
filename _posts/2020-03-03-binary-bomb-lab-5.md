---
title: "Binary Bomb Lab - phase 5"
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

just phase 5 i swear 

# Debugging

so let's disassemble it at first:

[![1](/assets/images/reverseEngineerinig/phase5/1.png)](/assets/images/reverseEngineerinig/phase5/1.png)

same scenario, by examining ```0x55555555730f```, it stors : ```"%d %d"```. now let's analyze block by block.

```
   0x00005555555557f5 <+49>:    mov    (%rsp),%eax
   0x00005555555557f8 <+52>:    and    $0xf,%eax
   0x00005555555557fb <+55>:    mov    %eax,(%rsp)
   0x00005555555557fe <+58>:    cmp    $0xf,%eax
   0x0000555555555801 <+61>:    je     0x555555555835 <phase_5+113>
   0x0000555555555803 <+63>:    mov    $0x0,%ecx
   0x0000555555555808 <+68>:    mov    $0x0,%edx
   0x000055555555580d <+73>:    lea    0x19ac(%rip),%rsi        # 0x5555555571c0 																	<array.3471>
   0x0000555555555814 <+80>:    add    $0x1,%edx
   0x0000555555555817 <+83>:    cltq
   0x0000555555555819 <+85>:    mov    (%rsi,%rax,4),%eax
   0x000055555555581c <+88>:    add    %eax,%ecx
   0x000055555555581e <+90>:    cmp    $0xf,%eax
   0x0000555555555821 <+93>:    jne    0x555555555814 <phase_5+80>
```

 so it start with first taking my first input and AND it with 15, that means that it deals with only first byte of my input, in otherword my first input will be treatet as it can hold at max value 15.

so it test it if it uquals to 15 it will jump to ```explode_bomb```. then it zero's out ecx & edx, this is usually happens when it will enter a loop and it needs a counter an so on.

then it loads an address to rsi, you can see that GDB has identified it as an address to an array. then it increments the edx and conver it from long to quad-word. access that last array with its base (rsi) and index eax and scale 4.then add the value returned from that array access with ecx and see if eax equals to 15, and if not it will repeat that loop.

the most important thing i see that it uses the same register to store the value of the returned value from the array, and use it also as an index. so it's like you get a value from the array, then use that value as an index to get another value from the array and so on.

I mentioned before that it checks eax if it equals to 15 as max value or not, meaning that this array seems to have 16 elemnt (0 --> 15 == 16 element). let's see what this array stores using this command : ```x/16wx 0x5555555571c0 ``` 

> ```w``` which mean word (4 byte) ( word in gdb means 4 byte, not like other names that treat word as 2 byte.) because it uses an scale of 4 when it access the array, so it's seems to be an array of integers.
>
> so don't be confused , ```long``` in AT&T is the name of size of 4 byte, but ```word``` in gdb debugger is the name of size 4 byte. 

  so it will print to us : 

```
0x5555555571c0 <array.3471>:    0x0000000a      0x00000002      0x0000000e      0x00000007
0x5555555571d0 <array.3471+16>: 0x00000008      0x0000000c      0x0000000f      0x0000000b
0x5555555571e0 <array.3471+32>: 0x00000000      0x00000004      0x00000001      0x0000000d
0x5555555571f0 <array.3471+48>: 0x00000003      0x00000009      0x00000006      0x00000005
```

and if you just make it prints 17 number instead of 16 you will see that it prints some random value out of the array. so now let's rewrite that elemnt in a clear array : ```[10, 2, 14, 7, 8, 12, 15, 11, 0, 4, 1, 13, 3, 9, 6, 5]``` you will notice that it even its values are values between 0 --> 15. so we will get out of that loop when eax get the value of 15 stored in array

let's go to another block and alayze it.

```
   0x0000555555555823 <+95>:    movl   $0xf,(%rsp)
   0x000055555555582a <+102>:   cmp    $0xf,%edx
   0x000055555555582d <+105>:   jne    0x555555555835 <phase_5+113>
   0x000055555555582f <+107>:   cmp    %ecx,0x4(%rsp)
   0x0000555555555833 <+111>:   je     0x55555555583a <phase_5+118>
   0x0000555555555835 <+113>:   call   0x555555555be5 <explode_bomb>
   0x000055555555583a <+118>:   mov    0x8(%rsp),%rax
   0x000055555555583f <+123>:   xor    %fs:0x28,%rax
   0x0000555555555848 <+132>:   jne    0x555555555856 <phase_5+146>
   0x000055555555584a <+134>:   add    $0x18,%rsp
   0x000055555555584e <+138>:   ret
   0x000055555555584f <+139>:   call   0x555555555be5 <explode_bomb>
   0x0000555555555854 <+144>:   jmp    0x5555555557f5 <phase_5+49>
   0x0000555555555856 <+146>:   call   0x555555555220 <__stack_chk_fail@plt>
```

Then now it checks if edx ( which was treated as a counter to us in the previos block) equals to 15, and if not the bomb will explode! so we need to make edx holds 15; and the only way to do so is to make the previos loop iterate 15 times. then lastly it checks if my second input equals to ecx, which was used lately as a **sum** to our elemnts (at offset ```<+88>```). so let's connect everything together.

- to pass this level we need our second input to be the sum of the elemnts - the value of my first input (because my first input will just treated as index without adding it).
- and the first input should be a some value less than 15 that will make the first loop iterate 15 times! 
- we know that the loop works as it first take my first input and use it as an index to get the value indexed by it in the array, then checks if it equals to 15 and if not it will that value as another index to enter the loop and so on.
- so we need to give the first input an index that will from it take a value to go to another value and do so 15 times and end that by lastly get the index of the value 15 stored in array
- so we can go reversly, by knowing what's the index of 15 ? it's index is (6) so the last value that the eax should store before get the value is the index to the value 6; which is 14, then again what's the index of the value 14 ? it's 2. so the before it the value should be stored is the index of the value 2; which is 1...etc (you should repeat this 14 times because already edx starts with 1 not 0).
- if you calculated it at this way you will find that you need the first to be ```5```. 
- and according to so, the second value should be the sum (120) - that value (5) = 115. let's try out:

[![2](/assets/images/reverseEngineerinig/phase5/2.png)](/assets/images/reverseEngineerinig/phase5/2.png)

***HERE YOU ARE***

