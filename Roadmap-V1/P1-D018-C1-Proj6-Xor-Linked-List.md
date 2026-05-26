# Phase 1 - Day 18: Wednesday 29 April 2026
## Summary:
Made the 6th project of Category 1 which is XOR Linked List, which is essentially a Doubled Linked List!

## Note:
Documentation first, Full code at the bottom.

## Documentation:
The next project I have is the XOR Linked List project! It is essentially a doubly linked list but instead of saving both next and previous pointers, it saves the xor of them in a single variable to save memory!

I’ll be using the code from the previous project (the normal LinkedList) and update it!

Starting with the Node struct, I renamed the node pointer variable from nextNode to nodePointerXored:
```c
struct Node {
    int data;
    struct Node *nodePointerXored;
};
```

In C, pointers can’t be Xored directly, they need to be converted to unsigned int pointers first! So I made a helper function named XorNodePointers to help with that!
```c
struct Node *XorNodePointers(struct Node *pointerA, struct Node *pointerB) {
    return (struct Node*)((uintptr_t)pointerA ^ (uintptr_t)pointerB);
}
```

Next, the CreateNode function, there is nothing much to do here since we’re just creating the node, so I just fixed the naming of the pointer to reflect the new one:
```c
struct Node *CreateNode(int data) {
    struct Node *newNode = (struct Node*)malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to create a node!\n\n");

        return NULL;
    }

    (*newNode).data = data;
    (*newNode).nodePointerXored = NULL;

    return newNode;
}
```

Next, SearchNode. The core logic can stay the same, no need to do something fancy like starting from the middle for example. The core logic is fine, only the traversal logic (getting the next node) is what needs to change. Since the pointer now holds a xor of the previous and next, to be able to get the next, I should keep track of both the previous and next. To get the previous we need to xor the current with the next. To get the next we need to xor the current with the previous. You might be wondering, but if each one depends on the other, how can we do it? The answer is simple since at the start of the list, the previous is NULL. so we can determine the next.. And based on that, the rest is easy to get! So the new function now looks like this:
```c
struct Node *SearchNode(struct Node *headNode, int data) {
    struct Node *currentNode = headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    while (currentNode != NULL) {
        if ((*currentNode).data == data) {
            printf("Node Found\nPointer: %p\n\n", currentNode);

            return currentNode;
        }

        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);
        
        previousNode = currentNode;
        currentNode = nextNode;
    }

    printf("Node was not found\n\n");
    return NULL;
}
```

Next, the Display function. I think its case is the same as the Search function, the core logic is fine, just the traversal logic needs to be updated:
```c
void Display(struct Node *headNode) {
    struct Node *currentNode = headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    if (currentNode == NULL) {
        printf("List is empty!\n\n");

        return;
    }

    int count = 1;
    while (currentNode != NULL) {
        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);

        printf("Node %d:\n", count);
        printf("Node Data: %d\n\n", (*currentNode).data);
        printf("Current Xored Node Pointer: %p\n", (*currentNode).nodePointerXored);
        printf("Previous Node Pointer: %p\n", previousNode);
        printf("Next Node Pointer: %p\n", nextNode);

        previousNode = currentNode;
        currentNode = nextNode;

        count++;
    }
}
```

Next, the PushNode function, Now this function will need a lot of changes:
- First we need to calculate the new node’s prev and next
- Then we need to xor them and store them to the new node
- Next we need to update the current hea’s prev node

Getting the new node’s prev is easy, it’s simply NULL since it will be the new head. And the next is also simple since it is the current head!

The current head’s next is basically the xored pointer since NULL ^ Something in C is something itself. So we can directly use it! Getting its previous is also simple since simply it is the new node! With that, this is the new code:
```c
void PushNode(struct Node **headNode, struct Node *newNode) {
    if (newNode == NULL) {
        printf("You can't add a null node\n\n");

        return;
    }
    
    // Null since it is the new head
    struct Node *newNodepreviousNodePointer = NULL;

    // to point at the current head
    struct Node *newNodeNextNodePointer = *headNode;

    // xor the pointers together to get the new node.. which in C NULL ^ something is something itself
    (*newNode).nodePointerXored = XorNodePointers(newNodepreviousNodePointer, newNodeNextNodePointer);

    if (*headNode != NULL) {
        // Since this is the current head, its head must be null
        // As mentioned before, NULL ^ Something is something itself
        // so no need to XOR
        struct Node *currentNodeNextPointer = (*(*headNode)).nodePointerXored;

        // new prev for the current head is the newNode
        (*(*headNode)).nodePointerXored = XorNodePointers(newNode, currentNodeNextPointer);
    }

    *headNode = newNode;
}
```

Next, the PopNode function. For this function, we need to go to the next node which is simple since the current prev is NULL since it is the head so the already xored pointer is already the next! So that’s done. And one more thing is the new head’s prev needs to be set to NULL:
```c
void PopNode(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The List is already empty!\n\n");

        return;
    }

    struct Node *oldHead = *headNode;
    struct Node *nextNode = XorNodePointers(NULL, (*oldHead).nodePointerXored);

    if (nextNode != NULL) {
        // getting the new head's next
        struct Node *newHeadNextNode = XorNodePointers(oldHead, (*nextNode).nodePointerXored);

        // storing it again but now with NULL
        (*nextNode).nodePointerXored = XorNodePointers(NULL, newHeadNextNode);
    }
    
    free(oldHead);

    *headNode = nextNode;

    printf("Node has been popped successfully!\n\n");
}
```

Now the DeleteList function, same as the Search and Display functions, the core logic is fine, just the traversal needs to be edited:
```c
void DeleteList(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The list is already empty!\n\n");

        return;
    }

    struct Node *currentNode = *headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    while (currentNode != NULL) {
        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);

        free(currentNode);

        previousNode = currentNode;
        currentNode = nextNode;
    }

    *headNode = NULL;
    printf("List deleted Successfully!\n\n");
}
```

**Reflection:**  
This was a fun project to do, made me practice how to work with Xor and pointers!

## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

struct Node {
    int data;
    struct Node *nodePointerXored;
};

struct Node *XorNodePointers(struct Node *pointerA, struct Node *pointerB);
struct Node *CreateNode(int data);
struct Node *SearchNode(struct Node *headNode, int data);
void Display(struct Node *headNode);
void PushNode(struct Node **headNode, struct Node *newNode);
void PopNode(struct Node **headNote);
void DeleteList(struct Node **headNote);

int main(int argc, char **argv) {
    struct Node *headNode = NULL;

    printf("Testing Displaying Empty Lists\n");
    Display(headNode);

    printf("Creating and pushing nodes\n");
    PushNode(&headNode, CreateNode(1));
    PushNode(&headNode, CreateNode(2));
    PushNode(&headNode, CreateNode(3));
    PushNode(&headNode, CreateNode(4));
    PushNode(&headNode, CreateNode(5));
    PushNode(&headNode, CreateNode(6));
    PushNode(&headNode, CreateNode(7));
    PushNode(&headNode, CreateNode(8));
    PushNode(&headNode, CreateNode(9));
    PushNode(&headNode, CreateNode(10));

    printf("Displaying the new list\n");
    Display(headNode);

    printf("Testing Search\n");
    SearchNode(headNode, 10);
    SearchNode(headNode, 5);
    SearchNode(headNode, 20);

    printf("Testing Pop\n");
    PopNode(&headNode);
    Display(headNode);

    printf("Testing the Delete function\n");
    DeleteList(&headNode);
    Display(headNode);

    return 0;
}

struct Node *XorNodePointers(struct Node *pointerA, struct Node *pointerB) {
    return (struct Node*)((uintptr_t)pointerA ^ (uintptr_t)pointerB);
}

struct Node *CreateNode(int data) {
    struct Node *newNode = (struct Node*)malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to create a node!\n\n");

        return NULL;
    }

    (*newNode).data = data;
    (*newNode).nodePointerXored = NULL;

    return newNode;
}

struct Node *SearchNode(struct Node *headNode, int data) {
    struct Node *currentNode = headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    while (currentNode != NULL) {
        if ((*currentNode).data == data) {
            printf("Node Found\nPointer: %p\n\n", currentNode);

            return currentNode;
        }

        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);
        
        previousNode = currentNode;
        currentNode = nextNode;
    }

    printf("Node was not found\n\n");
    return NULL;
}

void Display(struct Node *headNode) {
    struct Node *currentNode = headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    if (currentNode == NULL) {
        printf("List is empty!\n\n");

        return;
    }

    int count = 1;
    while (currentNode != NULL) {
        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);

        printf("Node %d:\n", count);
        printf("Node Data: %d\n\n", (*currentNode).data);
        printf("Current Xored Node Pointer: %p\n", (*currentNode).nodePointerXored);
        printf("Previous Node Pointer: %p\n", previousNode);
        printf("Next Node Pointer: %p\n", nextNode);

        previousNode = currentNode;
        currentNode = nextNode;

        count++;
    }
}

void PushNode(struct Node **headNode, struct Node *newNode) {
    if (newNode == NULL) {
        printf("You can't add a null node\n\n");

        return;
    }
    
    // Null since it is the new head
    struct Node *newNodepreviousNodePointer = NULL;

    // to point at the current head
    struct Node *newNodeNextNodePointer = *headNode;

    // xor the pointers together to get the new node.. which in C NULL ^ something is something itself
    (*newNode).nodePointerXored = XorNodePointers(newNodepreviousNodePointer, newNodeNextNodePointer);

    if (*headNode != NULL) {
        // Since this is the current head, its head must be null
        // As mentioned before, NULL ^ Something is something itself
        // so no need to XOR
        struct Node *currentNodeNextPointer = (*(*headNode)).nodePointerXored;

        // new prev for the current head is the newNode
        (*(*headNode)).nodePointerXored = XorNodePointers(newNode, currentNodeNextPointer);
    }

    *headNode = newNode;
}

void PopNode(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The List is already empty!\n\n");

        return;
    }

    struct Node *oldHead = *headNode;
    struct Node *nextNode = XorNodePointers(NULL, (*oldHead).nodePointerXored);

    if (nextNode != NULL) {
        // getting the new head's next
        struct Node *newHeadNextNode = XorNodePointers(oldHead, (*nextNode).nodePointerXored);

        // storing it again but now with NULL
        (*nextNode).nodePointerXored = XorNodePointers(NULL, newHeadNextNode);
    }
    
    free(oldHead);

    *headNode = nextNode;

    printf("Node has been popped successfully!\n\n");
}

void DeleteList(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The list is already empty!\n\n");

        return;
    }

    struct Node *currentNode = *headNode;
    struct Node *previousNode = NULL;
    struct Node *nextNode;

    while (currentNode != NULL) {
        nextNode = XorNodePointers(previousNode, (*currentNode).nodePointerXored);

        free(currentNode);

        previousNode = currentNode;
        currentNode = nextNode;
    }

    *headNode = NULL;
    printf("List deleted Successfully!\n\n");
}
```