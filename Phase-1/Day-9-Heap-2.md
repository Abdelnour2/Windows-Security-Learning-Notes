# Day 9: Monday 6 April 2026
## Summary:
Finished my research on the Heap.

## Note:
Documentation first, Notes at the bottom.

## Documentation:
For today, I’ll try my best to complete the Heap research! All the learnings will be in the Notes Section.

**Update:**  
I was able to finish the whole research today! I’ve used Google and Google’s AI Mode since YouTube was empty regarding these deep topics.

With that, my “Research Days” are done for now. Tomorrow I’ll report everything (from Day 3 to Day 9) to Gemini to continue the plan. Maybe I’ll focus on programming or researching C in the upcoming days. I’ll see!

## Notes:
### Topic: Heap (Cont.)

**NT Heap:**  
NT Heap (often called the “Windows Heap”) was the standard for decades. It was built for efficiency in an era where memory was scarce. It uses a Backend Allocator to manage large chunks of memory and Frontend Allocator to handle frequent, small allocations quickly. It relies heavily on FreeLists and Lookaside Lists. Because it uses complex linked lists for metadata, it is highly susceptible to Heap Overflow.

**Segment Heap:**  
Introduced in Windows 10. The Segment Heap is a total redesign. It is now the default for modern browsers and system processes. It focuses on security and scalability for modern multi-core processors. It moves away from the "everything is a linked list” approach to reduce metadata corruption risks. It organizes memory into segments instead of just one giant pool, it uses different managers for different sizes: Variable Size Slots for small objects and Large Allocations for massive ones. It uses Page Provider to grab memory from the OS in 4 KB chunks (pages) rather than large, contiguous virtual memory blocks. It is much harder to exploit. It uses Metadata Encoding so that if you overflow a buffer, you can’t easily predict what the pointer should be to redirect execution.

**Frontend vs. Backend Allocators:**  
The Heap Manager is split into two layers to balance speed and capacity.

**Backend Allocator:** Manages large chunks of virtual memory received from the OS. It handles large allocations and manages leftovers by splitting big blocks into smaller ones using FreeLists. It is slower because it requires locking the Heap to prevent multi-threading conflicts.

**Frontend Allocator:** It is an optimization layer sitting on top of the Backend designed specifically for speed. It handles small, frequent allocations to avoid the overhead of the Backend. The most common Frontend is the Low Fragmentation Heap (LFH). Instead of splitting large blocks which leaves behind small, unusable gaps (External Fragmentation), it uses Buckets of fixed sizes (like 16-byte or 32-byte buckets). This is much faster because it assigns pre-sized slots immediately rather than searching through complex lists for a fit.

**Segment Heap’s 3-Layer Model:**  
In the Segment Heap, the old Frontend/Backend split is replaced by three specialized managers based on the size of the request.

**LFH (Modern Frontend):** Still used for small, frequent allocations (usually < 16 KB). It is more secure than the old version because it uses randomized slot selection to defeat heap grooming.

**Variable Size Allocator:** A new middle-man layer for allocations too big for the LFH but too small for the Large Block Allocator. It is offset-based rather than pointer-based, making it harder to corrupt with traditional linked-list attacks.

**Segment / Large Block Allocator (Modern Backend):** It manages the actual large segment of memory from the OS. Instead of using the old NT FreeLists, it uses Free Trees to track free space, which changes how we research its metadata.

**Heap Structure:**  
**NT Heap Structure:**  
In the NT Heap, metadata and user data are stored together in the same memory range. This is called In-Band Metadata.
- **The Heap Header:** (_HEAP) This is the root structure. It contains the signature, flags, and pointer to the FreeLists and Lookaside Lists.
- **The Chunk Header (_HEAP_ENTRY):** Every single allocation starts with a header (8 bytes on x86, 16 bytes on x64). This header sits directly before the pointer the program receives.
  - **Size:** The total size of the block (Data + Header + Padding)
  - **Previous Size:** The size of the chunk physically located before it. This allows the manager to walk the heap backward.
  - **Flags:** A bitmask indicating if the chunk is Busy (Allocated) or Free, or if it’s the Last Entry in a segment.
  - **Unused Bytes:** If the program asked for 10 bytes but received a 16-byte block, this field tracks the 6 bytes of padding.
  - **Small Tag Index:** A security feature that helps detect if the header was messed with.
- **The Layout:** [HEADER][YOUR DATA][PADDING] --> [NEXT HEADER][NEXT DATA]

**Segment Heap Structure:**  
The Segment Heap is a total architectural shift. It moves away from “one header per chunk” for small objects to prevent simple overflows from hitting metadata. It uses a Hierarchical Structure:
- **Heap Segment:** The largest unit. The heap manager grabs large chunks of virtual memory (Segments) from the OS.
- **Pages (4 KB):** Segments are divided into standard memory pages.
- **Subsegments:** A group of one or more pages dedicated to a specific Bucket size (for example a 32-byte segment).

**A) LFH Structure:**  
For small objects, the Segment Heap uses Out-of-Band Metadata.
- **The Subsegment Header:** There is only one header at the very beginning of the page/subsegment.
- **The Bitmap:** Instead of each chunk having a Busy/Free flag in a header, the Subsegment Header contains a Bitmap. Each bit represents a slot in that page.
  - If bit number 5 is 1, the 5th slot in that page is occupied.
  - **User Data:** Your data sits in free slots with no headers between them. Overwriting one buffer just hits the next buffer’s data, not the heap’s management pointers.

**B) Variable Size (VS) Structure:**  
For objects too large for the LFH, the Segment Heap uses VS Chunks.
- **VS Header:** These chunks do have headers again, but they are Encoded.
- **The XOR Secret:** The header is XORed with a random Heap Key generated when the process starts. To the CPU (or an attacker), the header looks like random garbage. The manager decodes it on-the-fly to read the size and flags.

**C) Large Allocations:**  
For massive requests (usually > 512 KB), the Segment Heap bypasses the subsegment logic entirely and creates a dedicated Large Allocation Block that points directly to a range of virtual memory pages.

**Lookaside Lists:**  
We can think of Lookaside Lists as a small, private stash of recently freed blocks.
- **Structure:** A Singly-Linked List. It’s basically a LIFO stack.
- **How It Works:** When you free a small block, the OS doesn’t immediately put it back in the main warehouse. It stashes it here.
- **Key Detail:** It is Pre-Processor. This means a CPU core can grab memory from its own Lookaside List without locking the Heap, making it incredibly fast.
- **Limit:** It only holds a fixed number of small blocks. If the list is full, the block goes to the FreeLists.

**FreeLists:**  
When a block is too big for the Lookaside List, or the Lookaside List is full, it goes into the FreeLists.
- **Structure:** An array of 128 Doubly-Linked Lists.
  - **FreeList[0]:** It is the Catch-All bucket. Anything that doesn’t fit in the first 127 buckets (usually allocations > 1024 bytes).
    - Unlike other buckets which are Fixed Size, FreeList[0] is stored by size (smallest to largest).
    - If the OS can’t find a block in the fixed buckets, it performs a Best Fit search in FreeList[0]. It finds the smallest block that is still big enough for the request, splits it, and files the remainder back into the appropriate list.
  - **FreeList[1] Through FreeList[127]:** These are for Fixed Size blocks (for example FreeList[2] might only hold 16-byte blocks.
- **How It Works:** When you call malloc(32), the Heap Manager doesn’t search, it calculates (Requested Size + Header) / Alignment. It lands exactly on the index it needs. When a bucket is empty, a Search and Split algorithm starts. If you want 32 bytes (FreeList[4]) but that list is empty, the OS doesn’t stop. Instead it:
  - **Search:** It looks at the next bucket, FreeList[5] (40-byte blocks).
  - **Take:** It pulls a 40-byte block out of that list.
  - **Split:** It carves out the 32 bytes asked.
  - **Remainder:** Now there are 8 bytes left over.
  - **Re-File:** The OS takes those 8 bytes and puts them into the correct bucket for their new size (FreeList[1]) 
- **Vulnerability:** Since these are Doubly-Linked List, every free block has a Flink (Forward Link) and a Blink (Backward Link) pointer in the memory. Overwriting these is the classic way to get an Arbitrary Write.

**Free Trees:**  
Linked Lists in Segment Heap and larger allocations in the NT Heap are too slow to search, that’s why Free Trees exist.
- **Structure:** A Digital Tree or a Red-Black Tree.
- **How It Works:** Instead of walking a long list of blocks, the OS uses a tree structure where each node represents a range of free sizes.
- **Why Use It?** It allows the OS to find the Best Fit for a large allocation in O(log n) time instead of O(n). It’s much more efficient for managing the Deep Storage of the Heap.

**Coalescing (The Merge Operation):**  
If the Heap just kept splitting blocks, it would eventually turn into millions of tiny, useless fragments. Coalescing is the cleanup process.
- **The Logic:** When you free() a block, the OS looks at the blocks immediately before and after it in physical memory.
- **The Merge:** If the neighbor is also Free, the OS unlinks them from their current Freelists and merges them into one larger contiguous block.
- **Result:** This prevents fragmentation and ensures that large requests can still be fulfilled later.

**Allocation Algorithms:**  
On Windows, these functions usually call each other in a specific hierarchy: new (C++) –> malloc (C / CRT) –> HeapAlloc (Win32) –> VirtualAlloc (Kernel/OS).

**1. The High-Level Layer (Language Specific):**  
These are portable, meaning they work on Linux/Mac too, but on Windows ,they eventually talk to the OS-Specific APIs.
- **new / delete (C++):** This is the C++ operator. It does two things, it allocates the memory (usually by calling malloc) and then it calls the Constructor to initialize the object.
- **malloc / free (C / CRT):** This comes from the C Runtime (CRT)
  - **How It Works:** It manages a CRT Heap, which is actually just a wrapper around Windows HeapAlloc call.
  - **Debug Feature:** In Debug Mode, malloc adds extra guard bytes (like 0xFD or 0xBAADF00D) around the data to help catch overflows during development.

**2. The Middleware Layer (Win32 API):**  
Since my focus is on Windows, the following functions are Windows-specific, they won’t work on Linux or macOS.
- **HeapAlloc / HeapFree:** It takes a Heap Handle (like the GetProcessHeap()) and returns exactly the number of bytes you asked for. You can create your own private heaps using HeapCreate to prevent different parts of your program from messing with each other’s memory.
- **GlobalAlloc / LocalAlloc:** These are deprecated for new code. In modern 32/64-bit Windows, these are just wrappers that internally call HeapAlloc using the default process Heap.

**3. The Lowest Level (Memory API):**  
- **VirtualAlloc / VirtualFree:** It doesn’t care about bytes; it only cares about 4 KB Pages. When a heap (HeapAlloc) runs out of room, it calls VirtualAlloc to grab a fresh 64 KB or 1 MB segment memory from the OS. This allows you to set specific memory protections (like making a page Read-Only or Executable).

**More Details on C Functions:**  
- **malloc (Memory Allocation):** It requests a block of specific size (in bytes) from the Heap. It doesn’t clear the memory. Whatever was in that RAM before (garbage data) will still be there. If the CRT’s internal “pool” is empty, malloc calls HeapAlloc. If you ask for a massive amount (usually > 512 KB), it might skip the Heap Manager entirely and call VirtualAlloc to get a dedicated page for you.
- **calloc (Clear Allocation):** This is like malloc with a built-in clean up step. It takes two arguments: the number of elements and the size of each element (for example, calloc(10, sizeof(int))). It initializes all bits in the allocated block to 0. This is safer than malloc because it prevents Information Leakage, where a new variable accidently contains sensitive data left behind by a previous function. It performs an integer overflow check on the multiplication (count * size) before allocating, which provides a small layer of protection against Heap Overflow caused by calculation errors. 
- **realloc (Re-Allocation):** It tries to expand or shrink a block you already allocated. It works in two paths:
  - **In-Place:** If there is enough free space immediately after your current block, it just updates the Size field in the header and gives you back the same pointer.
  - **Move:** If there isn’t enough space, it finds a new hole elsewhere, copies your old data to the new spot, frees the old block, and returns a new pointer.  
  If realloc fails, it returns NULL, but the original pointer is still valid. If you do ptr = realloc(ptr, 1000) and it fails, you just lost the pointer to your original data (since ptr now is NULL) which leads to a Memory Leak.
- **free (De-Allocation):** It tells the Heap Manager “I’m done with this block, give it back to the OS”. Then the system does some work: It checks the Lookaside List first, can it stash this block there for a quick reuse? If not, it puts it back in a FreeList (NT Heap) or marks a bit as 0 in a Bitmap (Segment Heap). It checks if the neighbors are free to perform Coalescing. But there is a problem, free does not erase the pointer variable, it only marks the memory as available. If you use that pointer again, you’ve triggered a Use-After-Free. If you call free on the same pointer twice, you’ve triggered a Double Free.

**Heap Exploits and Defenses:**  
**Exploit 1:** Heap Overflow  
Heap Overflow occurs when you write data into a heap-allocated buffer beyond its capacity. This causes the extra data to overwrite the memory areas adjacent to the buffer.

In the NT Heap, the danger is that a Chunk Header sits directly after the data. By overflowing the buffer, an attacker can overwrite this header’s size, flags, or pointers (Flink/Blink). This tricks the Heap manager into merging blocks incorrectly or writing data to a location of the attacker’s choosing.

In the Segment Heap, while small objects don’t have headers to hit, the overflow will corrupt the User Data of the next object. If that next object contains a function pointer, the attacker can overwrite it to hijack the program’s flow.

**Exploit 2:** Use-After-Free (UAF)  
UAF occurs when a program continues to use a pointer after the memory it points to has been freed.

When a programmer frees a block, the Heap Manager marks that space as available in a FreeList or Bitmap, but the pointer variable in the code often remains unchanged (a dangling pointer). An attacker can perform a new allocation of the same size. Because the Heap Manager wants to be efficient, it will likely give the attacker the exact same memory slot that was just freed. Now, if the original program tries to use that old pointer, it is actually interacting with the attacker’s malicious data, leading to code execution or information leaks.

**Exploit 3:** Double Free  
A Double Free exploit happens when the free() function is called twice on the same memory address without a new allocation in between.

This is dangerous because it corrupts the Heap Manager’s internal tracking logic. In the NT Heap, freeing a block twice can cause the FreeLists to point back to themselves, creating a loop. When the program later asks for a new allocation, the manager might return a pointer to its own metadata. This allows an attacker to overwrite the heap’s management structures directly, often resulting in an Arbitrary Write, where the attacker can write any value to any memory address.

**Exploit 4:** Type Confusion  
Type Confusion occurs when a program allocates a memory block for one type of data (Type A) but later accesses it as a different type (Type B).

For example, a program might allocate 8 bytes to store a simple integer. If an attacker can trick the program into later treating those 8 bytes as a Function Pointer inside a struct, the program will try to execute whatever number is stored in that integer. By controlling the value of the integer, the attacker controls where the program jumps.

**Exploit 5:** Heap Grooming  
Heap Grooming is a technique used to make heap exploits more reliable. Because modern heaps are complex and somewhat randomized, an attacker doesn't always know where their target object is located.

Grooming involves making many specific allocations and deallocations to clean up the heap's layout. By filling up holes and forcing the manager to place objects in a predictable sequence, the attacker ensures that the vulnerable buffer is sitting exactly next to the target object they want to overwrite.

**Defense 1:** Heap Metadata Encryption (XORing)  
This defense protects the integrity of the Heap Headers. Instead of storing the header data in plain text, the OS XORs the header with a random Secret Cookie or Heap Key generated when the program starts.

Whenever the Heap Manager needs to read a header, it decodes it using the key. If an attacker performs a Heap Overflow and overwrites the header with a new value, the decoding process will result in a completely nonsensical value. The manager will detect this corruption and immediately crash the program to prevent the exploit from continuing.

**Defense 2:** Guard Pages  
Guard Pages are poisoned pages of memory placed at the edges of heap segments. These pages are set with no permissions (no read, no write, no execute).

If an attacker tries to perform a massive Heap Overflow or a Heap Spray to fill memory with malicious code, they will eventually hit one of these Guard Pages. Since the page cannot be accessed, the CPU triggers an Access Violation error, which stops the attack and shuts down the process.

**Defense 3:** User-Mode Stack Trace (UST)  
UST is a defense mechanism that records a database of who did what on the heap. It keeps track of the stack trace for every allocation and deallocation.

This helps prevent exploits like Use-After-Free and Double Free by allowing the system to verify the history of a memory block. If a piece of code tries to free a block that has already been freed, the system can check the trace database, see the previous free operation, and block the second one. While often used for debugging, it provides a strong layer of visibility into heap abuse.

**Defense 4:** Arbitrary Code Guard (ACG)  
ACG is a modern security policy that enforces the Write XOR Execute (W^X) principle more strictly. It prevents any memory from being both writable and executable at the same time.

Even if an attacker successfully overwrites a pointer on the heap, ACG makes it impossible for them to point that pointer to Shellcode they wrote into a heap buffer. This is because ACG prevents a heap page (which is Writable) from ever being turned into an Executable page. It also prevents the modification of existing executable code, making it very difficult for an attacker to find a place to run their malicious instructions.

**Defense 5:** Control Flow Guard  
The same as the one in the Stack!
