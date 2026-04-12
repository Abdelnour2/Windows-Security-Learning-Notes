# Phase 1 - Day 4: Tuesday 31 March 2026
## Summary:
I did a research on Virtual Memory in this day.

## Note:
Documentation first, Notes at the bottom.

## Documentation:
After starting my research on Stack, I got curious about multiple stuff:
- All about Virtual Memory
- During the prolog process, the function gives itself a “size” for its stack frame. If I’m doing this manually, how do I know the “size” I need to put?
- Stack Exploits and Defenses.

I’ll start with Virtual Memory. And I’ll see where to go from there. I found multiple sources to research about Virtual Memory, the main ones I used are Google’s AI Mode and this [video](https://www.youtube.com/watch?v=A9WLYbE0p-I).

**Update:**  
It turns out that there are a lot of things behind the title “Virtual Memory”. It took a lot of time! But it is fascinating to know how a small problem (letting programs access physical memory directly) introduces a lot of complex features and abstractions!

I think I have a solid foundation of Virtual Memory at the moment. So tomorrow I’ll go back to the Stack, and then the Heap.

## Notes:
### Topic: Virtual Memory

In the early days of computers, programs could actually talk directly to the Physical Memory (RAM). which created multiple problems:
**Problem 1:** Not Enough Memory  
Let’s say the computer has 1 GB of RAM. If the program tried to read from an address that is above 1 GB. The program would crash, since the address doesn’t exist!

**Problem 2:** Memory Fragmentation  
Let’s say the computer has 4 GB of RAM. And we need to run 3 programs: a video game that requires 2 GB of RAM, a video player that requires 1 GB of RAM, and a video editing software that requires 2 GB of RAM. So 5 GB of RAM in total.

<img src="Phase-1/Images/Day-4/Image-2.png" align="right" width="75">
<img src="Phase-1/Images/Day-4/Image-1.png" align="right" width="100">
Let’s say we ran the video player, and the game at the same time. That’s 3 GB out of 4 GB used. We can’t open the video editing software since there is no space left!

If we closed the video player, now we have 2 GB of RAM free, but we still can’t open the video editing software. Since the 2 GB of RAM are not connected! We can split the 2 GB into 2 1 GB to open the video editing software, but it requires a different program to handle that, and it would be very complicated since we can’t access memory from using just array indexing, we would have to implement some indirect memory accessing feature and in reality, this is what Virtual Memory is.

**Problem 3:** Security  
Since every program can access the whole Physical Memory. Any program running can see/edit any memory address. This means that a program can see what other programs are doing, and changing their data, corrupting other programs.

**Problem 4:** Memory Waisting  
<img src="Phase-1/Images/Day-4/Image-3.png" align="right" width="300">
Let’s say we want to run 4 programs. And these 4 programs require the same C libraries to function. Those 4 C libraries need to be loaded for each program, meaning it will be loaded 4 times in this example, taking more space than it should.

**Solution:**  
The solution was to give each program their own memory space which eventually is the Virtual Memory. So the video editing software will have its own Virtual Memory, the video game will have its own Virtual Memory, and so on.

**Some terminologies:**  
- **Virtual Address:** The memory address of a Virtual Memory
- **Physical Address:** The memory address of a Physical Memory.
- **Virtual Memory:** The virtual memory space each program has.
- **Physical Memory:** The actual RAM.

The Physical Memory is divided into 2 main parts, the first part is relatively small, and reserved for the OS. The second part, taking the rest of the memory space, is for programs.

<img src="Phase-1/Images/Day-4/Image-4.png" align="right" width="250">
Virtual Memory is essentially giving the illusion to programs that they have all the space they need! The size of the Virtual Memory is the maximum size the CPU can handle. A 32-bit CPU can read 232bytes which is 4,294,967,296 bytes which is ~4 GB. So the Virtual Memory in that case will be 4 GB. Between the Virtual Memory and the Physical Memory is a Mapper that takes the data written in the Virtual Addresses by programs in their Virtual Memory, and places them in Physical Addresses in the Physical Memory.

<br><br>
**Solution to Problem 1:** Not Enough Memory  
If a program writes something on its Virtual Memory, but there is no space for it in the Physical Memory, the mapper can place the data in the Virtual Address in a Physical Address on the Disk instead of RAM.

There is a process in this called Swap, where the CPU can remove one data from RAM and place it in Disk, and get another data from Disk and place it into RAM. The mapper will be updated too when this happens.

This process allows the computer to have additional memory in the Disk when needed. This memory is called Swap Memory.

When the CPU wants to read data from RAM, but it’s on the Disk instead, we call that a Page Fault.

The whole swapping process is slower than just reading from RAM, so having more RAM is much better!

**Solution to Problem 2:** Memory Fragmentation  
I mentioned an example where we had 4 GB of RAM and there is 2 GB free in RAM but they are separate. Virtual Memory solves this problem since the program sees that it has all the space it needs and its continuously. The mapper takes the data from the Virtual Memory and places them into the 2 separate 1 GB chunks in Physical Memory.

**Solution to Problem 3:** Security  
2 different programs can read/write the same Virtual Address (for example address 0x64), the mapper then takes these data and places them in the Physical Memory at different Physical Addresses (0x04 for program 1, and 0x10 for program 2)

**Solution to Problem 4:** Memory Waisting  
If multiple programs want to read from a single library. They can do that by reading from a Shared Memory Spacing RAM, or from other places like the Disk, or a Network Interface for example.

**Virtual Memory Implementation:**  
<img src="Phase-1/Images/Day-4/Image-5.png" align="right" width="350">
When a program asks for data stored at virtual address 64 in its virtual memory. The mapper looks at the physical address of that data and brings it back to the program.

The mapping table is called Page Table, and each mapping is called Page Table Entry (PTE).

Let’s say our CPU is 32-bit (meaning the size of a Virtual Memory is 4 GB). Page Tables work with WORDs, and a WORD is the size of a CPU register, so in this case, WORD = 32-bits (4 Bytes). So the Page Table has an entry for every WORD in the Virtual Address Space.

The size we need to store the mapping for each word is: 232 bytes = 230 words ~= 1 billion PTEs. A single entry stores the physical address too which is also 32-bits, so the size of the table will be 4 GB. And there is a Page Table for every program, so this is clearly a lot!

For this reason, the Virtual and Physical Memory are divided into chunks, where each chunk is a range of addresses called pages, hence the name Page Table.

<img src="Phase-1/Images/Day-4/Image-6.png" align="right" width="400">

**Quick Note:** Chunks in Virtual Memory are called Pages, and Chunks in Physical Memory are called Frames.

So the idea is that instead of mapping each individual address (WORD), we map each individual page. For example we can say that Page 0 is mapped to Frame 3.

This means that we don’t need a single PTE for each WORD anymore, instead, we need a PTE for each 4 KB (equivalent to 1024 words where a word is 32-bits). A PTE doesn’t need to be 4 KB, but it’s the most common size.

Since the max size of the Virtual Memory (in this example) is 4,294,967,296 bytes. And each Page is 4 KB. We’ll have 1,048,576 pages. So the PTE will have ~1 Million entries.

Since the PTE is still 4 Bytes in size, the Page Table will be 4 MB instead of 4 GB.

**How Do We Map Pages:**  
Let’s say that Page 1 (4096 - 8191) is mapped to Frame 2 (8192 - 16383). What is the physical address for the virtual address 4200? The answer is 8296. Because the individual address in the virtual page is mapped to the physical address using the same offset. Virtual Address 4200 is 4096 + 104. So the offset is 104. In the physical Address, 8192 + 104 is 8296.

**Page Faults:**  
When a CPU tries to read some Virtual Address, it looks at the mapping in the Page Table. The Page Table says that the data is on Disk, so the CPU doesn’t know how to read it from RAM. Instead, the CPU raises an exception called a Page Fault. The OS handles this exception by choosing which page to evict from RAM (usually the least recently used one). If the page is dirty (meaning the program has written something to it after loading it from Disk), it saves it in the Disk. If the program hasn’t written anything, then there is no reason to save it to the Disk. After that, the OS loads the requested page from Disk into RAM, updates the Page Table and goes back to execute the same instruction that caused the Page Fault.

Page Faults are extremely slow, and when they happen, the OS usually switches to execute another program’s task in the meantime. Modern CPU architecture has modules called DMAs which stands for Direct Memory Access. DMAs can load data from Disk to Ram directly while the CPU is doing something else.

**Translation Lookaside Buffer (TLB):**  
For every memory access, we need to find the Page Translation in the Page Table (which is stored in RAM, so this requires a RAM access), translation of the address and then access the actual data from RAM again. This is so expensive.

A special Hardware component in the CPU is added to find Physical Address very quickly. It caches the translations from Virtual to Physical Addresses. The component is called Translation Lookaside Buffer (TLB). This cache is very small but very fast.

When a CPU wants to translate the Virtual Address, it asks the TLB, if the translation is in the TLB, then it’s really fast, usually 0.5 - 1 cycle. If it’s not in TLB, then it needs to load it from the Page Table into TLB.

Modern CPU architectures have 2 TLBs, 1 for instructions, and the other for Data. TLBs are small in size, usually 4096 entries on Modern Architectures.

If the TLB table is full and needs to get a translation that is not in it. It removes the one that is least recently used and loads the corresponding page. If the program wants to use some data that is on disk, it’s still be saved in TLB, and the CPU will generate a Page Fault.

The Hardware that is responsible for address translation and generating page faults is called Memory Management Unit (MMU). MMU is usually on the CPU board and is programmed by the OS.

**Multi-Level Page Tables:**  
As I noted before, a 32-bit Page Table is about 4 MB. That sounds small on paper, but we need to keep in mind that every single process needs its own table. If 100 processes are opened, that’s 400 MB of RAM.

The problem here is that most programs only use a tiny bit of their 4 GB Virtual Memory space, for example, some at the bottom for code, and some at the top for the stack. And in most cases, the middle will be just empty.

A Single-Level Page Table forces the computer to keep the full 4 MB map for the entire 4 GB, even the empty parts.

The solution is to break the 4 MB table into smaller 4 KB chunks. Have a Page Directory (the Master Map). The Master Map points to smaller Page Tables. If a huge range of Virtual Memory is empty, you just don’t create the smaller Page Tables for it. The result is instead of 4 MB per process, a small program might only need 8 KB to map its memory.
