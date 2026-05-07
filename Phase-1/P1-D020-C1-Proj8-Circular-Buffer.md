# Phase 1 - Day 20: Thursday 7 May 2026
## Summary:
Made the 8th project of Category 1 which is Circular Buffer!

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Continuing the projects’ journey, today’s project is Circular Buffer. I don’t know anything about Circular Buffers so I started researching to learn about them! After the research I learned that they are similar to Queues where they follow the principle of First In First Out (FIFO). The only difference is that they never expand or shrink in size! Once the list is full they either block new data, or overwrite the existing ones starting from oldest to newest!

So what I’m going to do is make a struct that contains an array of integers of size 10, in addition to the array, I’m going to define integers that will represent the index of each head and the tail of that array, in addition to a count integer to keep track of the count we have! Here’s the struct:
```c
struct CircularBuffer {
    int list[10];
    int head;
    int tail;
    int count;
};
```

Next I’m going to make an Initialize function that takes an object of the struct and initializes it fully! First it will set the count to 0 indicating an empty array. Second, since the array is empty, both the head and the tail will be pointing at the first index, so both of them will have a value of 0. Function:
```c
void InitializeCircularBuffer(struct CircularBuffer *object) {
    (*object).count = 0;
    (*object).head = 0;
    (*object).tail = 0;
}
```

Next, before making the push and pop functions, I’m going to implement the Display Function to be able to test the push and pop later! Now displaying is a bit tricky in this program since this is not a linear array. Let’s say we have an array of 5, head and tail would be at 0. If we filled the array, tail would be at 0 and head at 5. If we pop elements, we won’t pop from the head, but from the tail instead. So we might have empty items from at the start and values at the middle or the end. So I can’t always start displaying from start to finish unless I want to display everything. If I want to only display the content I want to start from the tail and end at the head. After thinking about it, I’ll do both approaches for diversity! To achieve this, I added an enum called DisplayMode that has two values: full, and specific! The way a specific approach works is to get the index of the tail as a counter. Then have a for loop to loop exactly as the count! And at each iteration, prints the value at the counter index with its index! Full Function:
```c
enum DisplayMode {
    full,
    specific
};

void DisplayCircularBuffer(struct CircularBuffer *object, enum DisplayMode mode) {
    if ((*object).count == 0) {
        printf("List is Empty!\n\n");

        return;
    }

    if (mode == full) {
        printf("List Size: %d\n", CircularBufferListSize);
        printf("Items in the List: %d\n", (*object).count);
        printf("Head Pointing at: %d\n", (*object).head);
        printf("Tail pointing at: %d\n\n", (*object).tail);
        for (int i = 0; i < CircularBufferListSize; i++) {
            printf("Item at Index %d: %d\n", i, (*object).list[i]);
        }
    }
    else if (mode == specific) {
        int currentIndex = (*object).tail;

        for (int i = 0; i < (*object).count; i++) {
            printf("Item %d: %d (Stored at Index %d)\n", i + 1, (*object).list[currentIndex], currentIndex);

            currentIndex = (currentIndex + 1) % CircularBufferListSize;
        }
    }
}
```

Now, I’ll implement the Push function! Also for diversity, I’ll allow both block and overwrite approaches for when the array is full! To achieve that, I’ll use the same enum approach! The way the function will behave is that first, it will check if the array is full or not by seeing if the count equals the size of the array, if the array is full, it checks the mode, if block, a message will be printed to the user that the array is full!, if overwrite was the chosen mode, it will overwrite the value at the tail since it has the earliest member, then change the index of tail to the second earliest member! And then change the index of head to be the new overwritten member. If the array is not full, then it will just add a value in head’s position then increase it! Full Function:
```c
enum PushMode {
    block,
    overwrite
};

void Push(struct CircularBuffer *object, enum PushMode mode, int value) {
    if ((*object).count == CircularBufferListSize) {
        if (mode == block) {
            printf("The array is full and can't hold any more data!\n\n");

            return;
        }
        else if (mode == overwrite) {
            // The earliest data is at the tail's location
            // So the data at the tail's location needs to be overwritten
            // Then tail will point to the second earliest item in the list (so it will be shifted by 1 to the right since inserting goes from left to right)
            // Then the head needs to take the new index (which is tail's old value)
            (*object).list[(*object).tail] = value;
            (*object).tail = ((*object).tail + 1) % CircularBufferListSize;
            (*object).head = ((*object).head + 1) % CircularBufferListSize;

            return;
        }
    }

    // Add the value, then increase the head!
    (*object).list[(*object).head] = value;
    (*object).head = ((*object).head + 1) % CircularBufferListSize;
    

    (*object).count++;
}
```

And now finally, the Pop function. First, it will check if the list is empty by checking the count! If it is empty, then it will just return a message saying that the list is empty! If it is not empty, then it will pop the value at the tail’s location, then increase the tail by 1, and decrease the count! Full Code:
```c
int Pop(struct CircularBuffer *object) {
    if ((*object).count == 0) {
        printf("The list is already empty!\n\n");

        return -1;
    }

    int dataToPop = (*object).list[(*object).tail];
    (*object).tail = ((*object).tail + 1) % CircularBufferListSize;
    (*object).count--;

    return dataToPop;
}
```

**Reflection:**  
This is a small but really fun project to do! It made me practice enums, managing head and tails and many minor stuff!

## Full Code:
```c
#include <stdio.h>

#define CircularBufferListSize 10

enum DisplayMode {
    full,
    specific
};

enum PushMode {
    block,
    overwrite
};

struct CircularBuffer {
    int list[CircularBufferListSize];
    int head;
    int tail;
    int count;
};

void InitializeCircularBuffer(struct CircularBuffer *object);
void DisplayCircularBuffer(struct CircularBuffer *object, enum DisplayMode mode);
void Push(struct CircularBuffer *object, enum PushMode mode, int value);
int Pop(struct CircularBuffer *object);

int main(int argc, char **argv) {
    struct CircularBuffer buffer;
    InitializeCircularBuffer(&buffer);

    // Displaying the empty list
    DisplayCircularBuffer(&buffer, full);
    DisplayCircularBuffer(&buffer, specific);

    printf("\n\n");

    // Adding values
    for (int i = 0; i < CircularBufferListSize; i++) {
        Push(&buffer, block, i + 1);
    }

    // Displaying the full list
    DisplayCircularBuffer(&buffer, full);
    DisplayCircularBuffer(&buffer, specific);

    printf("\n\n");

    // Testing Push in Block and Overwrite modes
    Push(&buffer, block, 90);
    Push(&buffer, overwrite, 95);

    // Displaying the overwritten full list
    DisplayCircularBuffer(&buffer, full);
    DisplayCircularBuffer(&buffer, specific);

    printf("\n\n");

    // Popping Values
    printf("Value Popped: %d\n", Pop(&buffer));
    printf("Value Popped: %d\n", Pop(&buffer));
    printf("Value Popped: %d\n", Pop(&buffer));

    // Displaying the list after popping
    DisplayCircularBuffer(&buffer, full);
    DisplayCircularBuffer(&buffer, specific);

    printf("\n\n");

    return 0;
}

void InitializeCircularBuffer(struct CircularBuffer *object) {
    (*object).count = 0;
    (*object).head = 0;
    (*object).tail = 0;
}

void DisplayCircularBuffer(struct CircularBuffer *object, enum DisplayMode mode) {
    if ((*object).count == 0) {
        printf("List is Empty!\n\n");

        return;
    }

    if (mode == full) {
        printf("List Size: %d\n", CircularBufferListSize);
        printf("Items in the List: %d\n", (*object).count);
        printf("Head Pointing at: %d\n", (*object).head);
        printf("Tail pointing at: %d\n\n", (*object).tail);
        for (int i = 0; i < CircularBufferListSize; i++) {
            printf("Item at Index %d: %d\n", i, (*object).list[i]);
        }
    }
    else if (mode == specific) {
        int currentIndex = (*object).tail;

        for (int i = 0; i < (*object).count; i++) {
            printf("Item %d: %d (Stored at Index %d)\n", i + 1, (*object).list[currentIndex], currentIndex);

            currentIndex = (currentIndex + 1) % CircularBufferListSize;
        }
    }
}

void Push(struct CircularBuffer *object, enum PushMode mode, int value) {
    if ((*object).count == CircularBufferListSize) {
        if (mode == block) {
            printf("The array is full and can't hold any more data!\n\n");

            return;
        }
        else if (mode == overwrite) {
            // The earliest data is at the tail's location
            // So the data at the tail's location needs to be overwritten
            // Then tail will point to the second earliest item in the list (so it will be shifted by 1 to the right since inserting goes from left to right)
            // Then the head needs to take the new index (which is tail's old value)
            (*object).list[(*object).tail] = value;
            (*object).tail = ((*object).tail + 1) % CircularBufferListSize;
            (*object).head = ((*object).head + 1) % CircularBufferListSize;

            return;
        }
    }

    // Add the value, then increase the head!
    (*object).list[(*object).head] = value;
    (*object).head = ((*object).head + 1) % CircularBufferListSize;
    

    (*object).count++;
}

int Pop(struct CircularBuffer *object) {
    if ((*object).count == 0) {
        printf("The list is already empty!\n\n");

        return -1;
    }

    int dataToPop = (*object).list[(*object).tail];
    (*object).tail = ((*object).tail + 1) % CircularBufferListSize;
    (*object).count--;

    return dataToPop;
}
```