---
title: "Binary Bomb Lab - phase 6"
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

the last phase. phase 6

# Debugging

the disassembly :

[![1](/assets/images/reverseEngineerinig/phase6/1.png)](/assets/images/reverseEngineerinig/phase6/1.png)
[![2](/assets/images/reverseEngineerinig/phase6/2.png)](/assets/images/reverseEngineerinig/phase6/2.png)

so as you can see it's a big bunch of code. let's test block by block.


```
   0x000055555555585b <+0>:     endbr64
   0x000055555555585f <+4>:     push   %r14
   0x0000555555555861 <+6>:     push   %r13
   0x0000555555555863 <+8>:     push   %r12
   0x0000555555555865 <+10>:    push   %rbp
   0x0000555555555866 <+11>:    push   %rbx
   0x0000555555555867 <+12>:    sub    $0x60,%rsp
   0x000055555555586b <+16>:    mov    %fs:0x28,%rax
   0x0000555555555874 <+25>:    mov    %rax,0x58(%rsp)
   0x0000555555555879 <+30>:    xor    %eax,%eax
   0x000055555555587b <+32>:    mov    %rsp,%r13
   0x000055555555587e <+35>:    mov    %r13,%rsi
   0x0000555555555881 <+38>:    call   0x555555555c11 <read_six_numbers>
   0x0000555555555886 <+43>:    mov    $0x1,%r14d
   0x000055555555588c <+49>:    mov    %rsp,%r12
   0x000055555555588f <+52>:    jmp    0x5555555558b9 <phase_6+94>

```

same what was used before in [phase 2](https://omarshehata11.github.io/reverse%20engineering/binary-bomb-lab-2/). So according to our previous analysis it just needs six integer numbers. this block just check for it. let's move on to the next block:

```
   0x0000555555555898 <+61>:    add    $0x1,%rbx
   0x000055555555589c <+65>:    cmp    $0x5,%ebx
   0x000055555555589f <+68>:    jg     0x5555555558b1 <phase_6+86>
   0x00005555555558a1 <+70>:    mov    (%r12,%rbx,4),%eax
   0x00005555555558a5 <+74>:    cmp    %eax,0x0(%rbp)
   0x00005555555558a8 <+77>:    jne    0x555555555898 <phase_6+61>
   0x00005555555558aa <+79>:    call   0x555555555be5 <explode_bomb>
   0x00005555555558af <+84>:    jmp    0x555555555898 <phase_6+61>
   0x00005555555558b1 <+86>:    add    $0x1,%r14
   0x00005555555558b5 <+90>:    add    $0x4,%r13
   0x00005555555558b9 <+94>:    mov    %r13,%rbp
   0x00005555555558bc <+97>:    mov    0x0(%r13),%eax
   0x00005555555558c0 <+101>:   sub    $0x1,%eax
   0x00005555555558c3 <+104>:   cmp    $0x5,%eax
   0x00005555555558c6 <+107>:   ja     0x555555555891 <phase_6+54>
   0x00005555555558c8 <+109>:   cmp    $0x5,%r14d
   0x00005555555558cc <+113>:   jg     0x5555555558d3 <phase_6+120>
   0x00005555555558ce <+115>:   mov    %r14,%rbx
   0x00005555555558d1 <+118>:   jmp    0x5555555558a1 <phase_6+70>
```

the last block will make the rip jumps first to the offset ```<+94>```. the ```r13``` register contains the value of the```rsp``` register (see ```<+32>``` offset). so it actually points to our input values stored inside the stack. so it first taking my first integer value and check if it above 5 or not to take the jump after it decrements it. so it actually check if my input is less than or equal to 6 (as said before that the condition in assembly is the opposite the one written in high-level code.). so it not above it will check for the register ```r14``` and if it greater than 5 or not. and if not it moves its value to ```rbx``` then jumps.

Then it uses the ```rbx``` register as an index to access then values I have entered. and it should be not equal to my first input to not blown up. then try again but with incrementing the index. and again it returns to offset ```<+x86>``` and repeat the operation but with some increments. so it's very clear that those are two nested loops that checks if my inputs are unique or not repeated; and also the max value for my input is 6. so if I want to implement those loops in high level :

```c
int counter = 0
for (int i = 1; i <= 5; i++){
	if ( myInput[counter] > 6 ) // assume that my input is stored in an array.
		explod_bomb();
	
	for (int y = i; y <= 5; y++){
		if ( myInput[counter] == myInput[y] ) // to check they aren't repeated
			explode_bomb(); 
		}
		
	counter++;

}
```

 Now let's move on to the next block :

```
   0x00005555555558d3 <+120>:   mov    $0x0,%esi
   0x00005555555558d8 <+125>:   mov    (%rsp,%rsi,4),%ecx
   0x00005555555558db <+128>:   mov    $0x1,%eax
   0x00005555555558e0 <+133>:   lea    0x3919(%rip),%rdx        # 0x555555559200 <node1>
   0x00005555555558e7 <+140>:   cmp    $0x1,%ecx
   0x00005555555558ea <+143>:   jle    0x5555555558f7 <phase_6+156>
   0x00005555555558ec <+145>:   mov    0x8(%rdx),%rdx
   0x00005555555558f0 <+149>:   add    $0x1,%eax
   0x00005555555558f3 <+152>:   cmp    %ecx,%eax
   0x00005555555558f5 <+154>:   jne    0x5555555558ec <phase_6+145>

```

Here it starts again to enter my input values with ```esi``` used as an index and loads some address to ```rdx```, you can see that GDB identified it as a ```node```. so it actually seems to be some sort of structure. then it compares the value came from my input array with ```esi``` used as an index and check if it less than or equal to 1 or not. and if so it will jump out of that block of code. but if not it will use the offset of (+0x8) of the node and treat it as an address then increments the ```eax``` and check if it equals to my input or not. so this is actually another loop that gets specific node according to my input. so if my input is 4 it will get the 4th node and so on. we know before that the max size of my input should be 6 so it seems that there are 6 nodes because it uses my input as an index to get the node. so let's examine that address of the node and display them. but what's the size ? we also saw that it uses the offset +0x8 to get the next node; and use the value at that offset as an ***address***, and we know that the address is 8-byte long (in x64); so the final size - at least - of a node should be 16 (8(before address) + 8 (size of address)). and we need to display 6 nodes. so we need to display (6 * 8 = 48). which is same as 11 elemnts every one is 8 byte(```g``` == ```giant```) or 22 element every one  4 byte (```l``` == ```long```). all are the same, let's examine the giant size : 

```x/11gx  0x555555559200```  which will display :

[![3](/assets/images/reverseEngineerinig/phase6/3.png)](/assets/images/reverseEngineerinig/phase6/3.png)

it did not print the 6th node. but you can notice that the at every node at offset ```+x08``` it stores an address to the node after it. so it's seems to be a linked list; which the last element in it is the ```next``` pointer. all stores the address of the node reside after it in the memory except the last node it points else where, so the 6th node should be stored there, let's examine it by command ``` x/2gx 0x0000555555559110``` :

[![4](/assets/images/reverseEngineerinig/phase6/4.png)](/assets/images/reverseEngineerinig/phase6/4.png)

here you are. now let's represent the structure of every node :

```c++
struct Node
{
	int unKnown;
	int order;
	Node* next;
};
```

the first value of every node seems to store some random int value, and the second seems to be the order of the node and the third is the next pointer.

so now let's move to the next block of code:

```
   0x00005555555558f7 <+156>:   mov    %rdx,0x20(%rsp,%rsi,8)
   0x00005555555558fc <+161>:   add    $0x1,%rsi
   0x0000555555555900 <+165>:   cmp    $0x6,%rsi
   0x0000555555555904 <+169>:   jne    0x5555555558d8 <phase_6+125>
```

In the last block, the code ends with getting the address of the needed node according to my input and store it in ```rdx``` so here it moves that address to the stack with ```rsi	``` (same index used to access my input) but with offset ```+0x20```. that's the address actually just after my last input stored. then increments the index and check if it get the max value and if not it will repeat. so this actually tell us that it will store the nodes to the stack according to my input, like if i entered the pattern for example this ```"2 5 4 6 1 2"``` then the node number 2 will stored first then the node number 5 will stored after it and so on. so it uses my input now to rearrange the nodes. Let's continue to see what's the purpose of this : 

```
   0x0000555555555906 <+171>:   mov    0x20(%rsp),%rbx
   0x000055555555590b <+176>:   mov    0x28(%rsp),%rax
   0x0000555555555910 <+181>:   mov    %rax,0x8(%rbx)
   0x0000555555555914 <+185>:   mov    0x30(%rsp),%rdx
   0x0000555555555919 <+190>:   mov    %rdx,0x8(%rax)
   0x000055555555591d <+194>:   mov    0x38(%rsp),%rax
   0x0000555555555922 <+199>:   mov    %rax,0x8(%rdx)
   0x0000555555555926 <+203>:   mov    0x40(%rsp),%rdx
   0x000055555555592b <+208>:   mov    %rdx,0x8(%rax)
   0x000055555555592f <+212>:   mov    0x48(%rsp),%rax
   0x0000555555555934 <+217>:   mov    %rax,0x8(%rdx)
   0x0000555555555938 <+221>:   movq   $0x0,0x8(%rax)
   0x0000555555555940 <+229>:   mov    $0x5,%ebp
   0x0000555555555945 <+234>:   jmp    0x555555555950 <phase_6+245>
```

You can see that it's like a pattern that's repeated. it gets the address to the first node stored in the stack then get the address to the next node stored and move the address of the second to the offset `+0x8` (which is actually the next pointer element) and repeat again with the second node and so on. so it just modifying the next pointer of the nodes stored in the stack to make them point to each other again like before. Now it's the time for the last block: 

```
   0x0000555555555947 <+236>:   mov    0x8(%rbx),%rbx
   0x000055555555594b <+240>:   sub    $0x1,%ebp
   0x000055555555594e <+243>:   je     0x555555555961 <phase_6+262>
   0x0000555555555950 <+245>:   mov    0x8(%rbx),%rax
   0x0000555555555954 <+249>:   mov    (%rax),%eax
   0x0000555555555956 <+251>:   cmp    %eax,(%rbx)
   0x0000555555555958 <+253>:   jge    0x555555555947 <phase_6+236>
   0x000055555555595a <+255>:   call   0x555555555be5 <explode_bomb>
   0x000055555555595f <+260>:   jmp    0x555555555947 <phase_6+236>
   0x0000555555555961 <+262>:   mov    0x58(%rsp),%rax
   0x0000555555555966 <+267>:   xor    %fs:0x28,%rax
   0x000055555555596f <+276>:   jne    0x55555555597e <phase_6+291>
   0x0000555555555971 <+278>:   add    $0x60,%rsp
   0x0000555555555975 <+282>:   pop    %rbx
   0x0000555555555976 <+283>:   pop    %rbp
   0x0000555555555977 <+284>:   pop    %r12
   0x0000555555555979 <+286>:   pop    %r13
   0x000055555555597b <+288>:   pop    %r14
   0x000055555555597d <+290>:   ret
```

We know that ```rbx``` stores the address to the newly rearranged nodes. so it takes the address to the next node from it and store it inside ```eax``` then compares between the  ```first elemnt```  for the first node and the next node. first node contains value ***less than*** the second node; it will blown up. and if not it will check for the second and the third node and so on. So the idea is just to rearrange the node in ***Ascending order***. Now let's manage our needs :

- we first need to enter 6 int number every one should be less than 6
- the elements should not be repeated
- and it has to rearrange the nodes according to it's value in Ascending order.

the first value for every node from 1 --> 6 is ```[0x212, 0x1c2, 0x215, 0x393, 0x3a7, 0x200]```

so according to so; the entered value should be `"5 4 3 1 6 2"`

let's try out :

[![5](/assets/images/reverseEngineerinig/phase6/5.png)](/assets/images/reverseEngineerinig/phase6/5.png)

**AND FINALLY**

*we are done with all phase. I hope you enjoyed the journey with me. if you saw any mistake I made or anything I said that wasn't clear; please Mail me :)*

***SEE YA IN NEXT CHALLENGES*** 
