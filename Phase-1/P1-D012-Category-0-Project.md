# Phase 1 - Day 12: Friday 17 April 2026
## Summary:
Made the project mentioned yesterday about a PhoneBook.

## Note:
Documentation first, Full code and the used txt file at the bottom.

## Documentation:
Today I’m going to plan and implement the suggested project I talked about yesterday!

I’m thinking of a terminal program about a Phonebook. The phonebook will be saved in a text file. When running the program, the user needs to enter the name of the phonebook file and optionally a functionality to do. If a functionality was provided in the terminal, the program will do this functionality and exit automatically. If no functionality was provided, the program will run as a full program.

Program’s Functionalities:
- Add Contact (add in the terminal)
- Search Contact (search in the terminal)
- Display all Contacts (display in the terminal)
- Delete Contact (delete in the terminal)
- Exit

When the program opens the text file, it will translate each contact to a Contact struct that has name and phone number as variables. Then every contact will be placed in a Contact list.

All work will be on the Contact array, once done, it will be saved in the text file.

This project covers every point studied yesterday:
- **Pointers:** Strings, arrays, …
- **Arrays:** The Contact array
- **Heap:** The Contact array will be made dynamically since we can’t know how many contacts are there before reading the file.
- **Strings:** Each contact will have a name and a phone number which will be saved as strings.
- **Structs:** The Contact struct.
- **Reading Files:** The Phonebook text file.
- **Command Line Arguments:** The user will need to enter the text file and an optional functionality.

With that I’m starting the implementation!

First thing I’ll be doing is to handle the Command Line Arguments and reading the file. I was thinking if I should allow any name for the text file or make it specific like Phonebook.txt. Making it specific allows the job to get done fast, but I think more professionally, making it generic is better, so I’ll go with that approach.

I wrote these to get the correct number of command line arguments:
```c
if (argc < 2) {
    printf("Error: please provide the full path phonebook text file, and optionally, one functionality\n");
    printf("Usage example: .\\main.c \"C:\\Users\\User 1\\Desktop\\Phonebook.txt\" add\n\n");
    printf("Functionalities available:\n1) add: Add a Contact\n2) delete: Delete a Contact\n3) display: Display all Contacts\n4) search: Search a Contact\n");
    
    return 1;
}

if (argc > 3) {
    printf("Error: please provide the full path phonebook text file, and optionally, one functionality\n");
    printf("Usage example: .\\main.c \"C:\\Users\\User 1\\Desktop\\Phonebook.txt\" add\n\n");
    printf("Functionalities available:\n1) add: Add a Contact\n2) delete: Delete a Contact\n3) display: Display all Contacts\n4) search: Search a Contact\n");
    
    return 1;
}
```

After that, I got the path to the file from the command arguments:
```c
char *phonebookFilePath = argv[1];
```

Then handled the file opening and closing:
```c
FILE *phonebookFilePtr = fopen(phonebookFilePath, "r");

if (phonebookFilePtr == NULL) {
    printf("Error: Couldn't open the file\n");

    return 1;
}

fclose(phonebookFilePtr);
```

What I want specifically from the file is to display an error if it doesn’t exist, I don’t want the program to automatically create a new one since maybe the user just entered a typo or something, no need to create a new one for that mistake. And if the file does exist, I want to have read/write access. And finally, when saving, I want to overwrite the full file with the new content.

So I was researching on https://devdocs.io/c/ to see the privileges I can set to fopen based on my needs. Unfortunately there isn’t a privilege that meets my specifications, but fortunately there is a solution to this.

The solution is to first open the file using read privilege (r), If the file doesn’t exist I throw an error to the user (same approach I already implemented). But if the file does exist, I close the file that has been opened with r, and reopen it with w+ privilege since w+ has read/write privileges and overwrite the file.

So I transformed the existing file opening logic to a function to simplify the process:
```c
int CheckIfTextFileExist(char *filePath) {
    FILE *phonebookFilePtr = fopen(filePath, "r");

    if (phonebookFilePtr == NULL) {
        printf("Error: Couldn't open the file\n");

        return 1;
    }

    fclose(phonebookFilePtr);

    return 0;
}
```

in main:
```c
int doesFileExist = CheckIfTextFileExist(phonebookFilePath);

if (doesFileExist == 1) {
    printf("Error: Couldn't open the file\n");

    return 1;
}
```

So since now I made sure that the program does exist, now I want to check the extension of it since fopen accepts any file extension (as far as I know). So I googled how to get a file extension from a string in C, and I found that there is a function from the string header file called strrchr. After reading about it in https://devdocs.io/c/ I learned that this function searches for the last occurrence of a character in a string, and copies everything from that character to the end of the string onto another string and returns it.

This is exactly what I wanted! So I made another function to check for the file extension:
```c
int CheckFileExtension(char *filePath) {
    char *fileExtension = strrchr(filePath, '.');

    if (strcmp(fileExtension, ".txt") != 0) {
        return 1;
    }

    return 0;
}
```

in main:
```c
int isFileExtensionTxt = CheckFileExtension(phonebookFilePath);

if (isFileExtensionTxt == 1) {
    printf("Error: The file must be a text file (.txt)\n");

    return 1;
}
```

Okay now since we have the correct file, I think now is it’s time to complete the main part of the program, which is handling contacts. I started by creating the Contact struct, which contains 2 fields, a name, and a phone number. For the phone number, I made it fixed length since phone numbers have max length. So no need to add the complexity of dynamic memory.
```c
struct Contact {
    char *name;
    char phoneNumber[20];
};
```

Now it's time to read the data from the file. Data in the file will be as following:
Name: Some Name | Phone Number: Some Phone Number

I was researching how to read data from a file written in a specific format, and found that sscanf is used for that. So I read about it in https://devdocs.io/c/.

Reading from a file is done by a buffer, so I need the buffer version of sscanf. Sscanf also takes a format string that excludes everything written inside it unless we specify a %. It starts extracting from the %. It has the usual C formats, %d for integers, %c for characters, … Our data are strings. But the extractor in sscanf for strings stops at the first space, and since our data have spaces, we can’t use it. So another operator can be used which is %[]. This operator tells the extractor to extract until you hit something. For example, %[^|]. This tell the extractor to extract everything until it sees a |. This is exactly what we need.

So the format needs to be like this:”Name: %[^|]| Phone Number: %[^\n]”. The problem is that it will catch the space after the name. So I’ll need to remove it. Manually since there is no trimming function in C (as far as I know).

The problem with sscanf is that it needs to work with fixed data. So I need to create a fixed size name field to be the name extractor. This is not a real problem since it is reused for each contact. So it doesn’t introduce the original “a lot of wasted space” problem. But now, to prevent someone from entering a very long name, I need to make a name size limit rule.

So I made this extractor logic and tested it with a simple print:
```c
char name[100];
char phone[20];
char buffer[200];
while (fgets(buffer, sizeof(buffer), phonebookFilePtr) != NULL) {
    if (sscanf(buffer, "Name: %[^|]| Phone Number: %[^\n]", name, phone) == 2)
    {
        printf("Size of Name: %d, Name: %s | Phone Number: %s\n", strlen(name), name, phone);
    }
}
```

So after writing the extraction logic, I prepared the contacts list:
```c
int capacity = 10;
int count = 0;
struct Contact *contacts = malloc(capacity * sizeof(struct Contact));
```

After writing this extraction logic, I decided to make the trimming logic to trim the space at the end of the extracted name. What I thought of doing is to make a function that takes the extracted name, calculates the length of it, and makes another string with a size of the extracted name - 1. Like this:
```c
char *ExtractedNameTrimmer(char *originalExtractedName) {
    int originalExtractedNameLength = strlen(originalExtractedName);

    char trimmedName[originalExtractedNameLength - 1];
}
```

But I ran into a problem with that approach. After reading the error, it turned out that this can’t be done since the length of the name will be known only at runtime! So the solution to this is to use dynamic memory. And since the function returns this name, I need to free this name from the caller (which is main so far).

So I implemented this function:
```c
char *ExtractedNameTrimmer(char *originalExtractedName) {
    int originalExtractedNameLength = strlen(originalExtractedName);

    char *trimmedName = malloc((originalExtractedNameLength - 1) * sizeof(char));

    // -2 since I need to fill the trimmedName which has a size of -1 the original name.
    // The other -1 since we are starting the loop from 0 instead of 1.
    int loopLength = originalExtractedNameLength - 2;
    for (int i = 0; i < loopLength; i++) {
        trimmedName[i] = originalExtractedName[i];
    }

    trimmedName[strlen(trimmedName) - 1] = '\0';
    
    return trimmedName;
}
```

And this is the extraction logic I used for testing it:
```c
char name[100];
char phone[20];
char buffer[200];
while (fgets(buffer, sizeof(buffer), phonebookFilePtr) != NULL) {
    if (sscanf(buffer, "Name: %[^|]| Phone Number: %[^\n]", name, phone) == 2)
    {
        printf("Size of Name: %d, Name: %s\n", strlen(name), name);

        char *trimmedName = ExtractedNameTrimmer(name);
        printf("Size of the Trimmed Name: %d, Trimmed Name: %s\n\n", strlen(trimmedName), trimmedName);

        free(trimmedName);
    }
}
```

But it didn’t run well! Sometimes the new name is shorter than needed, sometimes it has garbage letters. So I went to fix the function.

First, I realized that I don’t need to ask for -1 size from memory since strlen doesn’t include the null terminator. So I need -1 size for characters and +1 size for the null terminator which basically mean the same size.

Second, malloc might fail, I didn’t handle that in the original function.

Third, for the loop, I need to do -1 not -2 size, since -1 is needed for the characters.

After the fixes, the function run perfectly, and this is the new implementation:
```c
char *ExtractedNameTrimmer(char *originalExtractedName) {
    int originalExtractedNameLength = strlen(originalExtractedName);

    char *trimmedName = malloc(originalExtractedNameLength * sizeof(char));

    if (trimmedName == NULL) {
        return NULL;
    }

    int newLength = originalExtractedNameLength - 1;
    for (int i = 0; i < newLength; i++) {
        trimmedName[i] = originalExtractedName[i];
    }

    trimmedName[newLength] = '\0';
    
    return trimmedName;
}
```

Now, after the trimming logic is complete, I prepared the contacts list:
```c
int capacity = 10;
int count = 0;
struct Contact *contacts = malloc(capacity * sizeof(struct Contact));
```

The capacity will help in defining the size of the array. I thought of making it 10 at a time. So 10, then when we’ll need the 11th contact, the size of the array will become 20, and so on. And the count will help in knowing how many contacts there are.

After that, I completed the extraction logic:
```c
char name[100];
char phone[20];
char buffer[200];
while (fgets(buffer, sizeof(buffer), phonebookFilePtr) != NULL) {
    if (sscanf(buffer, "Name: %[^|]| Phone Number: %[^\n]", name, phone) == 2)
    {
        if (count == capacity) {
            capacity = capacity + 10;
            contacts = realloc(contacts, capacity * sizeof(struct Contact));
        }

        char *trimmedName = ExtractedNameTrimmer(name);
        
        int trimmedNameLength = strlen(trimmedName);
        contacts[count].name = malloc(trimmedNameLength * sizeof(char));

        strcpy(contacts[count].name, trimmedName);
        strcpy(contacts[count].phoneNumber, phone);
        
        free(trimmedName);

        count++;
    }
}
```

And at the end I freed all the names, then the contacts list:
```c
for (int i = 0; i < count; i++) {
    free(contacts[i].name);
}

free(contacts);
```

Now this line:
```c
strcpy(contacts[count].name, trimmedName);
```

Caused a warning, that strcpy is writing one extra byte more than the allocated to the malloc. After digging into it, I realized that this warning is happening because I gave the name variable just enough space for the name without the null terminator. Since strlen doesn’t count the null terminator. So this is an easy fix:
```c
contacts[count].name = malloc((trimmedNameLength  + 1) * sizeof(char));
```

One note to add, when I was testing reading data from the file, I noticed that w+ privilege deletes the content of the file once open, which is not what I need. I switched to r mode for now.

With that, the contact list is done! Now, it's time to make the program’s functionalities. First, I made the main program loop:
```c
int programIsRunning = 1;
while (programIsRunning == 1) {
    char userOption;
    printf("Welcome!\n\n");
    printf("What do you want to do?\n");
    printf("1. Add a Contact\n2. Delete a Contact\n3. Display all Contacts\n4. Search a Contact\n5. Exit\n\n");
    printf("Please write the number of the option you want: ");

    scanf(" %c", &userOption);
    if (userOption < '1' || userOption > '5') {
        printf("Invalid Input\n\n");

        continue;
    }

    if (userOption == '1') {
        // Add a Contact Logic
    }
    else if (userOption == '2') {
        // Delete a Contact Logic
    }
    else if (userOption == '3') {
        // Display all Contacts Logic
    }
    else if (userOption == '4') {
        // Search a Contact Logic
    }
    else if (userOption == '5') {
        programIsRunning = 0;
    }
}
```

And now, it’s time to implement each one of them! Starting with Adding a Contact, I’m thinking about making a function that takes the Contact array, its count, and its capacity as parameters. First it needs to check for the count, if the array is full, we need to expand it. Then, it needs to ask the user for a name (max size 100 characters), and a phone number, then it creates a new Contact object based on the value given by the user and inserts it in the array. Finally, it needs to return this new array pointer.

This is the function:
```c
struct Contact *AddContact(struct Contact *contactList, int *count, int *capacity) {
    if (*count == *capacity) {
        *capacity = *capacity + 10;

        struct Contact *temp = realloc(contactList, *capacity * sizeof(struct Contact));

        if (temp == NULL) {
            printf("Failed to expand the contact list\n");

            return contactList;
        }

        contactList = temp;
    }

    char name[100];
    printf("Name (Name can be a maximum of 100 characters): ");
    scanf(" %[^\n]", &name);

    char phoneNumber[20];
    printf("Phone Number: ");
    scanf(" %[^\n]", &phoneNumber);

    contactList[*count].name = malloc((strlen(name) + 1) * sizeof(char));

    strcpy(contactList[*count].name, name);
    strcpy(contactList[*count].phoneNumber, phoneNumber);

    *count++;

    printf("Contact has been added successfully!\n\n");

    return contactList;
}
```

While making the function I realized that I missed an important thing in the extraction logic, which is that reallocation can fail, and failing means losing the pointer to the original memory block! I handled it in this function but not in the extraction logic, so I fixed it. And this is now the expansion logic in the extraction logic:
```c
if (count == capacity) {
    capacity = capacity + 10;

    struct Contact *temp = realloc(contacts, capacity * sizeof(struct Contact));

    if (temp == NULL) {
        printf("Failed to expand the contact list\n");

        return 1;
    }

    contacts = temp;
}
```

Now to test the adding logic, I skipped the Deleting part and made the Display Contacts logic. It’s very straight forward:
```c
void DisplayContacts(struct Contact *contactList, int *count, int *capacity) {
    printf("Contact List Capacity: %d\n", *capacity);
    printf("Contacts Count: %d\n\n", *count);

    for (int i = 0; i < *count; i++) {
        printf("Contact %d:\n", i + 1);
        printf("Name: %s\n", contactList[i].name);
        printf("Phone Number: %s\n\n", contactList[i].phoneNumber);
    }

    printf("\n\n");
}
```

And on testing I found a bug! The Adding function is displaying “Contact has been added successfully!” But when displaying all contacts, only the original ones got displayed! Is there a problem with the count number? I’m increasing it! I didn’t find an answer so I gave the add and display functions to Google’s AI Mode and asked about it. And it told me that the bug is with *count++. It told me that this increments the address not the value, and the way to increment the value is by doing this (*count)++. I tried it and it worked! And now both my add and display functions have been tested!

Now the deletion function. The function will take the contact list, and the count. The function then needs to ask the user for the name of the contact. If it finds it, it deletes, if not then it displays the contact doesn’t exist. On deletion, the count will be decreased, and the updated list will be returned to the user. But now there are 2 things to consider:
- How do I remove the space from the list? There is no built-in delete function in C (as far as I know), so I need to do that manually.
- Should I decrease the capacity of the list? The list has an original size of 10, and if the list gets full, I add 10 extra. Let’s say the capacity is 30 and I have 21 contacts. If I delete a contact and go back to 20, should I decrease the capacity of the list to 20 again? I think it’s more professional to do that so I’ll do it.

Now regarding deleting the value from the list entirely, after researching, I found that there is actually a function that can do that. It’s called memmove. It shifts values. So if I have 20 items for example, and I want to delete the 14th item, I can shift all the values from 15 to 20 by 1 to the left.

So to be able to shift the contacts, I calculated how many contacts needed to be shifted. And this is the formula: Total Count of Contacts - The Index of the Contact to Delete - 1. The final -1 is because indexing starts from 0. Let’s say I have 8 items. So the count is 8. I want to remove the item at index 3 (the 4th element). So there are 4 elements that need to be shifted. 8 - 3 - 1 = 4. If the final element is the one getting deleted, no shifts need to happen.

After the function, I got it working perfectly, here’s the full implementation:
```c
struct Contact *DeleteContact(struct Contact *contactList, int *count, int *capacity) {
    char name[100];
    printf("Enter the name of the contact you want to delete: ");
    scanf(" %[^\n]", &name);

    int contactIsFound = 0;
    int contactIndex = 0;
    for (int i = 0; i < *count; i++) {
        if (strcmp(contactList[i].name, name) == 0) {
            contactIsFound = 1;
            
            break;
        }

        contactIndex++;
    }

    if (contactIsFound == 0) {
        printf("Contact doesn't exist\n\n");

        return contactList;
    }
    
    int numberOfContactsToShift = *count - contactIndex - 1;
    if (numberOfContactsToShift > 0) {
        memmove(&contactList[contactIndex], &contactList[contactIndex + 1], numberOfContactsToShift * sizeof(struct Contact));
    }

    (*count)--;

    if (*count + 10 == *capacity) {
        *capacity = *capacity - 10;
    
        struct Contact *temp = realloc(contactList, *capacity * sizeof(struct Contact));

        if (temp == NULL) {
            printf("Failed to shrink the contact list\n");
        }

        contactList = temp;
    }

    printf("Contact has been deleted successfully!\n\n");

    return contactList;
}
```

Now, only searching is missing, which, but I think it’s pretty forward, I already have half of its logic in the deletion function. Here’s the full function:
```c
void SearchContact(struct Contact *contactList, int *count) {
    char name[100];
    printf("Enter the name of the contact: ");
    scanf(" %[^\n]", &name);

    for (int i = 0; i < *count; i++) {
        if (strcmp(contactList[i].name, name) == 0) {
            printf("Contact Found:\n");
            printf("Contact Name: %s\n", contactList[i].name);
            printf("Contact Phone Number: %s\n\n", contactList[i].phoneNumber);

            return;
        }
    }
    
    printf("Contact doesn't exist\n\n");
}
```

Okay, now there are 2 things left for the program, saving and quick actions if the user entered a functionality in the terminal. Starting with saving.

Based on fopen documentation on https://devdocs.io/c/. w privilege destroys the content of the file once opened, this allows me to write the new data again. So I’ll use that. And for writing, I searched for the function used to write data into a file in a special format, and I got fprintf. So I’ll use that too.

I’ll make the saving logic in a function that accepts the contact list, the count, and the file path as parameters. The function was really easy to implement:
```c
void SaveData(struct Contact *contactList, int *count, char *filePath) {
    FILE *fileToWriteTo = fopen(filePath, "w");

    for (int i = 0; i < *count; i++) {
        fprintf(fileToWriteTo, "Name: %s | Phone Number: %s\n", contactList[i].name, contactList[i].phoneNumber);
    }

    fclose(fileToWriteTo);

    printf("Data has been saved successfully!");
}
```

Now for the final thing, handling the quick actions. The way I’m thinking of doing it is to change the program loop. Instead of always triggering, I’ll put it behind an if statement that will check if the arguments given were exactly 2 (name of the program, and name of the file path), if so, it will trigger. If the arguments were 3 (name of the program, name of the file path, and an action), another set of activities will trigger.

This is the final version of the program loop and quick actions:
```c
if (argc == 2) {
    // Program Loop:
    int programIsRunning = 1;
    while (programIsRunning == 1) {
        char userOption;
        printf("Welcome!\n\n");
        printf("What do you want to do?\n");
        printf("1. Add a Contact\n2. Delete a Contact\n3. Display all Contacts\n4. Search a Contact\n5. Exit\n\n");
        printf("Please write the number of the option you want: ");

        scanf(" %c", &userOption);
        if (userOption < '1' || userOption > '5') {
            printf("Invalid Input\n\n");

            continue;
        }

        if (userOption == '1') {
            contacts = AddContact(contacts, &count, &capacity);
        }
        else if (userOption == '2') {
            contacts = DeleteContact(contacts, &count, &capacity);
        }
        else if (userOption == '3') {
            DisplayContacts(contacts, &count, &capacity);
        }
        else if (userOption == '4') {
            SearchContact(contacts, &count);
        }
        else if (userOption == '5') {
            SaveData(contacts, &count, phonebookFilePath);
            programIsRunning = 0;
        }
    }
}
else if (argc == 3) {
    if (strcmp(argv[2], "add") == 0) {
        contacts = AddContact(contacts, &count, &capacity);
        SaveData(contacts, &count, phonebookFilePath);
    }
    else if (strcmp(argv[2], "delete") == 0) {
        contacts = DeleteContact(contacts, &count, &capacity);
        SaveData(contacts, &count, phonebookFilePath);
    }
    else if (strcmp(argv[2], "display") == 0) {
        DisplayContacts(contacts, &count, &capacity);
    }
    else if (strcmp(argv[2], "search") == 0) {
        SearchContact(contacts, &count);
    }
    else {
        printf("Invalid Option!\n\n");
    }
}
```

With that the project is done!

**Reflection:**  
The project was really fun to do, and I learned a lot of new things while doing it! The only thing I’m not happy about is that it took me the full day to complete! I took time to think about algorithms and how I should do them, and researching new functions, … But I hope that with more practice I’ll get better and faster!

## Full Code:
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

struct Contact {
    char *name;
    char phoneNumber[20];
};

int CheckIfTextFileExist(char *filePath);
int CheckFileExtension(char *filePath);
char *ExtractedNameTrimmer(char *originalExtractedName);

struct Contact *AddContact(struct Contact *contactList, int *count, int *capacity);
struct Contact *DeleteContact(struct Contact *contactList, int *count, int *capacity);
void DisplayContacts(struct Contact *contactList, int *count, int *capacity);
void SearchContact(struct Contact *contactList, int *count);
void SaveData(struct Contact *contactList, int *count, char *filePath);


int main(int argc, char **argv) {
    // Handle Command Line Arguments
    if (argc < 2) {
        printf("Error: please provide the full path phonebook text file, and optionally, one functionality\n");
        printf("Usage example: .\\main.c \"C:\\Users\\User 1\\Desktop\\Phonebook.txt\" add\n\n");
        printf("Functionalities available:\n1) add: Add a Contact\n2) delete: Delete a Contact\n3) display: Display all Contacts\n4) search: Search a Contact\n");
        
        return 1;
    }

    if (argc > 3) {
        printf("Error: please provide the full path phonebook text file, and optionally, one functionality\n");
        printf("Usage example: .\\main.c \"C:\\Users\\User 1\\Desktop\\Phonebook.txt\" add\n\n");
        printf("Functionalities available:\n1) add: Add a Contact\n2) delete: Delete a Contact\n3) display: Display all Contacts\n4) search: Search a Contact\n");
        
        return 1;
    }

    // Get the file path
    char *phonebookFilePath = argv[1];

    // Check if the file exists
    int doesFileExist = CheckIfTextFileExist(phonebookFilePath);

    // If not, throw an error
    if (doesFileExist == 1) {
        printf("Error: Couldn't open the file\n");

        return 1;
    }

    // Check File Extension
    int isFileExtensionTxt = CheckFileExtension(phonebookFilePath);

    // If not, throw an error
    if (isFileExtensionTxt == 1) {
        printf("Error: The file must be a text file (.txt)\n");

        return 1;
    }

    // Open the file in r/w mode
    FILE *phonebookFilePtr = fopen(phonebookFilePath, "r");

    // Create the contact list
    int capacity = 10;
    int count = 0;
    struct Contact *contacts = malloc(capacity * sizeof(struct Contact));

    // Read the data from the file and store them in the contact list
    char name[100];
    char phone[20];
    char buffer[200];
    while (fgets(buffer, sizeof(buffer), phonebookFilePtr) != NULL) {
        if (sscanf(buffer, "Name: %[^|]| Phone Number: %[^\n]", name, phone) == 2)
        {
            if (count == capacity) {
                capacity = capacity + 10;

                struct Contact *temp = realloc(contacts, capacity * sizeof(struct Contact));

                if (temp == NULL) {
                    printf("Failed to expand the contact list\n");

                    return 1;
                }

                contacts = temp;
            }

            char *trimmedName = ExtractedNameTrimmer(name);
            
            int trimmedNameLength = strlen(trimmedName);
            contacts[count].name = malloc((trimmedNameLength + 1) * sizeof(char));

            strcpy(contacts[count].name, trimmedName);
            strcpy(contacts[count].phoneNumber, phone);
            
            free(trimmedName);

            count++;
        }
    }

    // Close the file
    fclose(phonebookFilePtr);

    if (argc == 2) {
        // Program Loop:
        int programIsRunning = 1;
        while (programIsRunning == 1) {
            char userOption;
            printf("Welcome!\n\n");
            printf("What do you want to do?\n");
            printf("1. Add a Contact\n2. Delete a Contact\n3. Display all Contacts\n4. Search a Contact\n5. Exit\n\n");
            printf("Please write the number of the option you want: ");

            scanf(" %c", &userOption);
            if (userOption < '1' || userOption > '5') {
                printf("Invalid Input\n\n");

                continue;
            }

            if (userOption == '1') {
                contacts = AddContact(contacts, &count, &capacity);
            }
            else if (userOption == '2') {
                contacts = DeleteContact(contacts, &count, &capacity);
            }
            else if (userOption == '3') {
                DisplayContacts(contacts, &count, &capacity);
            }
            else if (userOption == '4') {
                SearchContact(contacts, &count);
            }
            else if (userOption == '5') {
                SaveData(contacts, &count, phonebookFilePath);
                programIsRunning = 0;
            }
        }
    }
    else if (argc == 3) {
        if (strcmp(argv[2], "add") == 0) {
            contacts = AddContact(contacts, &count, &capacity);
            SaveData(contacts, &count, phonebookFilePath);
        }
        else if (strcmp(argv[2], "delete") == 0) {
            contacts = DeleteContact(contacts, &count, &capacity);
            SaveData(contacts, &count, phonebookFilePath);
        }
        else if (strcmp(argv[2], "display") == 0) {
            DisplayContacts(contacts, &count, &capacity);
        }
        else if (strcmp(argv[2], "search") == 0) {
            SearchContact(contacts, &count);
        }
        else {
            printf("Invalid Option!\n\n");
        }
    }

    // Free the names and the contact list
    for (int i = 0; i < count; i++) {
        free(contacts[i].name);
    }

    free(contacts);

    return 0;
}

int CheckIfTextFileExist(char *filePath) {
    FILE *phonebookFilePtr = fopen(filePath, "r");

    if (phonebookFilePtr == NULL) {
        return 1;
    }

    fclose(phonebookFilePtr);

    return 0;
}

int CheckFileExtension(char *filePath) {
    char *fileExtension = strrchr(filePath, '.');

    if (strcmp(fileExtension, ".txt") != 0) {
        return 1;
    }

    return 0;
}

char *ExtractedNameTrimmer(char *originalExtractedName) {
    int originalExtractedNameLength = strlen(originalExtractedName);

    char *trimmedName = malloc(originalExtractedNameLength * sizeof(char));

    if (trimmedName == NULL) {
        return NULL;
    }

    int newLength = originalExtractedNameLength - 1;
    for (int i = 0; i < newLength; i++) {
        trimmedName[i] = originalExtractedName[i];
    }

    trimmedName[newLength] = '\0';
    
    return trimmedName;
}

struct Contact *AddContact(struct Contact *contactList, int *count, int *capacity) {
    if (*count == *capacity) {
        *capacity = *capacity + 10;

        struct Contact *temp = realloc(contactList, *capacity * sizeof(struct Contact));

        if (temp == NULL) {
            printf("Failed to expand the contact list\n");

            return contactList;
        }

        contactList = temp;
    }

    char name[100];
    printf("Name (Name can be a maximum of 100 characters): ");
    scanf(" %[^\n]", &name);

    char phoneNumber[20];
    printf("Phone Number: ");
    scanf(" %[^\n]", &phoneNumber);

    contactList[*count].name = malloc((strlen(name) + 1) * sizeof(char));

    strcpy(contactList[*count].name, name);
    strcpy(contactList[*count].phoneNumber, phoneNumber);

    (*count)++;

    printf("Contact has been added successfully!\n\n");

    return contactList;
}

struct Contact *DeleteContact(struct Contact *contactList, int *count, int *capacity) {
    char name[100];
    printf("Enter the name of the contact you want to delete: ");
    scanf(" %[^\n]", &name);

    int contactIsFound = 0;
    int contactIndex = 0;
    for (int i = 0; i < *count; i++) {
        if (strcmp(contactList[i].name, name) == 0) {
            contactIsFound = 1;
            
            break;
        }

        contactIndex++;
    }

    if (contactIsFound == 0) {
        printf("Contact doesn't exist\n\n");

        return contactList;
    }
    
    int numberOfContactsToShift = *count - contactIndex - 1;
    if (numberOfContactsToShift > 0) {
        memmove(&contactList[contactIndex], &contactList[contactIndex + 1], numberOfContactsToShift * sizeof(struct Contact));
    }

    (*count)--;

    if (*count + 10 == *capacity) {
        *capacity = *capacity - 10;
    
        struct Contact *temp = realloc(contactList, *capacity * sizeof(struct Contact));

        if (temp == NULL) {
            printf("Failed to shrink the contact list\n");
        }

        contactList = temp;
    }

    printf("Contact has been deleted successfully!\n\n");

    return contactList;
}

void DisplayContacts(struct Contact *contactList, int *count, int *capacity) {
    printf("Contact List Capacity: %d\n", *capacity);
    printf("Contacts Count: %d\n\n", *count);

    for (int i = 0; i < *count; i++) {
        printf("Contact %d:\n", i + 1);
        printf("Name: %s\n", contactList[i].name);
        printf("Phone Number: %s\n\n", contactList[i].phoneNumber);
    }

    printf("\n\n");
}

void SearchContact(struct Contact *contactList, int *count) {
    char name[100];
    printf("Enter the name of the contact: ");
    scanf(" %[^\n]", &name);

    for (int i = 0; i < *count; i++) {
        if (strcmp(contactList[i].name, name) == 0) {
            printf("Contact Found:\n");
            printf("Contact Name: %s\n", contactList[i].name);
            printf("Contact Phone Number: %s\n\n", contactList[i].phoneNumber);

            return;
        }
    }
    
    printf("Contact doesn't exist\n\n");
}

void SaveData(struct Contact *contactList, int *count, char *filePath) {
    FILE *fileToWriteTo = fopen(filePath, "w");

    for (int i = 0; i < *count; i++) {
        fprintf(fileToWriteTo, "Name: %s | Phone Number: %s\n", contactList[i].name, contactList[i].phoneNumber);
    }

    fclose(fileToWriteTo);

    printf("Data has been saved successfully!");
}
```

## Used TXT File:
The names and numbers were generated randomly using Google's AI Mode:
Name: James Smith | Phone Number: +52 72627346  
Name: Aarav Sharma | Phone Number: +81 73456446  
Name: Isabella Conti | Phone Number: +20 19420027  
Name: Hye-jin Park | Phone Number: +61 25507668  
Name: Elena Popova | Phone Number: +39 04748380  
Name: Lucas Silva | Phone Number: +91 10464369  
Name: Yuki Tanaka | Phone Number: +55 94708177  
Name: Robert Johnson | Phone Number: +353 18773500  
Name: Mateo Fernandez | Phone Number: +27 94952764  
Name: Sofia Rossi | Phone Number: +49 33627373  
Name: Amira Mansour | Phone Number: +41 69493319  
Name: Chen Wei | Phone Number: +82 79989198  
Name: Noah Johansson | Phone Number: +34 36804327  
Name: Liam O'Connor | Phone Number: +1 69292940  
Name: Oliver Wright | Phone Number: +971 69123689  
Name: Emilie Müller | Phone Number: +44 99148560  
Name: Maria Garcia | Phone Number: +33 95615141  
Name: Chloe Lefebvre | Phone Number: +46 37878105  
Name: Fatima Al-Sayed | Phone Number: +7 37381102  
Name: Ahmed Hassan | Phone Number: +86 32374903
