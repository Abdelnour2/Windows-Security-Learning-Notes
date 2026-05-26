# Phase 1 - Day 22: Saturday 16 May 2026
## Summary:
Made the 10th and final project of Category 1 which is a Memory Sweeper!

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Today’s project is the final one in Category 1, which is about a Memory Sweeper. In this project, I’m going to make a function that scans the memory of the program and searches for a specific value. To make the function more versatile, I’m going to allow the user to specify a range for the scan, so that it doesn’t scan the entire memory all the time.

The way the function will work is to first define a temporary variable just to start somewhere on the stack. Since the function will be called after the variable is initialized, then this temp variable is below the variable on the stack since the stack goes downwards. So searching for the stack requires us to search upwards! This way, the function will get the pointer of the temp variable, then loops upward and checks each address for the value we want.

Intentionally I was planning to use int, but I learnt recently that a safer approach is to use uint, like uint32_t since it keeps everything structured and allows the program to work on all machines. Whereas int is platform/OS dependent, sometimes it’s 2 bytes, sometimes it’s 4, ..

**Reflection:**  
This was a very small and simple project to make! But it was a fun experience to practice on how the stack actually works.

## Full Code:
```c
#include <stdio.h>
#include <stdint.h>

void MemoryScanner(uint32_t numberToFind, size_t range);

int main(int argc, char **argv) {
    uint32_t number = 0x01020304;

    MemoryScanner(number, 60);

    return 0;
}

void MemoryScanner(uint32_t numberToFind, size_t range) {
    uint32_t startingPoint = 0;
    uint32_t *startingPointPointer = &startingPoint;

    printf("Starting the search from %p:\n\n", startingPointPointer);

    for (size_t i = 0; i < range; i++) {
        if (*startingPointPointer == numberToFind) {
            printf("Number Found!\nAddress: %p\nValue: 0x%X\n", startingPointPointer, *startingPointPointer);

            return;
        }

        startingPointPointer--;
    }

    printf("Not Found!\n\n");
}
```