# Phase 1 - Day 10: Saturday 11 April 2026
## Summary:
Planned a list of C projects to make to practice the language and some hacking/defending techniques that suit Phase 1!

## Documentation:
I have been busy since Day 9. Now I’m back on track! I reported back to Gemini and asked for the next step! It surprisingly gave me the “final” activity of Phase 1, which was a program that uses the stack and heap, and my challenge was to try to exploit the program using my research on Stack and Heap’s exploits and defenses.

I refused to do the activity now since I felt that that would be some sort of cheating, since Gemini gave me the full code, and told me the instructions and everything. If I just followed these instructions, this would look like cheating or taking a shortcut. So I told it that I want to practice C very well before doing this activity. And asked for a long list of projects that suits Phase 1, I don’t care how long, just a long one to practice well! So it gave me 35 projects divided into 5 categories. Using Google’s AI Mode, I added more to this list to include foundational stuff. So at the end, the list of projects that I’ll be doing looks like this:

**Category 0:** Foundational C Concept  
- **Pointers:** Understand that a pointer is just a number (an address).
- **Arrays and Pointer Arithmetic:** Understand that array[i] is just a fancy way of writing *(array + i).
- **The Heap:** Learn how to ask the OS for memory.
- **Strings:** Realize that strings in C are just arrays of bytes ending in 0x00 (null terminator).
- **Structs:** Grouping different types of data together.
- **Reading Files:** Getting data into your program.
- **Command Line Arguments:** Making tools, not just scripts.

**Category 1:** The pointers and Memory Architect  
**Goal:** Mastering the difference between the Stack and the Heap.  
1. **Custom malloc Wrapper:** Create a system that tracks every allocation and detects memory leaks before the program exits.
2. **Hex Dump Utility:** A program that takes a file or memory address and prints its contents in Hex and ASCII (like xxd or the hex view in x64dbg).
3. **The Struct Aligner:** A tool where you define various structs and it prints the offset of every member, revealing the "invisible" padding bytes.
4. **Generic Dynamic Array (Vector):** Recreate C++ std::vector in pure C using realloc.
5. **Manual Linked List (CRUD):** Build a library to create, read, update, and delete nodes without using global variables.
6. **XOR Linked List:** A memory-efficient linked list where each node only stores one pointer (the XOR of the previous and next pointers). This is great for pointer logic practice.
7. **Generic Hash Table:** Implement a hash table using open addressing and manual collision handling.
8. **Circular Buffer:** Create a fixed-size buffer used for streaming data (common in driver development).
9. **Binary Search Tree (BST):** Implement a tree that organizes data and allows for recursive searching.
10. **The Memory "Sweeper":** Write a function that scans a range of memory (the stack) for specific "magic numbers" or patterns

**Category 2:** The "Software Logic" Reconstructor  
**Goal:** Understanding how C constructs (loops, switches) turn into Assembly.  
1. **Switch-Case Simulator:** Build a program with a massive switch statement (20+ cases) to see how the compiler generates a "Jump Table" in assembly.
2. **Recursive vs. Iterative Factorial:** Compare the stack frames of both in a debugger to see "Stack Exhaustion" in action.
3. **Function Pointer Router:** A program that calls functions based on user input using an array of function pointers (simulates a VTable).
4. **Custom printf:** Reimplement printf from scratch using stdarg.h to understand how the stack handles variable arguments.
5. **Big Endian / Little Endian Converter:** A tool that swaps the byte order of a 32-bit and 64-bit integer.

**Category 3:** The "Pre-Exploitation" Lab  
**Goal:** Learning the bugs so you can learn the exploits later.  
1. **The Controlled Overflow:** Build a program that deliberately overflows a stack buffer to change the value of an adjacent integer variable.
2. **Integer Overflow Calculator:** A program that demonstrates what happens when you add 1 to 0xFFFFFFFF.
3. **The "Dangling" Pointer Simulator:** Create a Use-After-Free scenario and try to "hijack" the freed memory with a second object.
4. **Format String Vulnerability Lab:** A program that is vulnerable to %x and %n in printf (the classic way to read/write memory via strings).
5. **Off-By-One Error Demo:** A program that demonstrates how a loop that goes one step too far can corrupt the stack.

**Category 4:** The "System & Hardware" Emulator  
**Goal:** Understanding the Fetch-Decode-Execute cycle.  
1. **8-bit Virtual CPU:** Build a C program that "executes" its own custom bytecode (instruction set).
2. **The Bit-Wise Manipulator:** A tool that performs AND, OR, XOR, and Bit-Shifting to encrypt/decrypt strings.
3. **Manual Stack Machine:** A calculator that uses a push/pop stack to solve math problems (RPN).
4. **CPU Flag Simulator:** A program that mimics the EFLAGS register (Zero flag, Carry flag, etc.) based on math results.
5. **Memory Paging Simulator:** A simple script that translates "Virtual" addresses to "Physical" addresses using a mock Page Table.

**Category 5:** Practical Tools for Later Phases  
**Goal:** Building the tools you will actually use in Phase 2 and 3.  
1. **Base64 Encoder/Decoder:** Essential for obfuscating data in malware.
2. **Simple XOR Encryptor:** A tool to "pack" a file so it isn't readable by static analysis.
3. **String Obfuscator:** A program that takes a string and generates a C header where the string is hidden as a byte array.
4. **Process List (Snapshot):** Use basic C to list the names of all running processes (intro to Win32/Phase 2).
5. **Command Line Parser:** A robust argv parser that handles flags like -f, --file, etc.
6. **Checksum Generator (CRC32):** Build a tool that verifies file integrity.
7. **INI File Parser:** Read and write configuration files for your future tools.
8. **Logger Utility:** A thread-safe logging system that writes events to a file with timestamps.
9. **Shellcode Wrapper:** A program that allocates a buffer, marks it as executable (using VirtualAlloc), and jumps into it.
10. **The Final Foundation Boss:** Build a Static Library (.lib) that contains your Hash Table, Vector, and Logger, and use it in a separate Project.

I’m going to do my best to do every single project of these, from the first one to the last one in order, without cheating, taking shortcuts, or using AI.
