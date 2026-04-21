# Phase 1 - Day 14: Tuesday 21 April 2026
## Summary:
Made the second project of Category 1: Hex Dump Utility

## Note:
Documentation first, and Full code with the text file used at the bottom.

## Documentation:
For this project, I’ll make a terminal app that accepts a txt file path in the command line arguments, reads the file, and prints its content in both Hex and ASCII.

First, I made the command line argument validation, and file validation.
Main function code:
```c
if (argc < 2) {
    printf("Please provide the path of the txt file\n\n");

    return 1;
}

if (argc > 2) {
    printf("Invalid\n");
    printf("Usage Example: .\\main.exe File_Path\n\n");

    return 1;
}

char *filePath = argv[1];

int fileExist = CheckIfFileExist(filePath);

if (fileExist == 1) {
    printf("Failed to open the file!\n\n");

    return 1;
}

int fileFormatCorrect = CheckFileFormat(filePath);

if (fileFormatCorrect == 1) {
    printf("File must only be a txt file!\n\n");

    return 1;
}
```

Functions:
```c
int CheckIfFileExist(char *filePath) {
    FILE *filePointer = fopen(filePath, "r");

    if (filePointer == NULL) {
        return 1;
    }

    fclose(filePointer);

    return 0;
}

int CheckFileFormat(char *filePath) {
    char *fileExtension = strrchr(filePath, '.');

    if (strcmp(fileExtension, ".txt") != 0) {
        return 1;
    }

    return 0;
}
```

Next, the main logic, I need to read the text from the text file, transform it into Hex and ASCII then print the results. Since this is a textfile, I think it’s better to do things character by character.

I defined an int to represent each character, and looped through the file 3 times. The first time, I printed the actual letter! The second time I printed the HEX version of it. And the third time I printed the ASCII version of it.
```c
FILE *filePointer = fopen(filePath, "r");

int character;

// Normal Text
while ((character = fgetc(filePointer)) != EOF) {
    printf("%c", character);
}

printf("\n\n");

// Hex Text
while ((character = fgetc(filePointer)) != EOF) {
    printf("%02X", character);
}

printf("\n\n");

// ASCII Text
while ((character = fgetc(filePointer)) != EOF) {
    printf("%d", character);
}

fclose(filePointer);
```

Now this didn't work as expected! Only the first one loop worked, but the second and third didn't! I suspected that maybe because the file ended in the first loop, there is nothing to read in the second and third! the file just saves that it ended or something! After researching, I found that my speculations were almost correct! When reading a file, the "cursor" moves with it. so the character we're reading is the final character read at the end of the file. It stays at this stage. We need to manually reset to the top of the file again by using the function rewind. And to make the printing cleaner for the Hex and ASCII versions, I added a space between each character!

**Reflection:**  
This was a fun and small project to do! It made me practice more the file reading and validation I did in the C0 Project. And learned about multiple little things like EOF being the End of File, 02X means HEX, and that I need to rewind the file at each reading.

## Full Code:
```c
#include <stdio.h>
#include <string.h>

int CheckIfFileExist(char *filePath);
int CheckFileFormat(char *filePath);

int main(int argc, char **argv) {
    if (argc < 2) {
        printf("Please provide the path of the txt file\n\n");

        return 1;
    }
    
    if (argc > 2) {
        printf("Invalid\n");
        printf("Usage Example: .\\main.exe File_Path\n\n");

        return 1;
    }

    char *filePath = argv[1];

    int fileExist = CheckIfFileExist(filePath);
    
    if (fileExist == 1) {
        printf("Failed to open the file!\n\n");

        return 1;
    }

    int fileFormatCorrect = CheckFileFormat(filePath);

    if (fileFormatCorrect == 1) {
        printf("File must only be a txt file!\n\n");

        return 1;
    }

    FILE *filePointer = fopen(filePath, "r");

    int character;

    // Normal Text
    while ((character = fgetc(filePointer)) != EOF) {
        printf("%c", character);
    }

    printf("\n\n");

    // Hex Text
    rewind(filePointer);
    while ((character = fgetc(filePointer)) != EOF) {
        printf("%02X ", character);
    }

    printf("\n\n");

    // ASCII Text
    rewind(filePointer);
    while ((character = fgetc(filePointer)) != EOF) {
        printf("%d ", character);
    }

    fclose(filePointer);
    
    return 0;
}

int CheckIfFileExist(char *filePath) {
    FILE *filePointer = fopen(filePath, "r");

    if (filePointer == NULL) {
        return 1;
    }

    fclose(filePointer);

    return 0;
}

int CheckFileFormat(char *filePath) {
    char *fileExtension = strrchr(filePath, '.');

    if (strcmp(fileExtension, ".txt") != 0) {
        return 1;
    }

    return 0;
}
```

## Text File Used:
Hello World! This is Project 2 of Category 1 named Hex Dump Utility