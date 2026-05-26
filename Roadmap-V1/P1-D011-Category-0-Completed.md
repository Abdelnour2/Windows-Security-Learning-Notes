# Phase 1 - Day 11: Thursday 16 April 2026
## Summary:
Researched all the points mentioned in Category 0 (C Foundations).

## Documentation:
Today, I’ll try to learn all the foundational concepts covered in Category 0! I’ll be learning them point by point, so let’s start:

### 1. Pointers:
When I first learned the basics of C, I’ve learned that pointers are variables that stores the memory address of another variable by using * to indicate a pointer, and & to indicate the address.

But what I didn’t understand is why and how I’ll use this feature?

I started researching and found this [video](https://www.youtube.com/watch?v=2ybLD6_2gKM). The guy explained that a memory has addresses (the location), and values (the value of each address). For example, Address 0x1000 and value 0x4, means that the value 0x4 lives at address 0x1000. In C, this can be like this: int x = 4;

A pointer will be to store an address in the value cell in memory. So for example, the address 0x1004 can have a value of 0x1000. In C, this can be like this: int *pX = &x;

A pointer is basically a value in memory that is an address. The explanation of “int x = 4;” is “An integer named x is set to 4”. And the explanation of “int *pX = &x;” is as follows:
- **int:** An integer.
- ***:** When a * is placed next to a type, it modifies the type making the variable a pointer to that type.
- **pX:** the name of the variable. A naming convention for pointers is to start the name with p, then the name of the variable pointing to. So pX means a Pointer to X.
- **=:** is set to
- **&:** the address of
- **x:** The variable we are pointing to.

The full explanation of “int *pX = &x;” is “An integer pointer named pX is set to the address of x”.

By doing this, now we can access x by reference instead of by value. For example, if we want to copy this variable to another one, we can say “int y = *pX;”. When a pointer is alone next to the pointer, it acts as a dereference operator, meaning that it goes to the address it is pointing to, and grabs its value. So the explanation of the line of code is “An integer named y is set to the thing pointed to by pX”.

After the video, I solidified my understanding of pointers but I still don’t get where I’ll use them! Or even the definition where there is ** in a sentence! So I kept digging!

Using Google’s AI Mode, my questions have been answered!

**The Why or Where?**  
First, it explained that in C everything is passed by value. If we pass a variable to a function, C makes a complete copy of it. This is slow and uses double the memory. So pointers solve 3 main problems:
- **Modifying Data:** If we want a function to change a variable defined in main, we can’t just pass the variable since it only changes the copy. We need to pass the address (pointer) so the function can reach the original variable and change it.
- **Efficiency:** Instead of copying a massive structure, we can pass a small address.
- **Dynamic Memory:** When we don’t know how much memory we need until the program is running, we ask the OS for a chunk of memory. The OS gives us back a pointer to the start of that chunk.

**The **:**  
This is basically a pointer to a pointer. Let’s say we have x as the main variable, pX is a pointer to x, and ppX as the pointer that points to the pointer that points to x.

Why would we ever use that?
The most common use is when we want a function to change where a pointer is pointing. If we want to swap which variable pX is pointing at from inside a function, we need to pass the address of pX itself.

### 2. Arrays and Pointer Arithmetic:
I know the syntax of arrays in C, so what I researched here is the logic behind it! Using Google’s AI Mode. I learned that When declaring an array, we are setting aside a contiguous block of memory. The name of the array is actually a pointer to its first element.

Let’s take this array for example: int numbers[3] = {10, 20, 30};. If numbers starts at memory address 0x1000, we will have:
- **0x1000:** contains numbers[0] (10)
- **0x1004:** contains numbers[1] (20)
- **0x1008:** contains numbers[2] (30)

Writing numbers[i] is actually a “nice/easier writing”, let's call it, made for developers to use arrays. What actually happens is *(numbers + i). This sentence means “Start at the starting address of numbers (the start of the array), skip i number of elements. Then, once you land on the needed address, dereference it  (grab the value)”.

When we say numbers + 1, or + 2, we don’t mean literally add 1 or 2 to the memory address. Instead, it means skip 1 element. For example, an int is usually for bytes. So + 1, in this case, is skipping 4 bytes to go to the next element at 0x1004 (if the array starts at 0x1000).

The second thing we need to point out on arrays is how we pass them to functions. In functions we write: void func1(int *arr, int size) {...}

We do this because as I mentioned, the name of the array is literally a pointer to its first address. For that reason we ask for a pointer, not an array. And for that reason we pass the array’s size too, since the function doesn’t know how many elements this array has.

### 3. Heap:
Using Google’s AI Mode I learned that to ask the OS for memory, we need to use malloc. Malloc takes the size we want and returns a pointer to the starting address of the Heap space we asked for. For example: int *arr = malloc(5 * sizeof(int));. We asked the OS to give us a space in the Heap that can hold 5 integers.

Writing sizeof(data_type) is very important since it ensures that the code works on different computers. Since for example int is not always 4 bytes. Writing sizeof(int) handles this case.

An important concept for the Heap is that once we ask for space, we need to return it once done with it. If we lose the pointer or don't return the space, that leads to memory leaks.

Returning the space is done by using the free function. It only asks for the pointer we have. So in this case: free(arr). And it’s good practice after using free to also set arr to null by: arr = NULL;.

Other functions that are used are calloc and realloc. Calloc is exactly like malloc, but with one more functionality: it clears the space we asked for before giving it to us. By asking for space using malloc, we get that space but it might already contain some data. Calloc clears this data so we always get a fresh empty space to work with. Calloc takes 2 arguments instead of 1, the count, and the size. In malloc, we calculated the space we want in the Heap by for example saying 5 * sizeof(int). Calloc does this calculation automatically, it just asks us for the 2 parts individually, so in this case it would be calloc(5, sizeof(int)). Since calloc clears the space before giving it to us, it is a bit slower than malloc.

Realloc is used to change the size we asked for on the Heap. Let’s continue with our “array of size 5” example. Let’s say we have 6 elements now, or 10, or … what do we do? We can ask the OS to give us more room by writing: realloc(arr, 20 * sizeof(int));. We gave realloc the pointer we already had for the space, and we asked for the new space (20 integers).

But there are two things to point out on using realloc:
- First, when there is not enough space for expansion, realloc will return NULL, overwriting our existing pointer to the space resulting in memory leaks. So it’s a better approach to copy the pointer before doing that like this for example:“int *temp = realloc(arr, 20 * sizeof(int));”.
- Shrinking the size we asked for. Let’s say we asked for 10 integers. And we made an array and we filled it with 10 integers. If we shrink the asked size let’s say to 5, arr[0] to arr[4] are still with me. But arr[5] to arr[9] are lost. They are not erased, but we lost access to them!

### 4. Strings:
In C, a string is just an array of characters that ends with a null terminator “\0”. Let’s say we want to store the word Hello. It would be like this:
- 0x1000: ‘H’
- 0x1001: ‘e’
- 0x1002: ‘l’
- 0x1003: ‘l’
- 0x1004: ‘o’
- 0x1005: ‘\0’

But why the null terminator? Functions like printf and strlen and … don't know how long the array of characters is. They just read the array until they see a null terminator.

Let’s say we have this string “Hello World”, and we edit the first character to \0. The program will treat the string as an empty string. If we want to print it, we would have nothing printed.

Since strings are arrays of characters, they are pointers. As I mentioned before, the name of the array is a pointer to its starting address. So because of that, we can’t compare strings with == or copy them with =. Instead, we have to use special functions:
- **strlen(str):** Counts the number of characters found until it reaches the null terminator (it excludes \0).
- **strcpy(dest, src):** It copies the source string and stores it in the destination string.
- **strcmp(s1, s2):** It compares the 2 strings and returns the result.

### 5. Struct:
Structs are basically mini classes. They are a custom data type that holds multiple variables related to it. Let’s say we have a game and we need to store the player's data like name, health, and speed. We can do something like this:
```c
struct Player {
	char *name;
	int health;
	int speed;
};
```

And to define a player, we can write this:
```c
struct Player player1 = {“Player 1”, 100, 20}
```

Structs are often large, for that reason we pass them as pointers to other functions. And when they are passed as a pointer, we use the -> to access their data. Example:
```c
void TakeDamage(struct Player *p) {
	p->health -= 10;
}
```

p->health is the same as writing (*p).health. But the -> is a new operator and is advised to use it.

Since structs can’t have functions, the way to do it in C, is to pass the struct as a pointer the same way as the previous example. Example 2:
```c
// function
void Player_TakeDamage(struct Player *p, int amount) {
p->health -= amount;
}

// Usage
struct Player player1 = {"Player 1", 100};
Player_takeDamage(&player1, 10);
```

Modern languages implemented a layer on top of this to make programming easier for developers. Instead of passing the address of the object to the function, they use keywords like this and self.

Sometimes, we might malloc a struct, which let’s say also have a name field that also needs to be malloced. After usage, we need to free both mallocs to avoid memory leaks. And the way to do it is the inner malloc first (the name field), then the struct malloc. The reason is that if we free the struct malloc first, the pointer to the struct is lost, so we can’t access the name field inside it leading to memory leaks.

### 6. Reading Files:
In C, everything is treated as a Stream (sequence of bytes). On opening files, the OS gives us a pointer to a special struct called a FILE.

To open a file:“FILE *fptr = fopen(“file_name.txt”, “r”);”. The r stands for read. We are only reading from the file, not writing.

Like malloc, fopen can fail, so we always have to check if the pointer is NULL or not before using it.

To grab the data from the file, there are multiple ways:
- **fgetc(fptr):** Reads one single character at a time.
- **fgets(buffer, size, fptr):* Generally the safest and most common choice for reading text files. It reads an entire line of text until it hits a newline or reaches a specific buffer limit, preventing common buffer overflow security issues.
- **fread(...):** Used when we want to read raw bytes or non-text data. It reads a fixed number of bytes directly from memory without looking for new lines or special characters. It used to read images, audio files, or saved program structs.

Files need to be closed once done like heap memory needs to be freed! To close a file we use “fclose(fptr);”.

When reading files, malloc is often used a lot. Let’s say for example we have a name field inside of a struct. We might set a fixed size of 100 characters, but someone decides to write 200! This leads to buffer overflow if not handled. Or someone enters 4 characters, that’s 96 bytes wasted!

### 7. Command Line Arguments:
We can do this by letting the main functions accept arguments. To do this:“int main(int argc, char *argv[]) { … }”.
- **argc:** Stands for Argument Count. It is an integer representing how many things were typed. Noting that the name of the program counts as an argument. So there is always at least 1 argument present.
- **argv:** Stands for Argument Vector. It’s an array of strings (so it’s a pointer of pointer, which can be written as char **argv, or char *argv[]).

Let’s say we ran the program with: ./main.exe data.exe 10
- argc = 3
- argv[0] = ./main.exe
- argv[1] = data.txt
- argv[2] = 10

Note that 10 is a string not an integer, so we need to transform it to an integer by using a function like atoi which stands for Ascii to Integer.

That concludes Category 0. To practice I thought of making a simple program that combines every concept. Google’s AI mode gave me this idea: “Phonebook Program”.
