# Phase 1 - Day 19: Sunday 3 May 2026
## Summary:
Made the 7th project of Category 1 which is Generic Hash Table!

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Before starting, maybe you noticed that currently, several days might pass between each documentation day! That’s because I’m taking the Malware Analysis Skill Path course from LetsDefend, so I’m splitting my time between here and there! While I’m documenting my course learnings, I’m not publishing them to GitHub! Maybe later I don’t know!

Okay now back to work! Today, I’m continuing with the projects. Today's project is “Generic Hash Table”. I don’t know anything about Hash Tables so I started searching online to learn about it. First I watched this video and got the main idea of it. So it’s kind of like a dictionary, storing items in key, value pairs. But the storing logic is unique. First it takes the key and hash it in some hashing algorithm, then does a math calculation to know in which index in the table to store it. When two entries turn out in the same cell, this is called a collision, and I found 2 implementations to handle it. First is the one discussed in the YouTube video, which is about transforming the cell to a LinkedList which is super inefficient if the table got transformed into multiple LinkedLists where each list is very long! In that implementation, the table keeps its size! The second implementation which is the one I’ll use is to basically find the next empty slot! And the table expands overtime when the count of items reaches a certain threshold. I’m going to make the threshold 50% in this project!

For implementation, I’ll make 2 structs, one for the Entry itself which holds the key and the value as members. The other struct is for the table itself which will contain a list of entries, count, and capacity as members. And for the table’s functions I’m going to implement Initialize, Hash, Insert, Search, Delete, Display, and Free functions.

Starting with the structs, since the table must be generic, the key and value will be void pointers. In addition, from the research I did, I learned that I can’t just NULL the cell if it gets deleted, so I need to keep track of each cell! For that reason I added an enum called CellStatus to track each cell. Then I made the table struct. It has 3 members and 2 functions, the members are list of entries, count, and capacity. The functions are hash and compare functions! Structs in C can’t have functions, but I’ve learned recently that you can put function pointers in the struct as a workaround! I did these 2 functions since we’ll be working with generic values, so hashing and comparing are helper functions. Final code of the structs:
```c
enum CellStatus {
    empty,
    occupied,
    deleted
};

struct Entry {
    void *key;
    void *value;
    enum CellStatus status;
};

struct HashTable {
    struct Entry *entryList;
    int count;
    int capacity;

    unsigned int (*HashFunction)(void *key);
    int (*CompareFunction)(void *key1, void *key2);
};
```

Next, the Initialize function! The function will take the capacity as a parameter and first initialize the HashTable, then it will check if the malloc fails or not. Second, the function needs to initialize the entry list where the table will store the entries, and again if a failure check will be in place! Then table values will be set (capacity and count), then finally, each entry of the list will be initialized to NULL and empty. Full Code:
```c
struct HashTable *InitializeTable(int capacity) {
    struct HashTable *newTable = malloc(sizeof(struct HashTable));

    if (newTable == NULL) {
        printf("Initializing the Table has failed!\n\n");
        
        return NULL;
    }

    (*newTable).entryList = malloc(capacity * sizeof(struct Entry));

    if ((*newTable).entryList == NULL) {
        printf("Initializing the Entries has failed!\n\n");

        free(newTable);

        return NULL;
    }

    (*newTable).capacity = capacity;
    (*newTable).count = 0;

    // (*newTable).CompareFunction = ...
    // (*newTable).HashFunction = ...

    for (int i = 0; i < capacity; i++) {
        (*newTable).entryList[i].key = NULL;
        (*newTable).entryList[i].value = NULL;
        (*newTable).entryList[i].status = empty;
    }

    return newTable;
}
```
The compare and hash functions will be assigned once I do them, for now I just commented them

Next, the Insert Function, it will take a key and a value as parameters. First, I need to check if the key is already in the table! I think I can do it in two ways, the first is to just iterate the list to see if it is there. The second is to put the key to the hash function and I presume that it will generate the same index every time, so I can directly go to the index and check if there is an item there. If the key already exists, the value of it will be overwritten! If it’s not, then it will be stored in this cell. If the cell is occupied by a different ID, then this will be a collision that will be handled by iterating forward from the cell’s location until a free cell is located! And before all that, I need to check if 50% of the table is full or not, if not just continue, and if yes then I need to realloc with * 2 size.

For this function, I made several helper functions: ResizeTable, HashFunction, and CompareFunction.

For resizing, it just takes the table as a parameter.. I was going to use double pointers for this one, but after researching, I learned that double pointers are only used if I want to change the thing itself (the HashTable itself as an object), and 1 pointer can be used to change anything inside the object (the entrylist in this case). So I used 1 pointer. The function keeps the old capacity and list temporarily, then doubles the capacity and malloc a new list, then checks if the malloc fails. If it fails it returns a failing string and the insertion will fail with it. If malloc succeeded, we initialized the new list, reset the count, then inserted all old data again using the same Insert function, finally freeing the old list.

For Hashing, I converted the void to a string and used a string hashing algorithm. And for comparison, I just converted the keys into a string and  compared them and returned 0 if they match.

Full Code of the 4 functions:
```c
void Insert(struct HashTable *table, void *key, void *value) {
    // resizing if necessary
    if ((double)(*table).count / (*table).capacity >= 0.5) {
        int result = ResizeTable(table);

        if (result == 1) {
            printf("Failed at inserting!\n\n");

            return;
        }
    }

    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;

    // searching if the key is already in the table
    while ((*table).entryList[index].status == occupied) {
        if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
            (*table).entryList[index].value = value;

            return;
        }

        index = (index + 1) % (*table).capacity;
    }

    // Inserting new item
    (*table).entryList[index].key = key;
    (*table).entryList[index].value = value;
    (*table).entryList[index].status = occupied;
    (*table).count++;
}

int ResizeTable(struct HashTable *table) {
    int oldCapacity = (*table).capacity;
    struct Entry *oldList = (*table).entryList;

    // doubling the capacity and allocate the new memory
    
    int newCapacity = (*table).capacity * 2;
    struct Entry *newList = malloc(newCapacity * sizeof(struct Entry));

    if (newList == NULL) {
        printf("Failed to expand the table!\n\n");

        return 1;
    }

    (*table).capacity = newCapacity;
    (*table).entryList = newList;

    // initializing the new list to empty
    for (int i = 0; i < (*table).capacity; i++) {
        (*table).entryList[i].value = NULL;
        (*table).entryList[i].key = NULL;
        (*table).entryList[i].status = empty;
    }

    // resetting the count
    (*table).count = 0;

    // Using the Insert function to insert the old values to the new list
    for (int i = 0; i < oldCapacity; i++) {
        if (oldList[i].status == occupied) {
            Insert(table, oldList[i].key, oldList[i].value);
        }
    }

    // freeing the old memory
    free(oldList);

    return 0;
}

unsigned int HashFunction(void *key) {
    unsigned char *str = (unsigned char *)key;
    unsigned int hash = 5381;
    int c;

    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c;
    }

    return hash;
}

int CompareFunction(void *key1, void *key2) {
    char *convertedKey1 = (char *)key1;
    char *convertedKey2 = (char *)key2;

    return strcmp(convertedKey1, convertedKey2);
}
```

Next, the Search function. From what I learned, Hash Tables are O(1) since they don’t search from start to finish (that will make it O(n)). The function takes the key of the item, hashes it, goes to the index of it, and starts searching from there. Usually this should work instantly, which is O(1), but if the cell is already occupied, it iterates until it finds it. Code:
```c
void Search(struct HashTable *table, void *key) {
    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;
    int startIndex = index;

    while ((*table).entryList[index].status != empty) {
        if ((*table).entryList[index].status == occupied) {
            if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
                printf("Entry Found:\n");
                printf("Position: %d\n", index + 1);
                printf("Key: %s\n", (char *)key);
                printf("Value: %s\n\n", (char *)(*table).entryList[index].value);
                
                return;
            }
        }

        index = (index + 1) % (*table).capacity;

        // safety check
        if (index == startIndex) {
            break;
        }
    }

    printf("Entry wasn't found!\n\n");
}
```

Next, the Delete function, it uses the same Search mechanisms, but instead of printing the entry, it just change its status to deleted and decrease the count by 1, code:
```c
void Delete(struct HashTable *table, void *key) {
    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;
    int startIndex = index;

    while ((*table).entryList[index].status != empty) {
        if ((*table).entryList[index].status == occupied) {
            if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
                (*table).entryList[index].status = deleted;
                (*table).count--;

                printf("Entry has been deleted successfully!\n\n");

                return;
            }
        }

        index = (index + 1) % (*table).capacity;
        if (index == startIndex) {
            break;
        }
    }

    printf("Entry wasn't found!\n\n");
}
```

Next the Display function, it just iterates over the list and prints the details of each element. Code:
```c
void Display(struct HashTable *table) {
    if (table == NULL) {
        printf("Table has not been initialized!\n\n");

        return;
    }
    
    printf("Table Content: (Size: %d out of %d)\n\n", (*table).count, (*table).capacity);
    for (int i = 0; i < (*table).capacity; i++) {
        if ((*table).entryList[i].status == occupied) {
            printf("Entry Number %d: Key: %s - Value: %s\n\n", i, (char *)(*table).entryList[i].key, (char *)(*table).entryList[i].value);
        }
        else if ((*table).entryList[i].status == deleted) {
            printf("Entry Number %d: Deleted!\n\n", i);
        }
        else {
            printf("Entry Number %d: Empty!\n\n", i);
        }
    }
}
```

Finally, the FreeHahsTable function, first it frees the list, then the table! Code:
```c
void FreeHashTable(struct HashTable *table) {
    if (table == NULL) {
        printf("Table has not been initialized!\n\n");

        return;
    }

    if ((*table).entryList != NULL) {
        free((*table).entryList);
    }

    free(table);
}
```

**Reflection:**  
This was a very interesting and fun project to implement! Learned about Hash Tables and practiced it in addition to many C features!

## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

enum CellStatus {
    empty,
    occupied,
    deleted
};

struct Entry {
    void *key;
    void *value;
    enum CellStatus status;
};

struct HashTable {
    struct Entry *entryList;
    int count;
    int capacity;

    unsigned int (*HashFunction)(void *key);
    int (*CompareFunction)(void *key1, void *key2);
};

struct HashTable *InitializeTable(int capacity);
void Insert(struct HashTable *table, void *key, void *value);
int ResizeTable(struct HashTable *table);
unsigned int HashFunction(void *key);
int CompareFunction(void *key1, void *key2);
void Search(struct HashTable *table, void *key);
void Delete(struct HashTable *table, void *key);
void Display(struct HashTable *table);
void FreeHashTable(struct HashTable *table);

int main(int argc, char **argv) {
    int initialCapacity = 4;
    struct HashTable *table = InitializeTable(initialCapacity);

    if (table == NULL) {
        return 1;
    }

    Insert(table, "Key 1", "Value 1");
    Display(table);

    Insert(table, "Key 2", "Value 2");
    Insert(table, "Key 3", "Value 3");
    Insert(table, "Key 4", "Value 4");
    Insert(table, "Key 5", "Value 5");
    Display(table);

    Insert(table, "Key 3", "New Value 3");
    Display(table);

    Search(table, "Key 1");
    Search(table, "Key 6");

    Delete(table, "Key 4");
    Search(table, "Key 4");

    Display(table);

    FreeHashTable(table);
    table = NULL;
    Display(table);

    return 0;
}

struct HashTable *InitializeTable(int capacity) {
    struct HashTable *newTable = malloc(sizeof(struct HashTable));

    if (newTable == NULL) {
        printf("Initializing the Table has failed!\n\n");
        
        return NULL;
    }

    (*newTable).entryList = malloc(capacity * sizeof(struct Entry));

    if ((*newTable).entryList == NULL) {
        printf("Initializing the Entries has failed!\n\n");

        free(newTable);

        return NULL;
    }

    (*newTable).capacity = capacity;
    (*newTable).count = 0;

    (*newTable).CompareFunction = CompareFunction;
    (*newTable).HashFunction = HashFunction;

    for (int i = 0; i < capacity; i++) {
        (*newTable).entryList[i].key = NULL;
        (*newTable).entryList[i].value = NULL;
        (*newTable).entryList[i].status = empty;
    }

    return newTable;
}

void Insert(struct HashTable *table, void *key, void *value) {
    // resizing if necessary
    if ((double)(*table).count / (*table).capacity >= 0.5) {
        int result = ResizeTable(table);

        if (result == 1) {
            printf("Failed at inserting!\n\n");

            return;
        }
    }

    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;

    // searching if the key is already in the table
    while ((*table).entryList[index].status == occupied) {
        if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
            (*table).entryList[index].value = value;

            return;
        }

        index = (index + 1) % (*table).capacity;
    }

    // Inserting new item
    (*table).entryList[index].key = key;
    (*table).entryList[index].value = value;
    (*table).entryList[index].status = occupied;
    (*table).count++;
}

int ResizeTable(struct HashTable *table) {
    int oldCapacity = (*table).capacity;
    struct Entry *oldList = (*table).entryList;

    // doubling the capacity and allocate the new memory
    
    int newCapacity = (*table).capacity * 2;
    struct Entry *newList = malloc(newCapacity * sizeof(struct Entry));

    if (newList == NULL) {
        printf("Failed to expand the table!\n\n");

        return 1;
    }

    (*table).capacity = newCapacity;
    (*table).entryList = newList;

    // initializing the new list to empty
    for (int i = 0; i < (*table).capacity; i++) {
        (*table).entryList[i].value = NULL;
        (*table).entryList[i].key = NULL;
        (*table).entryList[i].status = empty;
    }

    // resetting the count
    (*table).count = 0;

    // Using the Insert function to insert the old values to the new list
    for (int i = 0; i < oldCapacity; i++) {
        if (oldList[i].status == occupied) {
            Insert(table, oldList[i].key, oldList[i].value);
        }
    }

    // freeing the old memory
    free(oldList);

    return 0;
}

unsigned int HashFunction(void *key) {
    unsigned char *str = (unsigned char *)key;
    unsigned int hash = 5381;
    int c;

    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c;
    }

    return hash;
}

int CompareFunction(void *key1, void *key2) {
    char *convertedKey1 = (char *)key1;
    char *convertedKey2 = (char *)key2;

    return strcmp(convertedKey1, convertedKey2);
}

void Search(struct HashTable *table, void *key) {
    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;
    int startIndex = index;

    while ((*table).entryList[index].status != empty) {
        if ((*table).entryList[index].status == occupied) {
            if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
                printf("Entry Found:\n");
                printf("Position: %d\n", index + 1);
                printf("Key: %s\n", (char *)key);
                printf("Value: %s\n\n", (char *)(*table).entryList[index].value);
                
                return;
            }
        }

        index = (index + 1) % (*table).capacity;

        // safety check
        if (index == startIndex) {
            break;
        }
    }

    printf("Entry wasn't found!\n\n");
}

void Delete(struct HashTable *table, void *key) {
    unsigned int hashValue = (*table).HashFunction(key);
    int index = hashValue % (*table).capacity;
    int startIndex = index;

    while ((*table).entryList[index].status != empty) {
        if ((*table).entryList[index].status == occupied) {
            if ((*table).CompareFunction((*table).entryList[index].key, key) == 0) {
                (*table).entryList[index].status = deleted;
                (*table).count--;

                printf("Entry has been deleted successfully!\n\n");

                return;
            }
        }

        index = (index + 1) % (*table).capacity;
        if (index == startIndex) {
            break;
        }
    }

    printf("Entry wasn't found!\n\n");
}

void Display(struct HashTable *table) {
    if (table == NULL) {
        printf("Table has not been initialized!\n\n");

        return;
    }
    
    printf("Table Content: (Size: %d out of %d)\n\n", (*table).count, (*table).capacity);
    for (int i = 0; i < (*table).capacity; i++) {
        if ((*table).entryList[i].status == occupied) {
            printf("Entry Number %d: Key: %s - Value: %s\n\n", i, (char *)(*table).entryList[i].key, (char *)(*table).entryList[i].value);
        }
        else if ((*table).entryList[i].status == deleted) {
            printf("Entry Number %d: Deleted!\n\n", i);
        }
        else {
            printf("Entry Number %d: Empty!\n\n", i);
        }
    }
}

void FreeHashTable(struct HashTable *table) {
    if (table == NULL) {
        printf("Table has not been initialized!\n\n");

        return;
    }

    if ((*table).entryList != NULL) {
        free((*table).entryList);
    }

    free(table);
}
```