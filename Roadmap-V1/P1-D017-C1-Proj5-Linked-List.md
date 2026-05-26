# Phase 1 - Day 17: Monday 27 April 2026
## Summary:
Made the 5th project of Category 1 which is a Linked List

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Continuing with the projects, today’s project is making a Linked List. I’m going to make an int Linked List with the following functions: Create to create a node, Search to search a node in the list, Display to print all nodes in the list, Push to push a node to the list, Pop to pop a node from the list, and Delete to free the whole list.

Starting with the node structure, pretty simple just a struct with an int and a pointer to the next node:
```c
struct Node {
    int data;
    struct Node *nextNode;
};
```

Next, the Create function. It takes an integer as a parameter, creates a node based on it and returns it:
```c
struct Node *CreateNode(int data) {
    struct Node *newNode = (struct Node*)malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to create a node!\n\n");

        return NULL;
    }

    (*newNode).data = data;
    (*newNode).nextNode = NULL;

    return newNode;
}
```

Now, the Search function, it takes the head node and an integer as parameters, loops through the nodes searching for a node that contains that integer and returns that node if found:
```c
struct Node *SearchNode(struct Node *headNode, int data) {
    struct Node *currentNode = headNode;

    while (currentNode != NULL) {
        if ((*currentNode).data == data) {
            printf("Node Found\nPointer: %p\n\n", currentNode);

            return currentNode;
        }

        currentNode = (*currentNode).nextNode;
    }

    printf("Node was not found\n\n");
    return NULL;
}
```

Next, the Display function. It takes the head node as a parameter and loops through all the nodes and prints each one of them:
```c
void Display(struct Node *headNode) {
    struct Node *currentNode = headNode;

    if (currentNode == NULL) {
        printf("List is empty!\n\n");

        return;
    }

    int count = 1;
    while (currentNode != NULL) {
        printf("Node %d:\n", count);
        printf("Node Data: %d\n", (*currentNode).data);
        printf("Next Node Pointer: %p\n\n", (*currentNode).nextNode);

        currentNode = (*currentNode).nextNode;
        count++;
    }
}
```

Next, the Push function, it takes the head and a node as parameters, the next node of the new node will be the current head, and with that the new head will be the new node. Now I made this function in two ways, the first is like this:
```c
struct Node *PushNode(struct Node *headNode, struct Node *newNode) {
    if (newNode == NULL) {
        printf("You can't add a null node\n\n");

        return headNode;
    }
    
    (*newNode).nextNode = headNode;

    return newNode;
}
```

Then I thought about making the head changes automatically through the function instead of returning it to the user, and achieving this was by making the headNode a double pointer:
```c
void PushNode(struct Node **headNode, struct Node *newNode) {
    if (newNode == NULL) {
        printf("You can't add a null node\n\n");

        return;
    }
    
    (*newNode).nextNode = *headNode;

    *headNode = newNode;
}
```

Next, the Pop function, it takes the head node as a parameter. Checks if the list is already empty, if it’s not, then it creates a temp node that equals the current head node, moves the head to the next node, then frees the temp node. Without the temp node, if I free the head node first, I lose access to all the rest of the nodes. If I move to the next node first, I can’t go back to free the head node since it’s now the second node. I used the same double pointer technique for this function too:
```c
void PopNode(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The List is already empty!\n\n");

        return;
    }

    struct Node *temp = *headNode;

    *headNode = (*(*headNode)).nextNode;

    free(temp);

    printf("Node has been popped successfully!\n\n");
}
```

Finally, the Delete function. It takes the head node as a parameter. It loops through the nodes from the head, it takes the address of the next pointer, then free the current one! All the way to the end:
```c
void DeleteList(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The list is already empty!\n\n");

        return;
    }

    struct Node *currentNode = *headNode;
    while (currentNode != NULL) {
        struct Node *nextNode = (*currentNode).nextNode;

        free(currentNode);

        currentNode = nextNode;
    }

    *headNode = NULL;
}
```

**Reflection:**  
Surprisingly, this was a very easy and straightforward project to make! I don’t know if it’s the project or just me getting better! But it was a fun project to do anyway! It made me solidify my understanding, and practice double pointers more!

## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node *nextNode;
};

struct Node *CreateNode(int data);
struct Node *SearchNode(struct Node *headNode, int data);
void Display(struct Node *headNode);
// struct Node *PushNode(struct Node *headNode, struct Node *newNode);
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

struct Node *CreateNode(int data) {
    struct Node *newNode = (struct Node*)malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to create a node!\n\n");

        return NULL;
    }

    (*newNode).data = data;
    (*newNode).nextNode = NULL;

    return newNode;
}

struct Node *SearchNode(struct Node *headNode, int data) {
    struct Node *currentNode = headNode;

    while (currentNode != NULL) {
        if ((*currentNode).data == data) {
            printf("Node Found\nPointer: %p\n\n", currentNode);

            return currentNode;
        }

        currentNode = (*currentNode).nextNode;
    }

    printf("Node was not found\n\n");
    return NULL;
}

void Display(struct Node *headNode) {
    struct Node *currentNode = headNode;

    if (currentNode == NULL) {
        printf("List is empty!\n\n");

        return;
    }

    int count = 1;
    while (currentNode != NULL) {
        printf("Node %d:\n", count);
        printf("Node Data: %d\n", (*currentNode).data);
        printf("Next Node Pointer: %p\n\n", (*currentNode).nextNode);

        currentNode = (*currentNode).nextNode;
        count++;
    }
}

// struct Node *PushNode(struct Node *headNode, struct Node *newNode) {
//     if (newNode == NULL) {
//         printf("You can't add a null node\n\n");

//         return headNode;
//     }
    
//     (*newNode).nextNode = headNode;

//     return newNode;
// }

void PushNode(struct Node **headNode, struct Node *newNode) {
    if (newNode == NULL) {
        printf("You can't add a null node\n\n");

        return;
    }
    
    (*newNode).nextNode = *headNode;

    *headNode = newNode;
}

void PopNode(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The List is already empty!\n\n");

        return;
    }

    struct Node *temp = *headNode;

    *headNode = (*(*headNode)).nextNode;

    free(temp);

    printf("Node has been popped successfully!\n\n");
}

void DeleteList(struct Node **headNode) {
    if (headNode == NULL || *headNode == NULL) {
        printf("The list is already empty!\n\n");

        return;
    }

    struct Node *currentNode = *headNode;
    while (currentNode != NULL) {
        struct Node *nextNode = (*currentNode).nextNode;

        free(currentNode);

        currentNode = nextNode;
    }

    *headNode = NULL;
}
```