# Phase 1 - Day 15: Wednesday 22 April 2026
## Summary:
Made the 3rd project of Category 1: The Struct Aligner

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Continuing my projects' journey, Today we have Project 3: The Struct Aligner. I was researching if there is a way to see the offset of memory, and found that there is a function in the stddef header called offsetof, so I’ll be using that. I went ahead of myself and was researching the goal of this project! And I learned that the order of struct members matter! If they were ordered from largest to smallest, that will consume much less memory than doing the opposite.

So this project is very small! I’m going to create 3 different structs with the same members but in different orders, and print the offsetof for each member for the 3 different structs and the size of them!

These are the structs:
```c
struct First {
    int number;
    char name[20];
    char letter;
};

struct Second {
    char letter;
    int number;
    char name[20];
};

struct Third {
    char name[20];
    char letter;
    int number;
};

```

The output of the program was interesting:  
Struct 1: First  
Total Size: 28  
Offset of Member 1: 0  
Offset of Member 2: 4  
Offset of Member 3: 24  

Struct 2: Second  
Total Size: 28  
Offset of Member 1: 0  
Offset of Member 2: 4  
Offset of Member 3: 8  

Struct 3: Third  
Total Size: 28  
Offset of Member 1: 0  
Offset of Member 2: 20  
Offset of Member 3: 24  

**Wrong Old Analysis:**  
I analyzed each one to see the reason behind each one:
In the first struct, the first member starts at offset 0, since it is an int, it needs 4 bytes so it will end at offset 3. So the name array will start at offset 4. The size of the name array is 20 bytes. So it will end at offset 23, making the letter start at offset 24. The letter is only 1 byte so it will use only the offset 24. Since now we have 25 bytes used, 25 is not divisible by 4, the program adds 3 extra bytes of padding to reach the size of 28 (so offsets 25, 26, and 27 will be used as padding).

The second struct, the letter starts at offset 0. The number will come next, but it can’t start at offset 1 since it is not divisible by 4, so the program will add 3 bytes to the letter (offsets 1, 2, and 3), and let the number start at offset 4 and end at offset 7. Letting the name array to start at offset 8 and ending at offset 27. Since we have a total of 28 bytes. No need for extra padding.

The third struct, first the name starts at offset 0 and ends at offset 19. Then the letter starts and ends at offset 20. We’ve used 21 bytes so far which is not divisible by 4, so the program adds padding (offsets 21, 22, and 23) and lets the number start at offset 24 and ends at offset 27. 28 bytes in total so no need for extra padding.

Interestingly, when I changed int to double, the results changed completely (The order of the members are still the same)!  The new results:
Struct 1: First  
Total Size: 32  
Offset of Member 1: 0  
Offset of Member 2: 8  
Offset of Member 3: 28  

Struct 2: Second  
Total Size: 40  
Offset of Member 1: 0  
Offset of Member 2: 8  
Offset of Member 3: 16  

Struct 3: Third  
Total Size: 32  
Offset of Member 1: 0  
Offset of Member 2: 20  
Offset of Member 3: 24  

**Wrong Old Analysis**  
Analyzing the new results:
The first struct, the number is now a double which means 8 bytes instead of 4. So it starts at offset 0, and ends at offset 7. Then comes the name starting at offset 8 and ends at offset 27, then comes the letter at offset 28. Now since we have 29 bytes, the program will add padding (offsets 29, 30, and 31) so the total size can be divisible by 4.

The second struct, starting with the letter at offset 0. Now interestingly, the program seems to be switched for the offset to be multiple of 8 now instead of 4, since the number came at offset 8 instead of 4. So the program added padding to all offsets 1, 2, 3, 4, 5, 6, and 7 letting the number to start at offset 8 and ending at offset 15. Then the name starts at offset 16, ending at offset 35. Since 36 bytes is not divisible by 8, the program added more padding making the total size of the struct to be 40 bytes!

The third struct, Starting with the name at offset 0 to offset 19. Then the letter at offset 20. Then the program added padding at offsets 21, 22, and 23 letting the number to start at offset 24 (divisible by 8), and ending at offset 31. The total size is 32 bytes so no need to add more paddings!

I wonder why the program didn’t use the multiple by 8 for the first struct!

After digging into it, I’ve realized that my old analysis is wrong! There is no such thing as “a general padding for everything”. Each data type needs to be multiple by its size! I’ll redo the analysis to make it clear, and mark the old ones as wrong!

**Case 1:** Int  
In the first struct, the first member starts at offset 0, since it is an int, it needs 4 bytes so it will end at offset 3. The name array will start at offset 4. The size of the name array is 20 bytes. So it will end at offset 23, making the letter start at offset 24. The letter is only 1 byte so it will use only the offset 24. We’ve used 25 bytes. The final struct needs to be aligned to be multiple of its own largest member which is the int. So the final size of the struct needs to be multiples of 4. 25 is not multiple of 4, so the program adds padding in the offsets 25, 26, and 27. Now the final size is 28 bytes.

The second struct, the letter starts at offset 0. The number will come next. Since its size is 4, it needs to start an offset that is multiple by 4. Offset 1 is not, so the program adds padding to offsets 1, 2, and 3, to let the number start at offset 4 which is divisible by 4. And the number will end at offset 7. Letting the name array to start at offset 8 and ending at offset 27. Since we have a total of 28 bytes and the largest member is 4 bytes. No need for extra padding.

The third struct, first the name starts at offset 0 and ends at offset 19. Then the letter starts and ends at offset 20. Then the number which needs 4 bytes. Offset 21 is not divisible by 4, so the program adds padding to offsets 21, 22, and 23 so that the number can start at offset 24 and end at offset 27. Again since we’ve 28 bytes and the largest member is 4 bytes, no need for extra padding.

**Case 2:** Double  
The first struct, the number is now a double which means 8 bytes instead of 4. So it starts at offset 0, and ends at offset 7. Then comes the name starting at offset 8 and ends at offset 27, then comes the letter at offset 28. We’ve used 29 bytes so far and the largest member is 8 bytes. So we need to make the struct divisible by 8. 29 is not so we need to add padding in the offsets 29, 30, and 31 so that total size can be 32 bytes which is divisible by 8.

The second struct, starting with the letter at offset 0. Then the number which needs 8 bytes, so it needs to start at an offset that is divisible by 8, so the program adds paddings from offset 1 to 7, so it lets the number start at offset 8 which is divisible by 8. Then the number ends at offset 15. Then the name starts at offset 16, and ends at 35. We’ve used 36 bytes and the largest member of the struct is 8 bytes, we need to make the total size a multiple of 8, so the program adds another padding from offset 36 to 39 to make the total size of the struct 40 which is a multiple of 8.

The third struct, Starting with the name at offset 0 to offset 19. Then the letter at offset 20. Then the number which needs 8 bytes needs to start at an offset divisible by 8. So the program added padding at offsets 21, 22, and 23 letting the number start at offset 24 which is divisible by 8, and ends at offset 31. We’ve used 32 bytes and the largest member we have is 8 bytes. 32 is a multiple of 8 so no need to add more paddings!

**Reflection:**  
This project was small and easy to implement but the outcomes of it were really interesting and valuable!

## Full Code:
```c
#include <stdio.h>
#include <stddef.h>

struct First {
    double number;
    char name[20];
    char letter;
};

struct Second {
    char letter;
    double number;
    char name[20];
};

struct Third {
    char name[20];
    char letter;
    double number;
};

int main(int argc, char **argv) {
    printf("Struct 1: First\n");
    printf("Total Size: %zu\n", sizeof(struct First));
    printf("Offset of Member 1: %zu\n", offsetof(struct First, number));
    printf("Offset of Member 2: %zu\n", offsetof(struct First, name));
    printf("Offset of Member 3: %zu\n", offsetof(struct First, letter));

    printf("\n\n");
    
    printf("Struct 2: Second\n");
    printf("Total Size: %zu\n", sizeof(struct Second));
    printf("Offset of Member 1: %zu\n", offsetof(struct Second, letter));
    printf("Offset of Member 2: %zu\n", offsetof(struct Second, number));
    printf("Offset of Member 3: %zu\n", offsetof(struct Second, name));

    printf("\n\n");
    
    printf("Struct 3: Third\n");
    printf("Total Size: %zu\n", sizeof(struct Third));
    printf("Offset of Member 1: %zu\n", offsetof(struct Third, name));
    printf("Offset of Member 2: %zu\n", offsetof(struct Third, letter));
    printf("Offset of Member 3: %zu\n", offsetof(struct Third, number));

    return 0;
}
```