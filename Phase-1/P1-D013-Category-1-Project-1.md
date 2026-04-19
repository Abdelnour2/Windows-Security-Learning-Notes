# Phase 1 - Day 13: Sunday 19 April 2026
## Summary:
Started the journey of making all 35 projects. Today, Project 1 of Category 1 has been made.

## Note:
Documentation first, and Full code at the bottom.

## Documentation:
Now that Category 0 is complete, I’m going to implement all 35 practice projects in order! Starting today with Category 1 Project 1: Custom malloc Wrapper.

My idea is that I’m going to add an array of pointers, that will contain every memory allocation I create in the project. In addition to implementing my own malloc and free functions (which basically call the real malloc and free), but in addition to keeping track of every pointer that has been created. Lastly, I’ll make a “report” function that checks the pointer array and sees if there is any data that has not been freed yet, and prints a warning.

To make it more fancy, I’ll create a custom struct that holds the pointer in addition to a name so that I know which pointer (the name of it) is not freed instead of just seeing the value of it. But this introduces a problem, since pointers can be in multiple types, for example int *number, this is an integer pointer, or char *name, this is a char pointer.

After researching, I found that I can indeed make the struct since there is a general type of pointer in C called void*. And in fact, malloc returns a void pointer that then transforms into the needed pointer.

So I made this struct:
```c
struct MemoryPointer {
    char name[50];
    void *pointer;
}
```

Then, I made a custom function to create the pointer array. It asks for the name of the pointer, its capacity, and count. Then creates that array, adds its address to it, and return it:
```c
struct MemoryPointer *CustomMainPointerListMalloc(char *listName, int *capacity, int *count) {
    struct MemoryPointer *tempArray = malloc(*capacity * sizeof(struct MemoryPointer));

    if (tempArray == NULL) {
        return NULL;
    }

    strcpy(tempArray[*count].name, listName);
    tempArray[*count].pointer = tempArray;
    (*count)++;

    return tempArray;
}
```

Usage in main:
```c
int pointerListCapacity = 5;
int pointerListCount = 0;
struct MemoryPointer *pointerList = CustomMainPointerListMalloc("pointerList", &pointerListCapacity, &pointerListCount);

if (pointerList == NULL) {
    printf("Creating the pointer array has failed!\n");

    return 1;
}
```

Now, to store names inside an array, I made a struct called People that has name as a field:
```c
struct People {
    char *name;
};
```

Then, I made this function to make any kind of arrays and adds its pointer and name to the pointer list:
```c
void *CustomArrayMalloc(char *listName, int *capacity, unsigned long long size, struct MemoryPointer *pointerList, int *pointerListCapacity, int *pointerListCount) {
    void *tempArray = malloc(*capacity * size);

    if (tempArray == NULL) {
        return NULL;
    }

    if (*pointerListCount == *pointerListCapacity) {
        *pointerListCapacity += 10;

        struct MemoryPointer *newPointerList = realloc(pointerList, *pointerListCapacity * sizeof(struct MemoryPointer));

        if (newPointerList == NULL) {
            printf("Failed at expanding the pointer array!\n");

            return NULL;
        }

        pointerList = newPointerList;
        pointerList[0].pointer = newPointerList;
    }

    strcpy(pointerList[*pointerListCount].name, listName);
    pointerList[*pointerListCount].pointer = tempArray;

    (*pointerListCount)++;

    return tempArray;
}
```

After this, now it’s time to make the name an additional function.

After making and testing the function, I noticed multiple problems. First, is that the peopleList doesn’t expand! Even though I implemented the expansion logic. After digging I noticed that the problem was that after the list expanded, the new pointer was not being assigned to the list which resulted in a dangling pointer. So I fixed that by changing the return type of the function from int to struct People *.

This fixed the addition problem, but now, another problem occurred, which is that whenever the pointerList expands.. The values of the peopleList are kind of corrupt! After digging, I noticed that it is the same problem! The pointerList pointer is not being updated on expansions. So I needed to fix this problem in both the addition, and the previous CustomArrayMalloc function. The solution I had was to use double pointers. That way I don’t need to change the return value of the previous function, and I can use it in this function since I can’t return two things.

With that, the previous function now look like this:
```c
void *CustomArrayMalloc(char *listName, int *capacity, unsigned long long size, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount) {
    void *tempArray = malloc(*capacity * size);

    if (tempArray == NULL) {
        return NULL;
    }

    if (*pointerListCount == *pointerListCapacity) {
        *pointerListCapacity += 5;

        struct MemoryPointer *newPointerList = realloc(*pointerList, *pointerListCapacity * sizeof(struct MemoryPointer));

        if (newPointerList == NULL) {
            printf("Failed at expanding the pointer array!\n");

            return NULL;
        }

        *pointerList = newPointerList;
        (*pointerList)[0].pointer = newPointerList;
    }

    strcpy((*pointerList)[*pointerListCount].name, listName);
    (*pointerList)[*pointerListCount].pointer = tempArray;

    (*pointerListCount)++;

    return tempArray;
}
```

And the addition function look like this:
```c
struct People *AddNametoListWithCustomMalloc(struct People *peopleList, int *peopleListCapacity, int *peopleListCount, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount, char *peopleListName) {
    char name[100];
    printf("Enter Name %d: ", *peopleListCount + 1);
    scanf(" %[^\n]", name);

    struct People newName;

    newName.name = malloc((strlen(name) + 1) * sizeof(char));

    if (newName.name == NULL) {
        printf("Name Malloc has Failed\n");

        return NULL;
    }

    strcpy(newName.name, name);

    if (*pointerListCount == *pointerListCapacity) {
        *pointerListCapacity += 5;

        struct MemoryPointer *newPointerList = realloc(*pointerList, *pointerListCapacity * sizeof(struct MemoryPointer));

        if (newPointerList == NULL) {
            printf("Failed at expanding the pointer array!\n");

            return NULL;
        }

        *pointerList = newPointerList;
        (*pointerList)[0].pointer = newPointerList;
    }

    strcpy((*pointerList)[*pointerListCount].name, name);
    (*pointerList)[*pointerListCount].pointer = newName.name;

    (*pointerListCount)++;

    if (*peopleListCount == *peopleListCapacity) {
        *peopleListCapacity += 5;

        struct People *newPeopleList = realloc(peopleList, *peopleListCapacity * sizeof(struct People));

        if (newPeopleList == NULL) {
            printf("Failed at expanding the people array!\n");

            (*pointerListCount)--;

            free((*pointerList)[*pointerListCount].pointer);

            (*pointerListCount)--;

            return NULL;
        }

        peopleList = newPeopleList;
        
        for (int i = 0; i < *pointerListCount; i++) {
            if (strcmp((*pointerList)[i].name, peopleListName) == 0) {
                (*pointerList)[i].pointer = peopleList;

                break;
            }
        }
    }

    peopleList[*peopleListCount] = newName;

    (*peopleListCount)++;

    return peopleList;
}
```


```c
void DisplayPeople(struct People *peopleList, int *peopleListCount, int *peopleListCapacity) {
    printf("People List Capacity: %d\n", *peopleListCapacity);
    printf("People List Count: %d\n\n", *peopleListCount);
    
    for (int i = 0; i < *peopleListCount; i++) {
        printf("Name %d: %s\n", i + 1, peopleList[i].name);
    }

    printf("\n\n");
}

void DisplayPointers(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity) {
    printf("Capacity: %d | Count: %d\n\n", *pointerListCapacity, *pointerListCount);
    
    for (int i = 0; i < *pointerListCount; i++) {
        printf("Pointer %d:\nName: %s | Address: %p\n", i + 1, pointerList[i].name, pointerList[i].pointer);
    }

    printf("\n\n");
}
```

Lastly, I made a function called “ProgramIsDone” to basically go through all pointers in the pointerList in reverse order and free everything:
```c
void ProgramIsDone(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity) {
    for (int i = *pointerListCount; i > 0; i--) {
        free(pointerList[i].pointer);

        (*pointerListCount)--;
    
        DisplayPointers(pointerList, pointerListCount, pointerListCapacity);
    }
}
```

**Reflection:**  
It was a good project, made me practice more the concepts I learned before in addition to learning void pointers and using double pointers!


## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct MemoryPointer {
    char name[50];
    void *pointer;
};

struct People {
    char *name;
};

struct MemoryPointer *CustomMainPointerListMalloc(char *listName, int *capacity, int *count);
void *CustomArrayMalloc(char *listName, int *capacity, unsigned long long size, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount);
struct People *AddNametoListWithCustomMalloc(struct People *peopleList, int *peopleListCapacity, int *peopleListCount, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount, char *peopleListName);

void ProgramIsDone(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity);

void DisplayPeople(struct People *peopleList, int *peopleListCount, int *peopleListCapacity);
void DisplayPointers(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity);

int main(int argc, char **argv) {
    // Pointer List
    int pointerListCapacity = 5;
    int pointerListCount = 0;
    struct MemoryPointer *pointerList = CustomMainPointerListMalloc("pointerList", &pointerListCapacity, &pointerListCount);

    if (pointerList == NULL) {
        printf("Creating the pointer array has failed!\n");

        return 1;
    }

    int peopleListCapacity = 5;
    int peopleListCount = 0;
    struct People *peopleList = CustomArrayMalloc("peopleList", &peopleListCapacity, sizeof(struct People), &pointerList, &pointerListCapacity, &pointerListCount);

    if (peopleList == NULL) {
        printf("Creating the people array has failed!\n");

        return 1;
    }

    // 11 names
    for (int i = 0; i < 11; i++) {
        peopleList = AddNametoListWithCustomMalloc(peopleList, &peopleListCapacity, &peopleListCount, &pointerList, &pointerListCapacity, &pointerListCount, "peopleList");
    }
    
    DisplayPeople(peopleList, &peopleListCount, &peopleListCapacity);
    DisplayPointers(pointerList, &pointerListCount, &pointerListCapacity);

    ProgramIsDone(pointerList, &pointerListCount, &pointerListCapacity);

    return 0;
}

struct MemoryPointer *CustomMainPointerListMalloc(char *listName, int *capacity, int *count) {
    struct MemoryPointer *tempArray = malloc(*capacity * sizeof(struct MemoryPointer));

    if (tempArray == NULL) {
        return NULL;
    }

    strcpy(tempArray[*count].name, listName);
    tempArray[*count].pointer = tempArray;
    (*count)++;

    return tempArray;
}

void *CustomArrayMalloc(char *listName, int *capacity, unsigned long long size, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount) {
    void *tempArray = malloc(*capacity * size);

    if (tempArray == NULL) {
        return NULL;
    }

    if (*pointerListCount == *pointerListCapacity) {
        *pointerListCapacity += 5;

        struct MemoryPointer *newPointerList = realloc(*pointerList, *pointerListCapacity * sizeof(struct MemoryPointer));

        if (newPointerList == NULL) {
            printf("Failed at expanding the pointer array!\n");

            return NULL;
        }

        *pointerList = newPointerList;
        (*pointerList)[0].pointer = newPointerList;
    }

    strcpy((*pointerList)[*pointerListCount].name, listName);
    (*pointerList)[*pointerListCount].pointer = tempArray;

    (*pointerListCount)++;

    return tempArray;
}

struct People *AddNametoListWithCustomMalloc(struct People *peopleList, int *peopleListCapacity, int *peopleListCount, struct MemoryPointer **pointerList, int *pointerListCapacity, int *pointerListCount, char *peopleListName) {
    char name[100];
    printf("Enter Name %d: ", *peopleListCount + 1);
    scanf(" %[^\n]", name);

    struct People newName;

    newName.name = malloc((strlen(name) + 1) * sizeof(char));

    if (newName.name == NULL) {
        printf("Name Malloc has Failed\n");

        return NULL;
    }

    strcpy(newName.name, name);

    if (*pointerListCount == *pointerListCapacity) {
        *pointerListCapacity += 5;

        struct MemoryPointer *newPointerList = realloc(*pointerList, *pointerListCapacity * sizeof(struct MemoryPointer));

        if (newPointerList == NULL) {
            printf("Failed at expanding the pointer array!\n");

            return NULL;
        }

        *pointerList = newPointerList;
        (*pointerList)[0].pointer = newPointerList;
    }

    strcpy((*pointerList)[*pointerListCount].name, name);
    (*pointerList)[*pointerListCount].pointer = newName.name;

    (*pointerListCount)++;

    if (*peopleListCount == *peopleListCapacity) {
        *peopleListCapacity += 5;

        struct People *newPeopleList = realloc(peopleList, *peopleListCapacity * sizeof(struct People));

        if (newPeopleList == NULL) {
            printf("Failed at expanding the people array!\n");

            (*pointerListCount)--;

            free((*pointerList)[*pointerListCount].pointer);

            (*pointerListCount)--;

            return NULL;
        }

        peopleList = newPeopleList;
        
        for (int i = 0; i < *pointerListCount; i++) {
            if (strcmp((*pointerList)[i].name, peopleListName) == 0) {
                (*pointerList)[i].pointer = peopleList;

                break;
            }
        }
    }

    peopleList[*peopleListCount] = newName;

    (*peopleListCount)++;

    return peopleList;
}

void DisplayPeople(struct People *peopleList, int *peopleListCount, int *peopleListCapacity) {
    printf("People List Capacity: %d\n", *peopleListCapacity);
    printf("People List Count: %d\n\n", *peopleListCount);
    
    for (int i = 0; i < *peopleListCount; i++) {
        printf("Name %d: %s\n", i + 1, peopleList[i].name);
    }

    printf("\n\n");
}

void DisplayPointers(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity) {
    printf("Capacity: %d | Count: %d\n\n", *pointerListCapacity, *pointerListCount);
    
    for (int i = 0; i < *pointerListCount; i++) {
        printf("Pointer %d:\nName: %s | Address: %p\n", i + 1, pointerList[i].name, pointerList[i].pointer);
    }

    printf("\n\n");
}

void ProgramIsDone(struct MemoryPointer *pointerList, int *pointerListCount, int *pointerListCapacity) {
    for (int i = *pointerListCount; i > 0; i--) {
        free(pointerList[i].pointer);

        (*pointerListCount)--;
    
        DisplayPointers(pointerList, pointerListCount, pointerListCapacity);
    }
}
```