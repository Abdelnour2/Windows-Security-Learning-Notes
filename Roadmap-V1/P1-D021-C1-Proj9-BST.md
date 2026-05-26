# Phase 1 - Day 21: Thursday 14 May 2026
## Summary:
Made the 9th project of Category 1 which is Binary Search Tree!

## Note:
Documentation first, Full code at the bottom.

## Documentation:
Today’s project is Binary Search Tree! I learned about BSTs when I was doing my bachelor's. Binary Search Tree is a data structure that starts at a top parent node called the root node. And each node can have 2 children nodes (from the name Binary)! The rule of BST is that the left node always must be smaller than the parent node. And the right node always must be larger than the parent node.

Let’s take an example to visualize the tree. Let’s say we started with the number 50. The tree is empty so 50 will be the root. Then comes number 30. 30 is smaller than 50, then it will be the left child node of 50. Then comes 70, it is larger than 50, so it will be the right child node of 50. Then comes 20. 20 is smaller than 50, so it needs to go to the left side, but 50 already has a left child node which is 30, so 20 must be a child to 30. Since 20 is smaller than 30, it will be the left child of 30. And so on..

Binary Search Trees allow us to search for an item in half the time (or even less)! It is O(log n) in time complexity since we don’t need to search the full tree to find a number. Let’s say we want to search for the number 20. First the root, 50. 20 is smaller than 50 so we’ll look at the left side of the tree, removing the need to search in the entire right side of the tree. Then we reach the node 30, 20 is smaller than 30 so we’ll need to search the left side of 30, removing the need to search for the right side of it.

After visualizing the BST, now it’s time for implementation. First I’ll define a node struct which will store the data itself as an integer, then the pointers to the left and right nodes below it. Then, I’ll make a function to initialize a node. Code:
```c
struct Node {
    int data;
    struct Node *leftNode;
    struct Node *rightNode;
};
```

```c
struct Node *InitializeNode(int data) {
    struct Node *newNode = malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to Allocate Memory for the Node\n\n");
        
        return NULL;
    }

    (*newNode).data = data;
    (*newNode).leftNode = NULL;
    (*newNode).rightNode = NULL;

    return newNode;
}
```

Next is the Insert function. It will take two arguments, the root node, and the new node. It will look at the new node’s data and compare it to its data. If it’s smaller, it will pass it to the left side, if it’s larger, it will pass it to the right side. If the parent node is already full, then the function will loop recursively assigning the parent node to be the next logical one, and the new node will stay the same. But what happens when the new node is the same? What if the tree already has that value? I don’t remember this case. After a quick research, it turned out that there is no unified solution to it, some just discard it, some add it to the left side making it less or equal. I’ll just discard it. This is the first attempt of the function:
```c
void InsertNode(struct Node *parentNode, struct Node *newNode) {
    // Case 1 of 4: New Node is NULL
    if (newNode == NULL) {
        printf("Cannot Insert an Empty Node\n\n");

        return;
    }
    // Case 2 of 4: New Node is Smaller than the Parent
    else if ((*newNode).data < (*parentNode).data) {
        // Sub-Case 1 of 2: Parent's Left Side is Empty 
        if ((*parentNode).leftNode == NULL) {
            (*parentNode).leftNode = newNode;
        }
        // Sub-Case 2 of 2: Parent's Left Side is Full
        else {
            InsertNode((*parentNode).leftNode, newNode);
        }
    }
    // Case 3 of 4: New Node is Bigger than the Parent 
    else if ((*newNode).data > (*parentNode).data) {
        // Sub-Case 1 of 2: Parent's Right Side is Empty
        if ((*parentNode).rightNode == NULL) {
            (*parentNode).rightNode = newNode;
        }
        // Sub-Case 2 of 2: Parent's Right Side is Full
        else {
            InsertNode((*parentNode).rightNode, newNode);
        }
    }
    // Case 4 of 4: New Node is equal to Parent
    else {
        printf("%d already exists\n\n", (*newNode).data);
    }
}
```

I wasn't good at recursions back then and haven’t used it in a very long time. So I showed my function to Google’s AI Mode to give me feedback on it. And it told me that the return type of the function must be node instead of void to handle the recursion correctly. And I don’t need to check if the left/right nodes are empty or not since recursion does that at the same time too. And there was 1 case missing which is if the root was NULL. so after these advises, I corrected the function, and here’s the full implementation now:
```c
struct Node *InsertNode(struct Node *parentNode, struct Node *newNode) {
    // Case 1: New Node is NULL
    if (newNode == NULL) {
        printf("Cannot Insert an Empty Node\n\n");
        return parentNode;
    }

    // Case 2: Parent is NULL
    if (parentNode == NULL) {
        return newNode; 
    }

    // Case 3: New Node is Smaller than the Parent
    if ((*newNode).data < (*parentNode).data) {
        (*parentNode).leftNode = InsertNode((*parentNode).leftNode, newNode);
    }
    // Case 4: New Node is Bigger than the Parent 
    else if ((*newNode).data > (*parentNode).data) {
        (*parentNode).rightNode = InsertNode((*parentNode).rightNode, newNode);
    }
    // Case 4: New Node is equal to Parent
    else {
        printf("%d already exists\n\n", (*newNode).data);
    }

    return parentNode;
}
```

After analyzing the code, now I understand how the recursion handles checking if the left/right sides are null or not. It’s because of Case 2 which is also called “the base case”. We are doing the recursion anyways whether the children are empty or not. And once we reach the empty ones, it will return the new node assigning it to the left/right node automatically.

Okay next is removing a node. I think this function will also be recursive but more complex than just inserting! The complexity comes to what if the node I want to remove has children? What do I do with the children nodes? After researching, I found that removing an element from a BST has 3 cases:
- If the node is a leaf node (leaf means it has no children at all), then I just remove it
- If the node has 1 child, here the child will be linked to the parent of the removed parent.
- If the node has multiple children, I found multiple solutions for it! The one I’m going for is an optimized solution which is to basically search for the min or max value on either the left side or the right side of the node that will be deleted. It doesn’t matter if we want the biggest or the smallest since it will always be correct! So I’m going for the minimum on the right side of the node.
To find the min node, I made a helper function that returns that node once it finds it. Function and Helper code:
```c
struct Node *RemoveNode(struct Node *parentNode, int data) {
    // Case 1: Data is not in the tree
    if (parentNode == NULL) {
        printf("%d was not found in the tree!\n\n", data);

        return NULL;
    }

    // Case 2: Data is smaller than the parent's data
    if (data < (*parentNode).data) {
        (*parentNode).leftNode = RemoveNode((*parentNode).leftNode, data);
    }
    // Case 3: Data is larger than the parent's data
    else if (data > (*parentNode).data) {
        (*parentNode).rightNode = RemoveNode((*parentNode).rightNode, data);
    }
    // Case 4: Found the node
    else {
        // Sub-Case 1: No Children
        if ((*parentNode).leftNode == NULL && (*parentNode).rightNode == NULL) {
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);

            return NULL;
        }
        // Sub-Case 2: Left Child is not Null
        else if ((*parentNode).leftNode != NULL && (*parentNode).rightNode == NULL) {
            struct Node *tempNode = (*parentNode).leftNode;
            
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);
            
            return tempNode;
        }
        // Sub-Case 3: Right Child is not NULL
        else if ((*parentNode).leftNode == NULL && (*parentNode).rightNode != NULL) {
            struct Node *tempNode = (*parentNode).rightNode;
            
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);
            
            return tempNode;
        }
        // Sub-Case 4: Both Children are not Null
        else if ((*parentNode).leftNode != NULL && (*parentNode).rightNode != NULL) {
            struct Node *tempNode = FindMinValueNode((*parentNode).rightNode);

            (*parentNode).data = (*tempNode).data;

            (*parentNode).rightNode = RemoveNode((*parentNode).rightNode, (*tempNode).data);
        }
    }

    return parentNode;
}
```

```c
struct Node *FindMinValueNode(struct Node *node) {
    struct Node *current = node;

    // Loop down to find the leftmost leaf
    while (current != NULL && (*current).leftNode != NULL) {
        current = (*current).leftNode;
    }

    return current;
}
```

Next I’ll make a search function. I think it will be very similar to the Remove function, just when we find the node, I’ll just return it instead of doing all the removal stuff. Code:
```c
struct Node *SearchNode(struct Node *parentNode, int data) {
    if (parentNode == NULL) {
        printf("%d was not found in the tree!\n\n", data);

        return NULL;
    }

    if (data < (*parentNode).data) {
        return SearchNode((*parentNode).leftNode, data);
    }
    else if (data > (*parentNode).data) {
        return SearchNode((*parentNode).rightNode, data);
    }
    else {
        printf("%d was found successfull!\n\n", data);
        
        return parentNode;
    }
}
```

Then, I made a Display function to print the level of the tree and every node in it:
```c
void DisplayTree(struct Node *parentNode, int currentTreeLevel) {
    if (parentNode == NULL) {
        return;
    }

    // spacing between the nodes depending on the level for more clarity
    for (int i = 0; i < currentTreeLevel; i++) {
        printf("    "); 
    }
    
    printf("Level: %d\nNode Data: %d\nLeft Node Pointer: %p\nRight Node Pointer: %p\n\n", currentTreeLevel, (*parentNode).data, (*parentNode).leftNode, (*parentNode).rightNode);

    DisplayTree((*parentNode).leftNode, currentTreeLevel + 1);
    DisplayTree((*parentNode).rightNode, currentTreeLevel + 1);
}
```

Next I made a Free function to free the tree once done with it. Code:
```c
void FreeTree(struct Node *parentNode) {
    if (parentNode == NULL) {
        return;
    }

    FreeTree((*parentNode).leftNode);
    FreeTree((*parentNode).rightNode);
    free(parentNode);
}
```

Finally, I implemented the main function to test the program fully:
```c
int main(int argc, char **argv) {
    struct Node *root = NULL;

    printf("Making a 4-level Tree:\n");
    // Level 1: Root Node
    root = InsertNode(root, InitializeNode(50));
    
    // Level 2 Nodes
    root = InsertNode(root, InitializeNode(30));
    root = InsertNode(root, InitializeNode(70));
    
    // Level 3 Nodes
    root = InsertNode(root, InitializeNode(20));
    root = InsertNode(root, InitializeNode(40));
    root = InsertNode(root, InitializeNode(60));
    root = InsertNode(root, InitializeNode(80));
    
    // Level 4 Nodes
    root = InsertNode(root, InitializeNode(35));
    root = InsertNode(root, InitializeNode(45));
    root = InsertNode(root, InitializeNode(55));

    DisplayTree(root, 1);
    printf("\n\n");

    printf("Testing Search:\n");
    SearchNode(root, 45); 
    SearchNode(root, 99); 

    printf("Testing Deletion for all Cases:\n");
    printf("Case 1: No Children:\n");
    root = RemoveNode(root, 35);

    printf("\nTree Structure after removing 35:\n");
    DisplayTree(root, 1);

    printf("Case 2: One Child\n");
    root = RemoveNode(root, 40);
    printf("\nTree Structure after removing 40:\n");
    DisplayTree(root, 1);

    printf("Case 3: Two Children\n");
    root = RemoveNode(root, 70);
    printf("\nTree Structure after removing 70:\n");
    DisplayTree(root, 1);

    printf("\n\n");

    FreeTree(root);
    root = NULL;

    return 0;
}
```

**Reflection:**  
This project was really great to make! It made me revisit some old concepts I hadn't touched in a long time like BSTs and recursions. I’m still not that confident with recursions yet so I’ll definitely practice more when I have the time!

## Full Code:
```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node *leftNode;
    struct Node *rightNode;
};

struct Node *InitializeNode(int data);
struct Node *InsertNode(struct Node *parentNode, struct Node *newNode);
struct Node *RemoveNode(struct Node *parentNode, int data);
struct Node *FindMinValueNode(struct Node *node);
struct Node *SearchNode(struct Node *parentNode, int data);
void DisplayTree(struct Node *parentNode, int currentTreeLevel);
void FreeTree(struct Node *parentNode);

int main(int argc, char **argv) {
    struct Node *root = NULL;

    printf("Making a 4-level Tree:\n");
    // Level 1: Root Node
    root = InsertNode(root, InitializeNode(50));
    
    // Level 2 Nodes
    root = InsertNode(root, InitializeNode(30));
    root = InsertNode(root, InitializeNode(70));
    
    // Level 3 Nodes
    root = InsertNode(root, InitializeNode(20));
    root = InsertNode(root, InitializeNode(40));
    root = InsertNode(root, InitializeNode(60));
    root = InsertNode(root, InitializeNode(80));
    
    // Level 4 Nodes
    root = InsertNode(root, InitializeNode(35));
    root = InsertNode(root, InitializeNode(45));
    root = InsertNode(root, InitializeNode(55));

    DisplayTree(root, 1);
    printf("\n\n");

    printf("Testing Search:\n");
    SearchNode(root, 45); 
    SearchNode(root, 99); 

    printf("Testing Deletion for all Cases:\n");
    printf("Case 1: No Children:\n");
    root = RemoveNode(root, 35);

    printf("\nTree Structure after removing 35:\n");
    DisplayTree(root, 1);

    printf("Case 2: One Child\n");
    root = RemoveNode(root, 40);
    printf("\nTree Structure after removing 40:\n");
    DisplayTree(root, 1);

    printf("Case 3: Two Children\n");
    root = RemoveNode(root, 70);
    printf("\nTree Structure after removing 70:\n");
    DisplayTree(root, 1);

    printf("\n\n");

    FreeTree(root);
    root = NULL;

    return 0;
}

struct Node *InitializeNode(int data) {
    struct Node *newNode = malloc(sizeof(struct Node));

    if (newNode == NULL) {
        printf("Failed to Allocate Memory for the Node\n\n");
        
        return NULL;
    }

    (*newNode).data = data;
    (*newNode).leftNode = NULL;
    (*newNode).rightNode = NULL;

    return newNode;
}

struct Node *InsertNode(struct Node *parentNode, struct Node *newNode) {
    // Case 1: New Node is NULL
    if (newNode == NULL) {
        printf("Cannot Insert an Empty Node\n\n");
        
        return parentNode;
    }

    // Case 2: Parent is NULL
    if (parentNode == NULL) {
        return newNode; 
    }

    // Case 3: New Node is Smaller than the Parent
    if ((*newNode).data < (*parentNode).data) {
        (*parentNode).leftNode = InsertNode((*parentNode).leftNode, newNode);
    }
    // Case 4: New Node is Bigger than the Parent 
    else if ((*newNode).data > (*parentNode).data) {
        (*parentNode).rightNode = InsertNode((*parentNode).rightNode, newNode);
    }
    // Case 4: New Node is equal to Parent
    else {
        printf("%d already exists\n\n", (*newNode).data);
    }

    return parentNode;
}

struct Node *RemoveNode(struct Node *parentNode, int data) {
    // Case 1: Data is not in the tree
    if (parentNode == NULL) {
        printf("%d was not found in the tree!\n\n", data);

        return NULL;
    }

    // Case 2: Data is smaller than the parent's data
    if (data < (*parentNode).data) {
        (*parentNode).leftNode = RemoveNode((*parentNode).leftNode, data);
    }
    // Case 3: Data is larger than the parent's data
    else if (data > (*parentNode).data) {
        (*parentNode).rightNode = RemoveNode((*parentNode).rightNode, data);
    }
    // Case 4: Found the node
    else {
        // Sub-Case 1: No Children
        if ((*parentNode).leftNode == NULL && (*parentNode).rightNode == NULL) {
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);

            return NULL;
        }
        // Sub-Case 2: Left Child is not Null
        else if ((*parentNode).leftNode != NULL && (*parentNode).rightNode == NULL) {
            struct Node *tempNode = (*parentNode).leftNode;
            
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);
            
            return tempNode;
        }
        // Sub-Case 3: Right Child is not NULL
        else if ((*parentNode).leftNode == NULL && (*parentNode).rightNode != NULL) {
            struct Node *tempNode = (*parentNode).rightNode;
            
            free(parentNode);

            printf("%d has been deleted successfully!\n\n", data);
            
            return tempNode;
        }
        // Sub-Case 4: Both Children are not Null
        else if ((*parentNode).leftNode != NULL && (*parentNode).rightNode != NULL) {
            struct Node *tempNode = FindMinValueNode((*parentNode).rightNode);

            (*parentNode).data = (*tempNode).data;

            (*parentNode).rightNode = RemoveNode((*parentNode).rightNode, (*tempNode).data);
        }
    }

    return parentNode;
}

struct Node *FindMinValueNode(struct Node *node) {
    struct Node *current = node;

    // Loop down to find the leftmost leaf
    while (current != NULL && (*current).leftNode != NULL) {
        current = (*current).leftNode;
    }

    return current;
}

struct Node *SearchNode(struct Node *parentNode, int data) {
    if (parentNode == NULL) {
        printf("%d was not found in the tree!\n\n", data);

        return NULL;
    }

    if (data < (*parentNode).data) {
        return SearchNode((*parentNode).leftNode, data);
    }
    else if (data > (*parentNode).data) {
        return SearchNode((*parentNode).rightNode, data);
    }
    else {
        printf("%d was found successfull!\n\n", data);

        return parentNode;
    }
}

void DisplayTree(struct Node *parentNode, int currentTreeLevel) {
    if (parentNode == NULL) {
        return;
    }

    // spacing between the nodes depending on the level for more clarity
    for (int i = 0; i < currentTreeLevel; i++) {
        printf("    "); 
    }
    
    printf("Level: %d\nNode Data: %d\nLeft Node Pointer: %p\nRight Node Pointer: %p\n\n", currentTreeLevel, (*parentNode).data, (*parentNode).leftNode, (*parentNode).rightNode);

    DisplayTree((*parentNode).leftNode, currentTreeLevel + 1);
    DisplayTree((*parentNode).rightNode, currentTreeLevel + 1);
}

void FreeTree(struct Node *parentNode) {
    if (parentNode == NULL) {
        return;
    }

    FreeTree((*parentNode).leftNode);
    FreeTree((*parentNode).rightNode);
    free(parentNode);
}
```