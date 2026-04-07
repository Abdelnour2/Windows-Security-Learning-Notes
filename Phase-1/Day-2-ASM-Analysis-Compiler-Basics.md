# Day 2: Thursday 26 March 2026
## Summary:
**- Task 1:** Fixing the building configuration on my VS Code
**- Task 2:** Researching about Debug and Release Builds
**- Task 3:** Researching about the Compiler and Linker
**- Task 4:** Analyzing a piece of ASM code
**- Task 5:** Observing JMP and CMP ASM instructions in a simple password C program

## Note:
Documentation of the Tasks is at the beginning while the notes will be at the bottom!

## Task 1:
First thing I did is check VS Code to see if it’s making a “Debug” or a “Release” build. I’m used to making my builds by simply pressing the run button.

When I run the program, VS Code executes this sequential command: clear ; if ($?) { cd "current path" } ; if ($?) { gcc main.c -o main } ; if ($?) { .\main }

This came from Code Runner:
"c": "clear && cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",

“gcc main.c -o main” is a basic command. There is no optimization specified so it uses -O0 default. And since the -g tag isn’t here so it is not a Debug build. So basically this is a basic build. Not optimized but also doesn’t contain the map for debugging.

To make the command better, I’ll use this “gcc -O3 -DNDEBUG main.c -o main” where:
- -O3 is the highest level of optimization
- -DNDEBUG means “No Debug” and activates a C macro (called NDEBUG) which disables some things to further increase the speed.

So finally, I updated my Code Runner configuration, rebuilt the program, and tested it in x64dbg! I noted my learnings in the Notes section.

# Task 2:
After encountering the terms “Release” and “Debug” modes. I want to research them to get a better idea of what they are! I noted my learnings in the Notes section.

# Task 3:
When I was researching the two kinds of builds, I ran into words like “compiler” and “linker”. So I researched those, and noted my learnings in the Notes section.

# Task 4:
After the research, I’ve returned to the original task Gemini gave me yesterday. The task was to identify the meaning of the first ASM instructions in a program:
push rbp
mov rsp, rbp
sub rsp, 30

At first, I didn’t know what it fully meant but I was sure that it has something to do with the stack since RSP is basically the Stack Pointer register. After digging, I realized that this sequence is the “Prolog” I studied in the Microsoft x64 Calling Convention.

**Feedback:**
My conclusion was correct that this is the Prolog, but Gemini added something important. He pointed that the program is subtracting 30 bytes in hex (48 bytes in decimal), why is that?

The Shadow space I learnt is always 32 bytes decimal (20 bytes in hex). He told me that what happened here is simply because of the Alignment rule (16 bytes alignment). Before calling the function, the stack was aligned perfectly. Calling the function added 8 bytes in decimal. Now the stack is not aligned. So the program pushes rbp which is another 8 bytes in decimal, that’s 16 bytes in decimal in total. Now the stack is aligned again. So finally, 32 bytes of shadow space, and the 16 alignment bytes adds up to 48 bytes in decimal (30 in hex).

**Task 5:**
After the last task, Gemini told me that in C, we use if, else, and loops. But the CPU only knows “Compare” and “Jump”. So he gave me a task to make a simple password protected program and see how it looks in ASM, and try to crack it.

So I made the program:
#include <stdio.h>

```c
int main() {
    int password = 1337;

    int user_input = 0;
    printf("Enter Password: ");

    scanf("%d", &user_input);

    if (user_input == password) {
        printf("Correct!\n");
    }
    else { 
        printf("Wrong!\n");
    }

    printf("Press Any Key to Exit..");

    getchar();
    getchar();

    return 0;
}
```

Built it as a “Release” build with full optimization, imported it into x64Dbg, and started analyzing the assembly.

After locating the main function, I immediately identified the assembly instructions and connected them to the C code:
lea rcx, … –> This is the Enter password string
mov dword ptr ss:[rsp+2C], 0 –> Initializing the user input to 0.
call … –> calling printf
lea rdx, qword ptr ss:[rsp+2C] –> Putting the user input in rdx
lea rcx, … –> Putting %d in rcx.
call … –> calling scanf
cmp dword ptr ss:[rsp+2C], 539 –> comparing the user input to 539 in hex (1337 in decimal)
je … –> if they are equal, jump to the specified location
lea rcx, … –> putting “Wrong!” in rcx
call … –> call printf
…

Another thing I noticed here is that learning the Microsoft x64 Calling Convention really helped me here since I immediately identified that the program is filling arguments when using rcx and rdx.

There are many ways to bypass the protection, either use the discovered 1337 password, or change the instruction of je to jmp to always be correct, or change the flag that came from cmp to always be correct to always trigger the je instruction.


## Notes:
### Topic: Building C Programs

To build a C program using GCC:
- -o: Output symbol, after it, we name the output file (.exe file)
  - gcc main.c -o main
- -g: Debug symbol, it tells gcc to make a “Debug” build
  - gcc -g main.c -o main
- -Ox: Optimization Level symbol, -O0 is the lowest level and means no optimization. -O3 is the highest and means maximum optimization.
  - gcc -O3 main.c -o main
-DNDEBUG: No Debug symbol, it activates a C macro (NDEBUG) that disables some debugging functionalities to make the speed of the program faster.
gcc -O3 -DNDEBUG main.c -o main

### Topic: The Difference Between Debug and Release Builds

**Debug Build:**
It is designed for developers. Its primary goal is to make the relationship between the source code and machine code as clear as possible so that they can find and fix bugs.
**- No Optimization:** The compiler is told to translate the code literally (often using flags like -O0). It doesn’t rearrange lines or skip steps, so when the developer steps through with a debugger, the instruction pointer matches exactly what they see in the .c file.
**- Symbolic Information:** It includes “debug symbols” (via flags like -g). These maps tell the debugger which memory address corresponds to which variable name or function in the source code.
**- Extra Safety Checks:** Many compilers add “canaries” or specific patterns (like 0xCC) to uninitialized memory to help detect buffer overflows and uninitialized variables.
**- Diagnostic Macros:** The preprocessor typically defines a _DEBUG macro. This enables features like assert(), which will halt the program if a condition isn’t met.

**Release Build:**
It is designed for end-users or the production environment. Its goal is maximum efficiency, speed, and minimal size.
**- Heavy Optimization:** The compiler uses aggressive flags (like -O2 or -O3) to rewrite the code for better performance. It might perform “inlining” which means replacing function calls with the actual code, or “loop unrolling” to save time.
**- No Debug Symbols:** These are stripped out to keep the binary file small and to make reverse engineering more difficult.
**- Disabled Checks:** To save CPU cycles, the NDEBUG macro is usually defined, which causes assert() statements to be completely ignored by the compiler.
**- Heisenbugs:** Because the compiler rearranges code for speed, a bug that was hidden in Debug (like a race condition) might suddenly appear in Release because execution timing has changed.

### Topic: What is a “Compiler” and a “Linker”

**Compiler:**
The compiler acts as the primary translator, taking individual C source files and converting them into machine-readable object code (.o files). It focuses on one file at a time, checking the syntax for errors and ensuring that every variable and function follows the rules of the language. However, the compiler is "short-sighted"; if the code calls a function located in a different file, the compiler simply leaves a placeholder or basically a note saying that the actual address for that function will be provided later. Its output is an intermediate object file that contains binary instructions but isn't yet a complete, runnable program.

**Linker:**
The linker is the final assembly stage that gathers all those individual object files and weaves them into a single, cohesive executable. It behaves like a master coordinator, scanning the "notes" left by the compiler to resolve memory addresses and connect function calls to their actual definitions across different files or external libraries. If the compiler ensures the grammar is correct, the linker ensures all the pieces of the puzzle actually exist and fit together. When the linker finishes its job, it produces the final file, like an .exe or a .out, that the operating system can finally load and run.
