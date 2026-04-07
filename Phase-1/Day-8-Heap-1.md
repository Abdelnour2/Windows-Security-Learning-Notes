# Day 8: Sunday 5 April 2026
## Summary:
Started my research on the Heap.

## Note:
Documenation first, Notes on the bottom.

## Documentation:
Today, I’m finally starting my research on the Heap. I started with this [video](https://www.youtube.com/watch?v=ep2xOW52mDY), the final 2 minutes were about the Heap. I learned that the Heap is slower and more complex to work with compared to the Stack. But it allows us to create more sophisticated and more complicated data structures. With help of Google’s AI Mode, I identified the data structures used in both the Stack and the Heap. Learnings are noted in the Notes Section.

After this, I used Google’s AI Mode to get an idea of what to research next, and this is what I need to research:
- Segment Heap vs. NT Heap
- Frontend vs. Backend Allocator
- Heap Structure
- How does the OS keep track of free memory? FreeLists, Lookaside Lists, and the heap merges small free blocks into larger ones
- Allocation Algorithms: what happens when I call functions like malloc, HeapAlloc, …
- **Heap Exploitations:**
  - Heap Overflow
  - Use-After-Free
  - Double Free
  - Type Confusion
  - Heap Grooming
- **Defenses:**
  - Heap Metadata Encryption
  - Guard Pages and User-Mode Stack Trace
  - Arbitrary Code Guard (ACG) and Control Flow Guard (CFG)

## Notes:
### Topic: Heap

The Heap is slower and more complex to work with compared to the Stack, but it allows us to create more sophisticated and more complicated data structures.

In the Heap, we need to dynamically allocate the memory and grab new resources from the OS. And it is possible for programmers to forget to free that memory (in languages like C and C++) which leads to memory leaks.

We use the Heap because sometimes we don’t know the size we need for an object ahead of time.

So the Stack is used when a size is known ahead of time and the return value exists within one function. While with the Heap, we use it when the size is unknown ahead of time or the return value isn’t limited to one function.

The Data Structures that can be placed on the Stack are Primitives (int, float, double, bool, and char), Fixed-Size Arrays, C Fixed-Size Strings (Since they are fixed-size arrays of char), and structs.

Data Structures that can be placed on the Heap are Dynamic Arrays, Modern Strings and Lists, LinkedLists, Trees, Graphs, Hash Tables, Maps, Sets, Class Objects, …
