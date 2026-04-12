# Phase 1 - Day 1: Wednesday 25 March 2026
## Summary:
- Task 1 and 2 were about cracking a Hello World program
- Task 3 was me learning and summarizing the MS x64 Calling Convention found on this [page](https://learn.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-170).

## Task 1:
The first task I had was to make a Hello World program in C, and finding the “Hello World” string in x64dbg. So I made the program:
#include <stdio.h>

```c
int main() {
    printf("Hello World!");
    
    return 0;
}
```

opened x64dbg, imported the .exe and used Search for –> All Modules –> String References, and searched “Hello World” and found the instruction!

## Feedback:
As a feedback from Gemini, It told me that I can’t always rely on the search feature since some programs encrypt their strings, or build them on the fly to hide them. So I need to rely on the assembly instruction to catch the string. It told me that this kind of instruction usually look like this:
lea rcx, qword ptr ds:[<address_of_string>]
call <JMP.&printf>

Which is true, these are the instructions for me:  
lea rax,qword ptr ds:[7FF6A957B050]  
mov rcx,rax  
call main.7FF6A9572E10  

Gemini explained further the reason for using RCX specifically, it said that in Windows x64 development, we use Microsoft x64 Calling Convention. When a function like printf is called, the CPU doesn’t know what to print, we have to put the arguments into specific registers before the call.

And it explained that:  
- RCX: 1st argument
- RDX: 2nd argument
- R8: 3rd argument
- R9: 4th argument
- Stack: 5th argument and beyond

## Task 2:
As a second task, I was asked to manipulate the “Hello World” instruction so that it prints something different!

What I did is that I set a breakpoint on this instruction: lea rax,qword ptr ds:[7FF6A957B050]. After the breakpoint, I stepped into one more time to finish executing this instruction. Then I selected RAX and selected “Follow in Dump” to see the HEX values of it. And changed these HEX values to be “48 61 63 6B 65 64” which means “Hacked”.. After that, I continued the execution and got “Hacked” printed instead of “Hello World”.

## Feedback:
After telling Gemini about these instructions:
lea rax,qword ptr ds:[7FF6A957B050]
mov rcx,rax
call main.7FF6A9572E10

It told me that this is typical of “unoptimized” or “Debug” builds. A compiler in “Release” mode would skip the middleman and go straight to lea rcx, [addr].

## Task 3:
After Task 2, Gemini gave me another task, but I delayed it to do something else! I got curious about the “Microsoft x64 Calling Convention” that Gemini told me about! So I decided to dive into it.

### Topic Microsoft x64 Calling Convention ([Page](https://learn.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-170))

**Caller:** The function (or code) that initiates the call. It’s responsible for setting up the arguments and invoking the function. For example, if main() calls printf(), then main() is the caller.

**Callee:** The function that receives the call. It executes its body using the arguments provided by the caller. Following the previous example, printf() is the callee.

The x64 Application Binary Interface (ABI) uses a four-register, fast-call calling convention by default. Space is allocated on the call stack as a shadow store for callees to save those registers.

**Shadow Space:**  
- **Definition:** Sometimes called Home Space, refers to a reserved area on the stack that the caller must allocate before making a function call. It is always present, even if a function takes no arguments. Each function call has its own shadow space within its stack frame. When the function ends, its stack frame and the shadow space will be discarded, allowing for a stack frame and shadow space to be created in its place on a new call.
- **Purpose:** Provides a storage for the first four arguments that are passed in registers. Even though those arguments are in registers, the stack space is reserved so the callee can spill (save) them if needed.
- **Size:** It’s always 32 bytes (enough for four 8-byte slots). This space is allocated by the caller before the call, regardless of how many arguments are actually passed.
- **Extra Arguments:** If a function takes more than four arguments, the additional ones are placed on the stack immediately after the shadow space.

Any argument that doesn’t fit in 8 bytes, or isn’t 1, 2, 4, or 8 bytes, must be passed by reference. A single argument is never spread across multiple registers.

In the past, there was a x87 register stack used to store floating points. The x87 floating-point unit (FPU) had an 8-register stack architecture (ST0 - ST7). The stack is still present, but it is not part of the calling convention anymore, so we shouldn’t use it, and instead, all floating points must be operated using the same ABI structure using the 16 XMM registers.
- **Integers:** Stored in RCX, RDX, R8, and R9 registers. More than these goes to the stack.  
- **Floating Points:** Stored in XMM0, XMM1, XMM2, XMM3 registers. More than these goes to the stack.  
- **Volatile Registers:** These are RAX, RCX, RDX, R8, R9, R10, R11, XMM0, XMM1, XMM2, XMM3, XMM4, and XMM5. These also can be used by the caller. Their value is not important so they can be overridden as much as we want. But their value is not guaranteed to survive after a function call. So if the program needs these values after the call, they must be stored somewhere safe before the function call ends.  
- **Non-Volatile Registers:** These are RBX, RBP, RDI, RSI, R12, R13, R14, R15, XMM6, XMM7, XMM8, XMM9, XMM10, XMM11, XMM12, XMM13, XMM14, and XMM15. These can also be used by the caller. But their value is essential, so before using them, we need to store their original values somewhere safe, and return them after finishing.  

XMM registers are 128-bit long. They can store:
- One 128-bit vector
- Two 64-bit doubles
- Four 32-bit floats
- Eight 16-bit integers

Strings are also stored here since they are represented as pointers to character arrays. For example: printf(“%s”, “Hello”); %s itself is a string passed as a pointer, and Hello is also a string passed as a pointer. So RCX will have %s, and RDX will have Hello.

**Varargs (Variable Arguments):**  
These are functions where the number and types of arguments can vary. Like printf for example. Floating points need to be duplicated and stored in both the normal registers and the XMM registers. For example: printf(“%f”, 3.14); 3.14 will be stored in both RCX and XMM0.
- XMM0 because that’s where floating points normally belong.
- RCX because in Varargs and Unprotected Functions, floating points need to be replicated.

The duplication ensures that the callee can interpret arguments correctly whether it expects floats or integers. So in this example:
- RCX = 3.14 (bit pattern of the double)
- XMM0 = 3.14

If we had more floats they’d go into XMM1/RDX, XMM2/R8, XMM3/R9 and so on.

**Unprototyped Functions:**  
These are functions declared without a prototype in C. For example: void foo(); When we call foo(42, 3.14), the compiler doesn’t know what types foo expects. The ABI requires arguments to be converted to the callee’s expected types before passing. The caller must still allocate shadow space for four register arguments, even if the callee doesn’t take that many. This rule ensures that even without a prototype, the caller can safely access arguments.

**Alignment:**  
- **Definition:** Alignment is how data is arranged in memory so that it starts at addresses that are multiples of its size. For example, a 4-byte integer is usually stored at an address divisible by 4, an 8‑byte double at an address divisible by 8. This makes access faster because the CPU can fetch aligned data in fewer cycles.
- **Stack Pointer:** The stack pointer (RSP) is always kept 16-byte aligned before a function call because XMM registers often operate on 16-byte chunks. If the stack weren’t aligned, instructions like movdqa (aligned load) would fail or be slower.
- **Dynamic Allocations:** Memory allocated with malloc or alloca is also guaranteed to be 16-byte aligned.
- **Above 16 Bytes:** Any alignments above 16-byte must be done manually.
- **Why 16 Bytes:** Matches the width of XMM registers, making SIMD/Floating Point operations efficient.

**Unwindability:**  
This is about how the system can unwind the stack when handling exceptions.

**Leaf Functions:** These are functions that don’t change any non-volatile registers. For example, a function that just returns a constant. Since they don’t touch the stack or non-volatile registers, they can be unwound simply by simulating  a return. No extra metadata is needed.

**Non-Leaf Functions:** These are functions that do change non-volatile registers or adjust the stack pointer. For example, a function that calls another function, or allocates local variables on the stack. These need extra metadata so the system knows how to restore registers and stack state during exception handling.

**Unwind Data:**  
- **pdata (Procedure Data):** Static metadata attached to each non-leaf function.
- **xdata (Exception Handling Data):** Contains the actual instructions for how to unwind. For example, how much stack space was allocated, which registers were saved, and where, …
- During an exception, the runtime uses pdata then xdata to walk back through the call stack and restore the program state correctly.

**Prologs and Epilogs:**  
- **Prolog:** The setup code at the start of a function (saving registers, adjusting RSP)
- **Epilog:** The cleanup code at the end (restoring registers, resetting RSP, returning)
- These are highly restricted in x64 so they can be described precisely in xdata.
- The stack pointer must remain 16-byte aligned everywhere except inside prologs/epilogs or in leaf functions.

The compiler does all this automatically, so the programmer doesn't need to worry about writing any of these unless he’s writing pure ASM code.

**Return Values:**  
- **Scalar Return Values:** If the return value fits into 64 bits (like int, long, __int64, or __m64), it’s returned in RAX.
- **Floating Point and Vector Return Values:** Floats, doubles, and vector types (__m128, __m128i, __m128d) are returned in XMM0.
- **User-Defined Types:** Two cases:
  - **Returned by Value in RAX:** Allowed only if the struct/union is small (<=64 bits) and meets strict requirements:
    - No constructors, destructors, or copy assignment operators.
    - No private/protected non‑static members.
    - No reference members.
    - No base classes or virtual functions.
    - Only POD‑like members (plain old data).
  - **Returned via Pointer (Caller-Allocated):**
    - If the struct/union is larger than 64 bits or doesn’t meet the POD rules, the caller must allocate memory for the return value.
    - The caller passes a pointer to that memory in RCX.
    - The callee writes the result into that memory and returns the same pointer in RAX.

**Function Pointers:**  
A function pointer is simply a pointer to the function’s entry point (its label).

**Floating-Point Support for Older Code:**  
Legacy MMX (MM0 - MM7) and x87 stack registers (ST0 - ST7) are not used anymore and strictly prohibited in Kernel Mode Development. We use XMM, YMM and ZMM registers instead now.

**FPCSR (Floating Point Control and Status Register):**  
This is the control register for the old x87 floating-point unit.
It holds settings like:
- **Exception masks:** whether floating‑point exceptions are ignored or not.
- **Precision control:** single, double, or extended precision.
- **Rounding mode:** round to nearest, round down, etc.
In the x64 ABI, FPCSR is non‑volatile, meaning that if a function changes it, it must restore it before returning.
Default values at program start:
- All exceptions masked
- Double precision
- Round to nearest

**MXCSR (SSE Control and Status Register):**  
This is the control register for SSE/XMM instructions (modern floating‑point and SIMD).
It contains both status flags and control flags:
- **Volatile part (bits 0 to 5):** status flags (like exception flags). These can change freely and aren’t guaranteed across calls.
- **Non‑volatile part (bits 6 to 15):** control settings (like rounding mode, exception masks). If a function changes these, it must restore them before returning.
Default values at program start:
- Exceptions masked
- Round to nearest
- Denormals = normal (not flushed to zero)
- Flush‑to‑zero = off

FPCSR is the old rules of floating-point math, which is rarely used today. MXCSR is the modern rules.

**setjmp and longjmp:**
- These are functions that allow saving and restoring execution states
- **setjmp:** Saves the current stack pointer, non-volatile registers, and MXCSR.
- **longjmp:** Restores them to the state saved by the most recent setjmp.
- **Difference from x86:** In x64, destructors and __finally clauses are invoked during unwind.
