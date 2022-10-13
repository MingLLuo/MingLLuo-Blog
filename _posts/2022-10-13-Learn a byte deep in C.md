---
layout: article
title: Learn a byte deep in C(1)
mathjax: true
permalink: :title
---
> Notice! This passage won't talk a lot of control flow, primitive variable and so on. All here is the message to know code better, the way to deal with memory better, and to be a good tool user(Also good at  Ctrl-C/V, but another perspective! To correct errors)

#### Thanks before : 
- Brown CSCI0300
- Berkeley CS61C
- CMU 15-213
- ...

## Basic Info of Computer:
In physics term, our computer just the combination of special material such as metal, plastic, glass, silicon... 

You can learn the way to construct them as a productivity tool in future study, maybe not in class🤣
Our computer contains important components such as **CPU**, **Memory**, **Devices**
-   A **CPU**, which contains one or more processors. These are the pieces of silicon that actually run programs and act.
-   **Memory**, which is a form of storage, and the **only** form of storage that processors can access directly.
-   **Devices**, which allow the computer to store data across reboots and power failures (harddisks and SSDs) and interact with the outside world (screens, keyboards, speakers, WiFi controllers).
These components need to work together to achieve things, and we need to understand them all in order to understand why code and systems behave the way they do.
### But
We are new in CS. We can't digest this part of knowledge. Fine, Let go in **memory** first, and I will try to help you construct what you need.

## A new way to inspect Computer: Binary
We like use power of ten in our daily life. But in computer, this black box only use 0 and 1, 😡, what a pity.(Just a joke). 
#### Position notation
A number 123 will be represent as $1\*10^2 + 2 * 10^1 + 3 * 10^0=(123)_{10} $ in **power of ten**.
In **power of two**, it will be represent as $ 1\*2^6 + 1\*2^5 + 1\*2^4 + 1\*2^3+0\*2^4+1\*2^1+0\*2^0=(1111010)_2 $
You may find some interesting things happen when facing arithmetic operation.
##### Terminology 
- We call a unit number base on 0 or 1 a **bit**.
- A collection with 8 bit is a **Byte**.
Using binary bits has many advantages: for example, error correction is much easier if presence or absence of electric current just represents "on" or "off". This choice influences many layers of hardware and system design.

All variables and data structures we use in our programs, and indeed all code that runs on the computer, need to be stored in these **byte-sized memory boxes**. How we lay them out can have major consequences for **safety** and **performance**!
![pic](/assets/images/pic/box.png){:class="img-responsive"}
(A graph o byte-sized memory boxes)

8 bit can represent a number range from 0-255.
You may calculate it by $1+2+4+...+128 = 255$. 
Why not $(11111111)_2 + 1 = (100000000)_2$
a **short** take two box to store. an **int** take four.


## Interpret Bytes in Memory
>The only place where a computer will store information is **memory**. It's up to the systems software and the programs that the computer runs to decide what these bytes actually **means**. 
They could be program code, data (integers, strings, images, etc.), or "meta-data" used to build more complex data structures from simple memory boxes. Understanding how complex programs boil down to bytes will help you debug your program, and will make you appreciate why they behave the way they do.

Our computer is silly! We need to tell the way to transform byte into different datatype. Or they are the data-bits that wait for invoke.
```c
// simple file : add.c
#include <stdio.h>   // <== import standard I/O (fprintf, printf)
#include <stdlib.h>   // <== import standard library

int main(int argc, char* argv[]) {  
//starting point of our program
    if (argc <= 2) {
        fprintf(stderr, "Usage: add A B\nPrints A + B.\n");   
	// print error message if arguments are missing. "\n" is a newline character!
        exit(1);
    }

    int a = strtol(argv[1], 0, 0);  // string to long?
	//covert first argument (string) to integer
    int b = strtol(argv[2], 0, 0);  
	//same for second argument
    printf("%d + %d = %d\n", a, b, add(a, b));  
	//invoke add() function, print result to console
}
```
[C library function - strtol()](https://www.tutorialspoint.com/c_standard_library/c_function_strtol.html)

If we try to compile it...
```bash
$gcc -o add add.c
#ERROR
Undefined symbols for architecture arm64:
  "_add", referenced from:
      _main in add-8fd8e1.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
There's an error, because we haven't actually provided an `add()` function.
```c
#include <stdio.h>
#include <stdlib.h>

int add(int a, int b);

int main(int argc, char* argv[]) {
    if (argc <= 2) {
        fprintf(stderr, "Usage: add A B\nPrints A + B.\n");
        exit(1);
    }

    int a = strtol(argv[1], 0, 0);
    int b = strtol(argv[2], 0, 0);
    printf("%d + %d = %d\n", a, b, add(a, b));
}
```
We add a declaration here: telling the compiler "there will be a function called `add()`, and you'll find out about its implementation later". All functions and variables in C have to be declared when you first use them, but they do not have to be defined.
```c
// addf.c
int add(int a, int b) {
    return a + b;
}
```
We need to tell compiler to look `addf.c` also.
```bash
$ gcc -o add add.c addf.c
```
The compiler first compiles `add.c` into a file called `add.o`, and then compiles `addf.c` into a file called `addf.o`. These files don't contain human-readable text, but binary bytes that the computer's CPU (central processing unit) understands to execute.

## Code as data: adding numbers
The CPU needs to know what calculations to run, and we tell it by putting bytes in memory that the CPU interprets as _machine code_, even though in other situations they may represent data like numbers or an image.

And with the right sequence of magic bytes in the right place, we can make almost any piece of data in memory run as code.
Let's see the function add in addf.c follow these steps.
1. Type `make`
2. Look magic bytes by `objdump -S addf.o`

You can see the add function encode as these bytes(might be different in different architecture)

```bash
$ objdump -d addf.o

addf.o:     file format mach-o-arm64
Disassembly of section .text:

0000000000000000 <_add>:
   0:   0b000020    add w0, w1, w0
   4:   d65f03c0    ret

$ objdump -d addf.o
addf.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 :
   0:   8d 04 37                lea    (%rdi,%rsi,1),%eax
   3:   c3                      retq
        ^                       ^
        | bytes in file         | their human-readable meaning in x86-64 machine language
        | in hexadecimal        | (not stored in the file; objdump generated this)
        | notation
```

Let's focus on the second code.

```c
// new addf.c (x86-64)
const unsigned char add[] = { 0x8d, 0x04, 0x37, 0xc3 };

// new addf.c (mach-o-arm64)
const unsigned char add[] = { 0x0b, 0x00, 0x00, 0x20, 0xd6, 0x5f, 0x03, 0xc0 };
```
We aren't writing a function in the C programming language, we're just defining an array of bytes called `add`. Do you think our `add` program will still work?
It work! Because we are manually storing **the exact same bytes** in memory that the compiler generates when compiling our `add` function into machine instructions. The processor doesn't care that we were storing an array of data there – if we tell it to go an execute these bytes, the dumb silicon goes and does as it's told!
## Pointers!
An address may occupy 8 bytes. Where would we store such an address? Well, we will store it in a variable itself. Where does that variable live? It better be in memory, too! In other words, the 8 bytes corresponding to the address will **have a memory location of their own**. We refer to such memory locations that hold addresses as **pointers**, because you can think of them as **arrows** pointing to other memory boxes.

In terms of C types, a type followed by an asterisk(\*) corresponds to a pointer. For example, `int*` is a pointer to an integer. An `int*` itself occupies 8 bytes of memory (since it stores an address), and it points to the first byte of a 4-byte sequence of memory boxes that store an `int`.

Similarly, a `char*` is a pointer to a memory cell that holds a `char` value (`char` is the C type name for a single-byte value). Since strings are represented as **sequences** of 1-byte characters in memory (this is where they type name comes from!), the type of a string in C is actually a pointer to the first character in the string, or a `char*`.

### PPS:
(Addition material for curious reader to read)
You may question. How can we print the content with other language? Chinese, Japanese, and so on.
##### Socially Responsible Computing: Character Sets
ASCII is the **American** Standard Code for Information Interchange. However, ASCII is only meant for English and it does not support accents or characters from non-Latin alphabets.
Programmers often turn to extended character sets (most commonly **Unicode**, and specifically its [UTF-8](https://www.cl.cam.ac.uk/~mgk25/unicode.html#utf-8) encoding) to create more inclusive applications.
When programming for extended character sets, some of the standard library support functions for single character (ASCII) strings will not work as intended. C provides support for multibyte characters through their `wchar_t` struct, which represents a character in 4 bytes. However, since many extended character sets have variable length characters (for example, UTF-8 encodes characters in 1-4 bytes), using `wchar_t` leads to wasted space in memory. Moreover, programs often receive UTF-8 text as input that isn’t nicely laid out in 4-byte chunks, and must still be able to process it. (As an example, consider a website that processes input from a form: users may put strings like “γνωρίζω” or “❤️😺🫠” into the form!)

UTF-8 characters follow a set byte format based on their length. 1 byte UTF-8 characters are meant to be compatible with ASCII, which means that they always begin with a 0 bit. For 2-4 byte UTF-8 characters, the number of leading 1s indicates the length of the character (in bytes). Each subsequent byte begins with the bits 10. You can find an overview of the format below:

| Length (bytes) | Encoding (binary) |
| -------------- | ----------------- |
| 1              | 0xxxxxxx          |
| 2              |110xxxxx 10xxxxxx        |
| 3               |  1110xxxx 10xxxxxx 10xxxxxx    |
|4 |11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|
{:.mbtablestyle}
![relevant xkcd Unicode](/assets/images/pic/relevant%20xkcd%20Unicode.png)