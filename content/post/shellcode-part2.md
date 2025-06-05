---
title: "Shellcode part 2"
date: 2024-12-15
draft: false
summary: "Notes and thoughts from an exercise testing shellcodes, building upon shellcode part 1"
---

# Shellcode Part 2

## Objective and scope
This exercise builds upon the basics established in Shellcode Part 1 by advancing to more realistic payload injection scenarios. The primary objective is to develop a shellcode that, when injected through three distinct methods (stdin, file, command-line argument), consistently executes and displays the string "TTC6520-3003!", followed by an exit with status code 0. The payload must also comply with typical shellcode constraints: namely, the exclusion of null-bytes (\x00) and efficient execution inside a constrained memory region.

As an initial test, I used the payload from part 1 using GDB. The purpose of this was to see how the program reacted to the input.

![Shellcode from part 1](/images/shellcode-part2/1.jpg)

I sat a breakpoint at the function vulnerable, break vulnerable. From the looks of it, the buffer is reset at the next line (17). 
Used next till I was at line 18, then inspecting the local buffer var with print &buffer as bytes which is what our payload is constructed by:

<img src="/images/shellcode-part2/2.jpg"
     alt="Shellcode from part 1, running through GDB"
     width="700" />

Next, I tried to find out how big the buffer needs to be before it overwrites the EIP in the program after the vulnerable function is done

<img src="/images/shellcode-part2/3.jpg"
     alt="Testing input of different sizes, to determine how long of a payload and sled I need"
     width="700" />

From the testing it seems buffer fills up the last part of the return address of the function vulnerable with 144 bytes

## Understanding the vulnerability
The shell_1.c file contains a vulnerable function named vulnerable(). This function allocates a 128-byte buffer and uses memcpy() to copy data from a user-supplied pointer (p) to the buffer. However, there is no bounds checking, meaning that a well-crafted input can overwrite memory beyond the allocated buffer, specifically targeting the saved return address on the stack.

## Stack frame explanation
When the function is called, the call instruction pushes the return address onto the stack. The function then sets up a new stack frame. Upon return, the CPU pops the saved return address from the stack into the EIP register. By overflowing the buffer and strategically placing a new return address pointing to a NOP-sled followed by shellcode, I redirect execution into our payload.

## Payload, first thoughts

The first strategy followed the following layout:

    NOP-sled: Allows some flexibility in the return address.

    Shellcode: Injected code to execute the desired command.

    Padding: Filler bytes to match expected buffer length.

    Return address: Overwrites the saved return pointer on the stack.

Initially, the payload size was limited to 144 bytes due to the buffer constraints in the vulnerable function.

<img src="/images/shellcode-part2/4.jpg"
     alt="More testing"
     width="500"/>

<img src="/images/shellcode-part2/5.jpg"
     alt="More testing"
     width="500" />

However, outside of GDB, the payload failed to work. This prompted a hypothesis that Address Space Layout Randomization (ASLR) might be interfering. Disabling ASLR was an essential step, as shellcode execution relies on predictable memory addresses.

Despite turning off ASLR, the shellcode still failed. Investigation revealed that the issue might lie in the small payload size and randomization in stack usage across different input methods.

## Determining the buffer address

Using GDB, the buffer address during vulnerable() was identified as approximately 0xffffbdd0.

From this, a target NOP-sled region was planned:

    Start: 0xffffbdd0

    End: 0xffffbe0c (with a 60-byte sled)

The goal was to place the return address somewhere within this range. However, as tests with multiple return addresses failed, a change in strategy was needed.

<img src="/images/shellcode-part2/6.jpg"
     alt="More testing"
     width="900" />

## Bigger payload

After further testing, the shellcode and NOP-sled were placed after the return address instead of before. This shift allowed for a significantly larger payload, closer to the 4096-byte buffer size used in the main application context.

This new payload strategy allowed the exploit to succeed when input was provided via stdin and file. However, the -c option (command-line argument) required a different layout. This led to in-depth analysis using GDB.

## GDB comparison between modes

I created a quick script that wrote a simple payload with A’s, B’s and a NOP-sled. Then I used the same file in gdb –args ./shell_1 for both -f option and -c option.

From the results of this, it seemed like the return address differ by about 0x100 (256 bytes), which could be explained by how the argv is being handled in -c mode. 
In both cases, the buffer length is the same so my return address in the payload can be at the same location. However, the actual address of the EIP from the vulnerable function seems to differ. 

Giving it a new go, comparing how it looks on the stack for both -c (left) and -f (right on the picture below)

<img src="/images/shellcode-part2/7.jpg"
     alt="More testing"
     width="900" />

The overlap of usable address ranges in both modes was:

Common NOP-sled zone: 0xffffbed0 to 0xffffbe20

This showed that although the payload is the same, the return address must be selected based on how the input is delivered. This is due to how the shell_1 binary handles arguments and buffers.

Continuing testing, I continued with the script adding a payload to attempt to cover both options:

<img src="/images/shellcode-part2/8.jpg"
     alt="More testing"
     width="900" />

And testing for both -c and -f side by side to easier visualize the differences.

<img src="/images/shellcode-part2/9.jpg"
     alt="More testing"
     width="900" />

Before starting any new lines, checking the stack to see addresses up there, and I see -c start very early with bb00 and -f is already at be

<img src="/images/shellcode-part2/10.jpg"
     alt="More testing"
     width="900" />

And at the end of the NOP sled, -c ends at be and -f ends at c1

<img src="/images/shellcode-part2/11.jpg"
     alt="More testing"
     width="900" />

## Dual payloads

Given the variation in stack behavior, two separate payloads were generated:

Payload A: Tuned for stdin/file input.

Payload B: Tuned for command-line input.

Both payloads were created using a Python script that:

    Constructs a large NOP-sled.

    Inserts the null-free shellcode.

    Adds padding.

    Appends the appropriate return address.

<img src="/images/shellcode-part2/12.jpg"
     alt="Dual payload, final script"
     width="900" />

Pseudocode:

     Define shellcode (pre-assembled payload to execute).
     Set buffer size to match the overflow threshold.
     Set specific return addresses for each payload type:
     - One for command-line input (-c)
     - One for file input (-f)
     Define NOP sled sizes:
     - Large sled for command-line payload
     - Smaller sled for file-based payload
     Create each payload:
     - Payload = 'A' * buffer + return address + NOP sled + shellcode
     Save both payloads to binary files:
     - payload_command.bin
     - payload_file.bin

With the dual payload setup, the script worked as intended for all options. 

<img src="/images/shellcode-part2/13.jpg"
     alt="Final script successful test"
     width="900" />

# Part 3: toward malicious shellcode

The next phase of the exercise introduces another change. Rather than simply printing a message, the goal is to build shellcode that executes arbitrary commands, such as spawning a shell or executing system binaries. This opens the door to command execution vulnerabilities, commonly exploited in real-world attacks.

For example, replacing the print+exit shellcode with:

    xor    eax,eax
    push   eax
    push   0x68732f2f ; // '//sh'
    push   0x6e69622f ; // '/bin'
    mov    ebx,esp
    mov    al,0xb
    xor    ecx,ecx
    xor    edx,edx
    int    0x80

Would allow execution of /bin//sh, launching an interactive shell. This type of payload is commonly used in reverse shells and privilege escalation exploits.

## Testing

In the Python script used to generate the attack input, the existing print-only shellcode is replaced with this more dangerous version. The payload is structured by first filling the buffer with "A" characters to reach the vulnerable offset, then appending a crafted return address, followed by a NOP sled, and finally appending the shellcode bytes. Two different payloads are created: one for use with file input, and another for command-line argument injection.

The custom assembly (part2_payload.asm) is written to call the execve system call, which executes a new process. In this case, the process is a shell. The code first zeroes out eax, then pushes the string "/bin//sh" onto the stack. Registers are assigned to point to this location, and the syscall number for execve (which is 11) is loaded into eax. Finally, int 0x80 is used to execute the syscall. This results in a new shell being spawned if successful.

<img src="/images/shellcode-part2/14.jpg"
     alt="Part 3 payload asm script"
     width="900" />

Testing the payload confirms there are no null-bytes and I can proceed.

<img src="/images/shellcode-part2/15.jpg"
     alt="Part 3 shellcode"
     width="900" />

Replacing the previous shellcode with the new more malicious one.
<img src="/images/shellcode-part2/16.jpg"
     alt="Part 3 python script"
     width="900" />

<img src="/images/shellcode-part2/17.jpg"
     alt="Shell opened"
     width="900" />

Once the payload is written and the vulnerable program (shell_1) is executed using the file-based payload, the shellcode is executed. This is confirmed by running commands such as whoami and pwd inside the newly spawned shell. The output of these commands verifies that the shell is active and running with the same privileges as the vulnerable program.

In particular, this final test shows the practical impact of buffer overflow vulnerabilities, especially when mitigations like stack canaries or non-executable stacks are absent or weak. It transitions the exercise from a theoretical flag-capturing task to a real-world example of arbitrary code execution using custom shellcode.

