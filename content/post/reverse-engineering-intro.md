---
title: "Reverse engineering, introduction labs"
date: 2024-11-09
draft: false
summary: "Reverse engineering using IDA and GDB focused on Linux elf binaries."
---

<img src="/images/reverse-engineering-intro/header.png"
     alt="IDA header image"
     width="900" />

# Introduction

This article presents an in-depth walkthrough of three foundational reverse engineering labs focused on analyzing Linux ELF binaries. The goal was to gain knowledge and insight in practical tools and mental models for understanding how low-level logic is implemented in compiled executables. These exercises both developed proficiency with the IDA disassembler and debuggers like GDB, but also help build an analytical mindset needed to infer behavior from indirect cues in binary form.

The progression of the labs—starting from simple integer comparison, advancing to hexadecimal parsing, and culminating in memory-based password reconstruction, gradually deepening the themes complexity. Each lab builds on the concepts introduced in the previous one while introducing a unique twist or obstacle requiring a new analysis strategy.

# First lab

## Initial Binary Analysis

The first step involves basic reconnaissance of the binary using native Linux tools. The file command identifies the initial binary as a 32-bit, dynamically linked ELF executable for the Intel architecture using little-endian byte ordering. The binary is also reported as "not stripped," which is crucial: it retains symbol names like function labels, making reverse engineering significantly more straightforward.

Following that, the xxd command provides a hexadecimal and ASCII dump of the binary, and strings reveals a list of embedded ASCII strings. These initial checks expose several informative details:

The presence of human-readable strings like check_password indicates the program likely performs password validation.

The use of integer formatting specifiers like %d confirms the nature of expected input.

A suspicious-looking string like ;*2$”H may suggest either garbage data or some form of obfuscation.

## Disassembly Walkthrough in IDA

The binary is loaded into IDA, where default settings are used except for several visualization improvements: enabling line prefixes, increasing the number of displayed opcodes per instruction, and activating auto-comments. This results in a more readable graph and disassembly view, especially useful for tracking stack variables and control flow.

<img src="/images/reverse-engineering-intro/1.jpg"
     alt="IDA Settings"
     width="400" />

### main function

In the main function, IDA highlights a 40-byte stack frame allocation. While argc and argv are present, they are unused, possibly a leftover from compilation templates or to conform with calling conventions, as I am not using command line arguments to these binaries. 

<img src="/images/reverse-engineering-intro/2.jpg"
     alt="Setting up stack"
     width="400" />

Initially variables are set, from var_18 to var_4, are likely used for storing the information needed in the program later. It sets up a new stack by saving the base pointer (ebp), and starting the stack at the current stack pointer (esp). The size of the stack is shown as 26h, where H denotes hexadecimal, which would be a total of 40 bytes. 

<img src="/images/reverse-engineering-intro/3.jpg"
     alt="main part 1"
     width="250" />
<img src="/images/reverse-engineering-intro/4.jpg"
     alt="main part 2"
     width="250" />

To prepare the password prompt, a label aInsertPassword (the string "Insert password") is loaded into edx and pushed onto the stack. The program then calls printf, displaying this prompt to the user.

After this, the program loads another label aD, representing the string format "%d". This format tells the scanf function to expect an integer. This string is also pushed onto the stack.

The key point here is how the program prepares the call to scanf. It pushes two arguments: the format string and the address where the user input will be stored. This address is resolved through the local variable var_8, which IDA denotes as [ebp+var_8]. At this point, the program is ready to receive user input.

Once the user inputs a value, it's stored in var_8, and then this value is moved to a register and pushed again. The program then calls the function check_password and passes the user input as its argument.

After the function call returns, the eax register is cleared using an XOR instruction (a common idiom for setting a register to zero), and the stack is cleaned up by restoring esp and ebp. The function returns control to the operating system, completing the main function. A brief summary:

    Stack allocation and addressing using ebp

    Preparing function arguments using push and registers

    The role of format strings and buffers in scanf

    Flow control using call, xor, and ret


### check_password function

After the main function collects user input and passes it into check_password, this function performs the actual validation logic. As with typical x86 calling conventions, the input value (an integer from scanf) is passed as the first argument on the stack.

<img src="/images/reverse-engineering-intro/5.jpg"
     alt="check_password function"
     width="250" />

After setting up a stack frame, allocating 8 bytes of local storage, the user input value is accessed from the stack using the offset [ebp+arg_0], which IDA recognizes as the argument passed into the function. This value is moved into eax for use in the comparison

    mov eax, [ebp+arg_0]
    cmp [ebp+arg_0], 539h

The comparison checks if the user-supplied value matches the hardcoded number 0x539, which is 1337 in decimal. If the values are not equal, the program jumps to a label (loc_80491A9) that handles incorrect input.

If the comparison succeeds, the function continues to the success branch. It loads the address of the string label aCorrect into eax, pushes it onto the stack, and calls printf to print the message “correct”.

After printing the success message, the function skips over the incorrect handler by jumping directly to the cleanup block at loc_80491B7.

Finally, the function concludes with stack cleanup. It restores the original esp position by adding back the 8 bytes previously subtracted, restores the ebp value, and returns execution back to main.

# Second lab

## Overview

The second lab builds upon the foundational logic introduced in Lab 01. While the program flow and structure are broadly similar, Lab 02 introduces additional complexity through format string parsing and hexadecimal interpretation. This lab shows a bit more of the subtleties of user input conversion, showcasing how data can be interpreted differently based on the functions used and their format specifiers.

Just like the first lab, the binary is a 32-bit dynamically linked ELF file with debugging symbols left intact (not stripped), allowing us to see named functions like main and check_password.

## IDA inspection and main function flow

I load the binary into IDA using the same preferred settings: enabling line prefixes, showing additional opcode bytes, and enabling auto-comments for better orientation. The main function structure and setup will look familiar by now.

<img src="/images/reverse-engineering-intro/6.jpg" alt="IDA view showing stack setup" width="250" />

The program begins by setting up a stack frame. As before, it subtracts space from esp—in this case, 40 bytes (0x28)—which is reserved for local variables. The variables initialized are similar in naming convention (e.g., var_4, var_8, etc.) and structure to the first lab. Their purpose becomes clearer as I analyze how values move through the registers and memory.

The printf function is used again to present a password prompt to the user:

    lea edx, aPassword
    mov [esp], edx
    call _printf

This instructs the program to print the static string “Password:”, loaded from the .rodata section of the binary. The format string is positioned on the stack and printf is called to output it.

Next comes a call to scanf, just like in the first lab. However, there's a difference: instead of collecting an integer directly, this version prepares to treat the input more like a string that will later be interpreted, unlike lab01 where the integer was compared directly.

<img src="/images/reverse-engineering-intro/7.jpg" alt="main preparing format string for scanf" width="250" />

At this point, the integer entered by the user is stored in var_8. The value is moved from var_8 into ecx, pushed onto the stack again, and then passed as an argument to the check_password function.

## check_password function

Once inside check_password, the logic change toward a more indirect form of comparison. Instead of checking the value directly like in Lab 01, the program performs an intermediate conversion using sscanf. The function starts by setting up an 0x18-byte stack frame, which is slightly larger than before, and defines three variables: arg_0 (user input from main), var_4, and var_8.

<img src="/images/reverse-engineering-intro/8.jpg" alt="check_password overview" width="900" />

Next, var_4 is cleared to make sure it has a clean state before conversion. Now I hit the core of this lab’s difference: the function uses sscanf to parse a hex string (%x) from the user’s decimal input.

<img src="/images/reverse-engineering-intro/9.jpg" alt="check_password entry and sscanf setup" width="900" />

    lea ecx, [ebp+var_4]
    mov [esp+8], ecx
    mov [esp+4], offset asc_804A00F  ; "%x"
    mov [esp], eax                   ; user input from main
    call _sscanf

Here, three arguments are prepared on the stack for sscanf, which converts the user input integer into a hexadecimal format, and stores the result in var_4.

    The string to read (from the user’s decimal input).

    The format specifier (%x)—telling the function to interpret the input as a hexadecimal.

    The destination address where the result should be stored (var_4).

Continuing, the program runs a compare vs a specific hexadecimal value 0xBEEF, and if the result from the sscanf converted value matches this the password is correct. Otherwise, the program jumps to a block where an “incorrect” message is printed.

# Lab three

## Overview

Lab 03 is a bigger step forward in terms of complexity. It changes from using static values or format-parsed input to a form of dynamic password generation through memory manipulation. While the setup phase in main is familiar by now, the behavior inside check_password introduces techniques like looping, array copying, and runtime buffer construction. This type of behavior is closer to what might be seen in obfuscated binaries or malware attempting to hide sensitive logic from direct static analysis.

The password is not visible as a constant and isn't compared directly in its original format. Instead, it is built piece by piece at runtime and then compared against the user’s input using a string comparison.

## IDA inspection and main function flow

As with previous labs, the binary is loaded in IDA with the same settings, giving a clean, organized disassembly view. The main function is again responsible for initializing the stack, printing a password prompt, and receiving the user’s input, this time as a string instead of an integer.

<img src="/images/reverse-engineering-intro/10.jpg" alt="main stack and input setup for lab03" width="250" />

The user input is read with

    scanf("%s", &var_40)

The program is using %s to capture string input. This hints that we’re moving into string comparison territory. The user’s string is stored in a memory location (var_40), which is then passed to check_password.

## check_password function

The bulk of the logics sits in the check_password function. It no longer compares integers or single values. Instead, it constructs a password by copying memory and manipulating it in a loop. Once the password is built in memory, the user’s input is compared to the built string using strcmp.

The function begins by setting up a stack frame of 100 bytes (0x64) and defining multiple local variables. As shown in previous labs, the user input is passed via arg_0 and is saved into a local buffer.

<img src="/images/reverse-engineering-intro/11.jpg" alt="check_password stack and memcpy in lab03" width="400" />

The first major operation is a memcpy call, which copies a fixed memory structure into var_40. This is a static structure embedded in the binary—likely the base for constructing the password. The copy is set for a total of 60 bytes (0x3C), and three arguments are passed:

    Destination: var_40

    Source: hardcoded structure in data section

    Length: 0x3C

This builds a raw buffer of values that are then interpreted or modified.

 <img src="/images/reverse-engineering-intro/12.jpg" alt="memset and structure prep" width="400" /> 

 Following memcpy, a memset call zeros out another memory region, var_50, which will hold the constructed password string. This prepares the destination buffer for clean construction without garbage data. 16 bytes are cleared here—likely the maximum password length expected.

 ### Password loop

The next portion of the function is a loop that executes 16 times. It uses a loop counter and a base index into var_40, multiplied by 4 each iteration (indicating word-aligned values). This looks like the function is calculating an offset inside var_40, moving that value into a register, and writing it into the corresponding index of var_50.

<img src="/images/reverse-engineering-intro/13.jpg" alt="loop building password string" width="900" /> 

This part is particularly interesting because it introduces indirect computation. Instead of comparing a fixed value, the program is now dynamically assembling the correct password from values embedded in the binary, but only visible at runtime.

The loop ends when the counter reaches 15, and the full password string is now ready in var_50.

### String comparison and validation

After the password has been built, the function performs a direct string comparison using strcmp

    strcmp(var_50, user_input)

If the comparison returns zero (meaning the strings match exactly), then the program prints the success message

    printf("correct")

<img src="/images/reverse-engineering-intro/14.jpg" alt="final comparison and branch" width="400" /> 


### Extracting the Password with GDB

Because the password is built dynamically in memory, I did not find a way for it to be discovered through IDA alone. The only way I could figure out to see what var_50 ends up containing is to inspect it at runtime.

To do this, I used GDB and set a breakpoint just before the strcmp call. Then I used the following command to dump the string in memory:

    display/s $ebp-0x50

This reads the password directly from the var_50 buffer in RAM, and reveals the correct string that the user must input. 

<img src="/images/reverse-engineering-intro/15.jpg" alt="gdb displaying constructed password" width="500" />

And finally

<img src="/images/reverse-engineering-intro/16.jpg" alt="gdb displaying constructed password" width="500" />