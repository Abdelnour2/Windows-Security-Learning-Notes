# Phase 1 - Day 16: Saturday 25 April 2026
## Summary:
Decided to start LetsDefend's course on Malware Analysis, and Made the 4th project of Category 1: Writing C++'s Vectors in C

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Before continuing with a new project, I was researching some security related stuff yesterday and I stumbled upon a platform called LetsDefend. They seem to be very well known in the cybersecurity industry, so I took a look at their lessons, courses and saw a Malware Analysis course in addition to a SOC Analysis Course. After more research about the platform and the courses, I decided to take the Malware Analysis course they offer, and maybe the SOC one in the future.

Now continuing with the projects! Today’s project is Generic Dynamic Array (Vector). I need to recreate the std::vector from C++ in C. I don’t know what the std::vector is so I'm going to research it.

After the research, I’ve learned that vectors in C++ are basically Dynamic arrays with extra functions to make it more useful! So for this project I’ll make a vector struct! And it will have CreateVector, DestroyVector, Display, AddToFront, RemoveFromFront, AddToBack, RemoveFromBack, RemoveAt functions.

Starting with the struct, it will have a pointer to its data, count, capacity, and itemSize of elements to know which data type the vector needs to operate.
```c
struct Vector {
    void *dataPointer;
    int count;
    int capacity;
    int itemSize;
};
```

Then, I made the CreateVector function, first it sets the count to 0, and has 10 default capacity and sets the item size then allocate some memory and the pointer is assigned to its pointer variable:
```c
void CreateVector(struct Vector *vector, int itemSize) {
    (*vector).count = 0;
    (*vector).capacity = 10;
    (*vector).itemSize = itemSize;

    (*vector).dataPointer = malloc((*vector).capacity * (*vector).itemSize);
}
```

Then I made the DestroyVector function, which is basically just freeing the vector:
```c
void DestroyVector(struct Vector *vector) {
    free((*vector).dataPointer);
}
```

After that, I started implementing the Display function which displays all elements of a vector. But here, I ran into a problem! The vector is generic, but printing data is specific to the data type. So without knowing the data type used for the vector, I can’t print the data correctly!

The first solution that came to mind is that I can ask the user for the data type! But the problem is that the user might misspell the name of the type, and another problem is that it would be annoying in the long run, where the user needs to enter the name of the data type for each Display.

Another solution was to deduce the data type based on the itemSize. For example 4 bytes is int, 8 bytes is double, 1 byte is char, … But this also has a problem of mismatching! A float is also 4 bytes. An array of characters can be any size.

After searching, I found that there is the concept of enums in C, so I learned it and made a simple enum as a start:
```c
enum DataType {
    typeInt,
    typeFloat,
    typeChar,
};
```

And updated the struct and the CreateVector functions:
```c
struct Vector {
    void *dataPointer;
    int count;
    int capacity;
    int itemSize;
    enum DataType dataType;
};

void CreateVector(struct Vector *vector, int itemSize, enum DataType dataType) {
    (*vector).count = 0;
    (*vector).capacity = 10;
    (*vector).itemSize = itemSize;
    (*vector).dataType = dataType;

    (*vector).dataPointer = malloc((*vector).capacity * (*vector).itemSize);
}
```

Now in the Display function, before printing, I checked for the dataType then printed the data. On my first attempt, an error occurred when I tried to do this:
```c
printf("Element %d: %d\n", i + 1, (*vector).dataPointer[i]);
```

After researching, it turned out that the problem is because initially the array is a void, so it can’t know its type, so I had to do some type casting! And the final function now look like this:
```c
void Display(struct Vector *vector) {
    printf("Vector's Capacity: %d\n", (*vector).capacity);
    printf("Vector's Count: %d\n\n", (*vector).count);
    
    for (int i = 0; i < (*vector).count; i++) {
        if ((*vector).dataType == typeInt) {
            int *intArray = (int *)(*vector).dataPointer;
            printf("Element %d: %d\n", i + 1, intArray[i]);
        }
        else if ((*vector).dataType == typeFloat) {
            float *floatArray = (float *)(*vector).dataPointer;
            printf("Element %d: %.2f\n", i + 1, floatArray[i]);
        }
        else if ((*vector).dataType == typeChar) {
            char *charArray = (char *)(*vector).dataPointer;
            printf("Element %d: %c\n", i + 1, charArray[i]);
        }
    }
}
```

Next, I’ll implement the AddToBack function which basically just adds an element at the index of the count. Since the vector is generic, I can’t add items at the count index as seen in the Display function, so I need to calculate the end index then put the raw bit data in the location. Final function:
```c
void AddToBack(struct Vector *vector, void *data) {
    if ((*vector).count == (*vector).capacity) {
        void *temp = realloc((*vector).dataPointer, ((*vector).capacity * 2) * (*vector).itemSize);

        if (temp == NULL) {
            printf("Failed to expand the list\n\n");

            return;
        }

        (*vector).capacity *= 2;
        (*vector).dataPointer = temp;
    }

    // start at the beginning then go to the count index in terms of item size, that's the location we're currently in
    void *memoryLocationToStore = (char *)(*vector).dataPointer + ((*vector).count * (*vector).itemSize);

    memcpy(memoryLocationToStore, data, (*vector).itemSize);

    (*vector).count++;
}
```

Now I’ll implement the RemoveFromBack function, I think this is very simple, I’ll just decrease the count by 1 so that when we want to add items later, the item deleted will get overwritten:
```c
void RemoveFromBack(struct Vector *vector) {
    if ((*vector).count == 0) {
        printf("The list is empty!\n\n");

        return;
    }

    (*vector).count--;
}
```

Next, I’ll do the RemoveAt function! First, if the list is empty, the function will stop immediately. Next, the function will ask the user for an index! If the number is larger than the count or negative, it will return an error message. If the index is correct, I’ll calculate the memory location of the item getting removed and the next one, then use memmove to shift the list! Finally decrease the count by 1:
```c
void RemoveAt(struct Vector *vector, int index) {
    if ((*vector).count == 0) {
        printf("List is empty!\n\n");

        return;
    }

    if (index < 0 || index >= (*vector).count) {
        printf("Invalid Index\n\n");

        return;
    }

    void *locationToDelete = (char *)(*vector).dataPointer + (index * (*vector).itemSize);
    void *nextItem = (char *)locationToDelete + (*vector).itemSize;

    int numberOfFollowingItems = (*vector).count - index - 1;

    if (numberOfFollowingItems > 0) {
        memmove(locationToDelete, nextItem, numberOfFollowingItems * (*vector).itemSize);
    }

    (*vector).count--;

    printf("Item has been removed successfully!\n\n");
}
```

Now, I’ll implement AddAtFront, similar to AddToBack, but now I’ll use memmove to shift the whole list to the right and overwrite the first element with the new one:
```c
void AddToFront(struct Vector *vector, void *data) {
    if ((*vector).count == (*vector).capacity) {
        void *temp = realloc((*vector).dataPointer, ((*vector).capacity * 2) * (*vector).itemSize);

        if (temp == NULL) {
            printf("Failed to expand the list\n\n");

            return;
        }

        (*vector).capacity *= 2;
        (*vector).dataPointer = temp;
    }

    void *firstIndex = (*vector).dataPointer;
    void *secondIndex = (char *)firstIndex + (*vector).itemSize;

    memmove(secondIndex, firstIndex, (*vector).count * (*vector).itemSize);

    memcpy(firstIndex, data, (*vector).itemSize);

    (*vector).count++;
}
```

Finally, RemoveFromFront, this is also similar to the previous Remove function but now we need to shift the whole list to the left and decrease the count by 1:
```c
void RemoveFromFront(struct Vector *vector) {
    if ((*vector).count == 0) {
        printf("List is empty!\n\n");

        return;
    }

    void *firstIndex = (*vector).dataPointer;
    void *secondIndex = (char *)firstIndex + (*vector).itemSize;

    size_t itemsToShift = (*vector).count - 1;

    if (itemsToShift > 0) {
        memmove(firstIndex, secondIndex, itemsToShift * (*vector).itemSize);
    }

    (*vector).count++;
}
```

## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

enum DataType {
    typeInt,
    typeFloat,
    typeChar,
};

struct Vector {
    void *dataPointer;
    int count;
    int capacity;
    int itemSize;
    enum DataType dataType;
};

void CreateVector(struct Vector *vector, int itemSize, enum DataType dataType);
void DestroyVector(struct Vector *vector);
void Display(struct Vector *vector);
void AddToBack(struct Vector *vector, void *data);
void RemoveFromBack(struct Vector *vector);
void RemoveAt(struct Vector *vector, int index);
void AddToFront(struct Vector *vector, void *data);
void RemoveFromFront(struct Vector *vector);

int main(int argc, char **argv) {
    struct Vector vector1;
    CreateVector(&vector1, sizeof(int), typeInt);
    
    // Taking just the int case
    for (int i = 0; i < 15; i++) {
        printf("Enter Element %d: ", i + 1);
        int data;
        scanf("%d", &data);
        AddToBack(&vector1, &data);
    }
    
    Display(&vector1);

    RemoveFromBack(&vector1);

    Display(&vector1);

    RemoveAt(&vector1, 3);

    Display(&vector1);   

    printf("\n\nEnter new Element: ");
    int data;
    scanf("%d", &data);
    AddToFront(&vector1, &data);

    Display(&vector1);

    DestroyVector(&vector1);

    return 0;
}

void CreateVector(struct Vector *vector, int itemSize, enum DataType dataType) {
    (*vector).count = 0;
    (*vector).capacity = 10;
    (*vector).itemSize = itemSize;
    (*vector).dataType = dataType;

    (*vector).dataPointer = malloc((*vector).capacity * (*vector).itemSize);
}

void DestroyVector(struct Vector *vector) {
    free((*vector).dataPointer);
}

void Display(struct Vector *vector) {
    printf("Vector's Capacity: %d\n", (*vector).capacity);
    printf("Vector's Count: %d\n\n", (*vector).count);
    
    for (int i = 0; i < (*vector).count; i++) {
        if ((*vector).dataType == typeInt) {
            int *intArray = (int *)(*vector).dataPointer;
            printf("Element %d: %d\n", i + 1, intArray[i]);
        }
        else if ((*vector).dataType == typeFloat) {
            float *floatArray = (float *)(*vector).dataPointer;
            printf("Element %d: %.2f\n", i + 1, floatArray[i]);
        }
        else if ((*vector).dataType == typeChar) {
            char *charArray = (char *)(*vector).dataPointer;
            printf("Element %d: %c\n", i + 1, charArray[i]);
        }
    }
}

void AddToBack(struct Vector *vector, void *data) {
    if ((*vector).count == (*vector).capacity) {
        void *temp = realloc((*vector).dataPointer, ((*vector).capacity * 2) * (*vector).itemSize);

        if (temp == NULL) {
            printf("Failed to expand the list\n\n");

            return;
        }

        (*vector).capacity *= 2;
        (*vector).dataPointer = temp;
    }

    // start at the beginning then go to the count index in terms of item size, that's the location we're currenly in
    void *memoryLocationToStore = (char *)(*vector).dataPointer + ((*vector).count * (*vector).itemSize);

    memcpy(memoryLocationToStore, data, (*vector).itemSize);

    (*vector).count++;
}

void RemoveFromBack(struct Vector *vector) {
    if ((*vector).count == 0) {
        printf("The list is empty!\n\n");

        return;
    }

    (*vector).count--;
}

void RemoveAt(struct Vector *vector, int index) {
    if ((*vector).count == 0) {
        printf("List is empty!\n\n");

        return;
    }

    if (index < 0 || (size_t)index >= (*vector).count) {
        printf("Invalid Index\n\n");

        return;
    }

    void *locationToDelete = (char *)(*vector).dataPointer + (index * (*vector).itemSize);
    void *nextItem = (char *)locationToDelete + (*vector).itemSize;

    size_t numberOfFollowingItems = (*vector).count - index - 1;

    if (numberOfFollowingItems > 0) {
        memmove(locationToDelete, nextItem, numberOfFollowingItems * (*vector).itemSize);
    }

    (*vector).count--;

    printf("Item has been removed successfully!\n\n");
}

void AddToFront(struct Vector *vector, void *data) {
    if ((*vector).count == (*vector).capacity) {
        void *temp = realloc((*vector).dataPointer, ((*vector).capacity * 2) * (*vector).itemSize);

        if (temp == NULL) {
            printf("Failed to expand the list\n\n");

            return;
        }

        (*vector).capacity *= 2;
        (*vector).dataPointer = temp;
    }

    void *firstIndex = (*vector).dataPointer;
    void *secondIndex = (char *)firstIndex + (*vector).itemSize;

    memmove(secondIndex, firstIndex, (*vector).count * (*vector).itemSize);

    memcpy(firstIndex, data, (*vector).itemSize);

    (*vector).count++;
}

void RemoveFromFront(struct Vector *vector) {
    if ((*vector).count == 0) {
        printf("List is empty!\n\n");

        return;
    }

    void *firstIndex = (*vector).dataPointer;
    void *secondIndex = (char *)firstIndex + (*vector).itemSize;

    size_t itemsToShift = (*vector).count - 1;

    if (itemsToShift > 0) {
        memmove(firstIndex, secondIndex, itemsToShift * (*vector).itemSize);
    }

    (*vector).count--;
}
```