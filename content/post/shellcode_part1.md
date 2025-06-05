---
title: "Shellcode part 1"
date: 2024-12-13
draft: false
summary: "Notes and thoughts from an exercise testing shellcodes"
---

# Shellcode part 1

## Introduction to Shellcode

Shellcode typically refers to a small piece of executable machine code that is used as the payload in the exploitation of software vulnerabilities, often buffer overflows. Unlike regular executables, shellcode is raw machine code designed to be injected into vulnerable processes to execute arbitrary instructions. Importantly, it must be carefully constructed to avoid null bytes (\x00) because these bytes terminate strings in many programming languages, preventing complete injection.

## Provided Files and Initial Examination

![Provided files for the exercise](/images/shellcode-part1/1.jpg)

For this shellcode exercise, various files were provided to illustrate unsafe programming practices and shellcode testing methodologies:

Shell_1.c: Demonstrates unsafe input handling by copying data into buffers without bounds checking, indicating a vulnerability to buffer overflow.

shell_stub.c: Shows how shellcode execution is set up using function pointers. It includes a breakpoint trap (0xcc) for debugging purposes.

Shelltest.c: Analyzes shellcode specifically looking for null bytes (\x00). It prints the shellcode in raw and hexadecimal forms, confirming its validity.

Assembly files (exit.asm and hello.asm): Contain basic system calls illustrating how shellcode interacts with the kernel. For example, hello.asm writes "Hello, World" to standard output and then exits gracefully.

## Understanding System Calls and Registers

In Linux, system calls are invoked using interrupt int 0x80, with parameters passed via specific CPU registers:

EAX: Specifies the system call number (e.g., 1 for exit, 4 for write).

EBX, ECX, EDX: Used for the first, second, and third arguments respectively.

Registers are manipulated using assembly instructions like mov.

![Code snippet](/images/shellcode-part1/2.jpg)

## Exercise Objective

The exercise involves creating shellcode that prints the string "TTC6520-3003!", and then exits with a status code of 0.
Additionally, it can also not contain any null bytes (\x00)

For a test, I create a simple program in asm:

![First asmx86 code test](/images/shellcode-part1/3.jpg)

Doing this in this way clearly did not work, and contains a lot of nullbytes. Need to refine our approach. Additionally, so far I have created a self running program – and not a shellcode to be injected into another program. 

![First asmx86 code test, contains null-bytes](/images/shellcode-part1/4.jpg)

When we have a shellcode, often it is injected into a process as a string, for example using functions like strcpy(). Many of these functions will terminate if it comes across a null byte, hence for the shellcode to survive it cant contain any. 
Shellcode vs. executable – shellcode is raw machine code (not an elf executable). This could be extracted from the .text section of my assembled object file and used directly.  

We need to remove the null bytes, first idea is to manually use methods such as xor’ing registers, using the lower bytes of registers, using short jmps or somehow calling and placing our values on the stack without adding any 0s.

![Second asmx86 code test, contains null-bytes](/images/shellcode-part1/5.jpg)

Using the above asm program, I compile it and prepare to test it using the provided test files, checking for null bytes and creating an array.

![Compile and prepare the script](/images/shellcode-part1/6.jpg)

![First asmx86 code test, should contain no null-bytes](/images/shellcode-part1/7.jpg)

Using the provided shelltest, I run it against the shellcode I created, and seeing there is no null-bytes hence this should be ok.

![testing the shellcode, if it contains nullbytes](/images/shellcode-part1/8.jpg)

To easily format the shellcode into an array, that I can use for the final file replacing the original one (that contains null-bytes and dont work), I run the shell_array.sh script.

![Creating the new array of shellcode](/images/shellcode-part1/9.jpg)

Then replacing the example array in shell_array.sh with the above array from my own shellcode. Deleting the old shell_stub file and running make again gives the following:

![Creating the new array of shellcode](/images/shellcode-part1/10.jpg)

And by that we managed to create a piece of shellcode that did not have any null-bytes, and follows the criteria wanted. A fun introduction!

# References

Erickson, Jon. Hacking: The Art of Exploitation. 2nd ed., No Starch Press, 2008.