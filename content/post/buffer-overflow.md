---
title: "Buffer overflow analysis: stack_1"
date: 2024-10-25
draft: false
summary: "Exploring an executable through linux and windows, using IDA community edition"
---

# Buffer overflow analysis: stack_1

## Initial analysis

The readelf command is often the first step in analyzing an executable file, providing an overview of its sections, symbols, and structure. Running readelf on stack_1 hinted at the presence of several hidden flags within the binary, specifically suggesting that at least five flags could be found. Commands such as -S (section headers) and -h (file headers) were also tested, but these did not yield additional noteworthy insights.

![Checking the file stack_1 using readelf command](/images/buffer-overflow-stack1/1.jpg)

Utilizing the strings command allowed for the extraction of readable ASCII strings from the binary. This revealed explicit references to flags labeled from 1 to 5, clearly indicating multiple levels or checks within the application. Additionally, a username "Shirley" was discovered, prompting further investigation into her potential role or significance in the execution flow.

![Strings command](/images/buffer-overflow-stack1/2.jpg)

The objdump tool offered a detailed view of the binary’s assembly-level code, identifying specific memory addresses and code sections that warranted closer inspection. These addresses were noted for future reference, as they might relate to conditions checked for each flag.

![Objdump command](/images/buffer-overflow-stack1/3.jpg)

## Analysis with IDA community edition

At this point I had opened a copy of stack_1 in IDA, which showed different chains of locations. Some information could be seen there too, like Shirley. Another interesting thing was that on the same part that replied regarding Shirley, another option would be to test to compare a value and if not zero – it would jump to a new chain later promting good work flag 1 completed. 

The program prompts for a username, and then uses _strtok and _strcmp to process it. If it checks Shirley, it prints “ok Shirley, try to get the flags next”. If not, it prints “who is *whatever we inserted*”. 

If Shirley is entered as a input, then a series of checks are performed, where it evaluates the “flag” variable and other things. 
We do see multiple flags, flag_1 done, flag_2 done etc up until flag 5. 

Each flag seems to be checked by comparing a specific value or strings in memory.

### Flag 1 	
If the flag variable is zero, then it writes the Shirley message. If it is not zero, then it writes the flag_1 line. 

![Shirley check vs flag 1, IDA](/images/buffer-overflow-stack1/4.jpg)

From the local variables in the main function, we see what should be the location for the flag variable: 0x18.

![variable location flag 1, IDA](/images/buffer-overflow-stack1/9.jpg)

Opening stack_1 with GDB, setting a breakpoint at the location for the flag conditional check, and at that point we can inspect the variable as well as change it to 1 which we want to test to pass the check.

![Changing variable to pass conditional check, IDA](/images/buffer-overflow-stack1/10.jpg)

### Flag 2 	
After the flag_1 check, the program checks for a specific value in the flag variable, 0xDEADC0DE

![Flag2, IDA](/images/buffer-overflow-stack1/5.jpg)

Since we derived from looking at the disassembly earlier, the flag variable is also the one checked to find the flag_2 – using the deadcode hex instead of the 0 or 1. Testing by running the program again, on the same breakpoint. At that stop, I similarly as in flag_1 change to 0xDEADC0DE instead of 0. Continuing the program after that, reveals the flag_2 done string. 

![Flag2, GDB](/images/buffer-overflow-stack1/11.jpg)

As the green arrow indicates, also the third flag message is revealed at this point. 

### Flag 3
Seems to involve canary values, security measure against buffer overflow attacks. The value seems to be set to an ascii string “notabird”. 
The program checks if the canary is unchanged, and if it is intact flag_3 message is printed.

![Flag3, IDA](/images/buffer-overflow-stack1/6.jpg)

As the requirement for the flag_3 was that the canary value should be unchanged, and the methods used so far does not overwrite it, I run it again to inspect the local variable for canary which is stored in ebp-0x12. As it shows in the figure below, its still “notabird.” and presumably that’s it, at this part, we might need to overwrite it later – should be enough for the third flag. 

![Flag3 canary location and value, GDB](/images/buffer-overflow-stack1/12.jpg)

### Flag 4 	
In the same section as the canary above, there is a value 0xCAFECAFE that should refer to flag_4. After the canary passes, it checks a memory location ebp+4 to see if it holds the mentioned value.

![Flag4, IDA](/images/buffer-overflow-stack1/7.jpg)

For the fourth flag, as discussed above it compares a value at ebp+4 for CAFECAFE.

![Flag4, GDB](/images/buffer-overflow-stack1/13.jpg)

Testing to change the ebp+4 value, but quickly notice that the previous flags need to be changed as well due to it all running in a chain. 

![Flag4, GDB](/images/buffer-overflow-stack1/14.jpg)

Trying again with all flags set – and then it prompts the flag_4 message successfully:

![Flag1-4, GDB](/images/buffer-overflow-stack1/15.jpg)

### Flag 5
The last one (from what we see so far) has its own subroutine and seems to be a bit different than the others. The other flags are chained if we inspect the graph view of IDA, so it seems maybe it needs to have the previous flags correct and then some other values must be maybe overflowed.

![Flag4, IDA](/images/buffer-overflow-stack1/8.jpg)

The last flag seems more intricate than the previous ones. If I understand it correctly we would need to somehow overflow more, and then I am also guessing we are overflowing the canary which breaks the chain. If we can create an input that contains the canary at the right location that could be something we could try. From what I have read so far in the course, canaries should be created as random on each instance the program is run – so it is not predictable. However, this does not seem to be the case here? Running everything we have done so far again, and inspecting the canary, both as string and bytes:

![Flag5 testing, GDB](/images/buffer-overflow-stack1/16.jpg)

Additionally, I cant see a way it jumps to the flag_5 area, so it might be that we need to set the address that it should jump to manually. However, as seen in the image below while finding flag_5 I am overwriting flag_4:

![Flag5 found, GDB](/images/buffer-overflow-stack1/17.jpg)

### GDB scripts, briefly

Setting up .gdb scripts can make life a bit easier in these processes, where steps are repeated over and over again. In this test, I createed a .gdb file that I can pre-fill with the commands I want entered upon launch, as seen below in the flag1to4.gdb

![.gdb script](/images/buffer-overflow-stack1/18.jpg)

Then when running gdb with the -x option and naming the file, it loads everything ready up until the green line below.

![.gdb script in use](/images/buffer-overflow-stack1/19.jpg)


## Creating a python exploit script

Analyzing the code first, as I did not initially realize I had this and instead added letters one by one to the script till I got the segmentation fault error. 

![Analyzing the stack](/images/buffer-overflow-stack1/20.jpg)

    Possible stack:
    Buffer for username 20 bytes
    Canary 10 bytes
    Volatile (not to optimize/change) int paddy 4 bytes
    Flag variable (volatile) 4 bytes
    Base pointer 4 bytes
    Return address 4 bytes
    (higher addresses)

Total 46 (48 from testing with GDB)  bytes used above here, fgets reads 128, so we can recreate the required info within the input.

According to the variable sizes I found in the code, and similar to previously analyzed in IDA and GDB, the fgets function reads 128 bytes from user input in the username buffer – which is only 20 bytes long. This seems like the buffer overflow exploit possibility. 

The canary is not random but “notabird.” And initialized and set when the program is run from the #define CANARY. We can calculate the location of this, at least from the start of the input. 

    Username Shirley
    Flag 1 if the flag var is set to a non-zero value
    Flag 2 if the flag var is set to 0xdeadc0de
    Flag 3 canary check, if it still matches “notabird.”
    Flag 4 if the return address of main equals 0xcafecafe

Using msf-pattern_create, I created a pattern to test, so I can find more information about the buffer

![Pattern test](/images/buffer-overflow-stack1/21.jpg)

The program crashes at 0x41366241 location. Which we see the instruction pointer EIP is pointing at. If we focus on the instruction pointer, as we want to overwrite this with a return address of our chosing.

(gdb) run < <(python3 -c "print('A' * 45 + 'BBBB')") EIP: 0xf7000042 0xf7000042 B

(gdb) run < <(python3 -c "print('A' * 47 + 'BBBB')") EIP: 0x424242 0x424242 BBB

49 gives EIP 0x42424241 0x42424241 BBBA

48 seems to be fully overwriting EIP eip 0x42424242 0x42424242 BBBB

![Pattern test 2](/images/buffer-overflow-stack1/22.jpg)

So, 20(username)10(canary)4(paddy-padding)4(flag)4(basepointer)4(returnaddress). This only account for 46, maybe some kind of extra padding somewhere added by the compiler to make it align to 4 or 8 byte sections. First attempt fails due to Shirley-compare check gets incorrect due to I initially just padded the username with As:

![First exploit test](/images/buffer-overflow-stack1/23.jpg)

Instead, I added a null terminator between the username and the padding with A’s,  which successfully shows flag 1:

![Second exploit test](/images/buffer-overflow-stack1/24.jpg)

Played around with the ordering and if we check the code it suggests the order of information-pieces we need to insert into our buffer should be:
Username – padding to fill username – flag value (non zero (and deadc0de) – canary – paddy padding, padding for argc, padding for argv - return address

![Third exploit test](/images/buffer-overflow-stack1/25.jpg)

For the final 5th flag, simply switching out the return address for flag 4 to the flag 5 location, which we saw earlier using IDA.

![IDA flag 5 return address, IDA](/images/buffer-overflow-stack1/27.jpg)

![Fourth exploit test](/images/buffer-overflow-stack1/26.jpg)