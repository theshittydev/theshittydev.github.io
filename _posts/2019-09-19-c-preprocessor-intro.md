---
title: How to be a Lazy Developer with the C Preprocessor
layout: post
tags:
  - C
  - Preprocessor
---
Alright - finally decided to get around to writing one of these blog posts!

Today's fun-fact of poor development comes to you from the fun world of GCC and the beauty that is macros.

## Intro
For the unindoctrinated, turning a .c file into a functioning and runable program can be broken down into ~4 separate stages:   
  1. Preprocessing
  1. Compiling
  1. Assembling
  1. Linking

The scope of this post is limited to the 1st stage, but if you want to learn more about the other 3 stages, you can search for it on the internet like everyone else.

### Preprocessing
The Preprocessing stage is responsible for the *pre*-processing of a given source file (as the name might suggest 😉 ). Before moving on to the next stage, the compiler:
  * brings in external source files
  * runs code in a limited language
  * strips comments
  * changes some human-readable syntax to more of what the computer expects to see

The code that is run in that limited language?  That's where most of today's fun is going to reside!

### Macros
Macros and preprocessor directives look similar, but perform different actions/purposes for the preprocessor.

```c
// These are directives
#include <stdio.h>
#if !__unix__
#  error Use a real operating system!
#endif /* !__unix__ */

// These are macros
#define my_macro 1
#define another_macro(p,x) something_function_##x((p))
#undef my_macro
```

Directives tell the preprocessor to do something - or to conditionally do something (in the case of `#if`, `#elif`, and `#else`). Macros are ways of dynamically filling things in throughout the code

### Simple Example

Macros can be useful to avoid having to change constants all over a block of code, and limit the changes to ONE place.

Here's two examples. One using macros, and one not:

#### Without
```c
#include <stdio.h>

int main(void)
{
    int i;
    for(i=0;i<50;i+=5)
    {
        printf("I+%d*n is: %d\n", 5, i);
    }
    return 0;
}
```

#### With
```c
#include <stdio.h>

#define INCREMENT 5

int main(void)
{
    int i;
    for(i=0;i<INCREMENT*10;i+=INCREMENT)
    {
        printf("I+%d*n is: %d\n", INCREMENT, i);
    }
    return 0;
}
```

#### So What?
It's a simple example here - changing the increment value isn't too hard. But, I'm a shitty dev. I want to do less work when someone decides they want to change a number. Changing one number, instead of 3 (`i<60`, `i+=6`, and the `printf` line) makes much less work for maintaining the code and makes it less bug prone too!

### Several Useful Examples
Macros are pretty powerful, and save a lot of work on the behalf of the developer. I've included a few useful examples that I use when I write code

#### Debug Printing!
I use `log()` for debugging/production logging. When debugging, I add -DDEBUG=1 to the Makefile and that suddenly prints all debugging messages with the .c file, function it was called from, and line number the print statement was on.
```c
#include <stdio.h>

#ifndef DEBUG_FD
// If you want to use a different file descriptor than stderr, #define it before #including this file
#define DEBUG_FD 2
#endif /* DEBUG_FD */

#ifdef DEBUG
#   define log(...)                                                     \
{                                                                       \
    dprintf(DEBUG_FD, "%s:%d:%s: ", __FILE__, __LINE__, __func__);      \
    dprintf(DEBUG_FD, __VA_ARGS__);                                     \
}
#else
#   define log(...)
#endif /* DEBUG */
```

#### Function and Variable Attributes
These are more legible/prettier to look at!
```c
/* Variable Attributes */
#define PACKED __attribute__((packed))

/* Function Attributes */
#define INLINE __attribute__((always_inline))
#define DEPRECATED __attribute__((deprecated))
#define NOINLINE __attribute__((noinline))
#define NORETURN __attribute__((noreturn))

/* BOTH */
#define ALIGNED(n) __attribute__(( aligned(n) ))
#define HIDDEN __attribute__((visibility("hidden")))
#define UNUSED __attribute__((unused))

/* Examples */
//#if 0
typedef struct PACKED {
    unsigned char a;  // 1-byte char
    unsigned short b; // 2-byte short
    unsigned int c;   // 4-byte int
} seven_byte_struct;

void NORETURN exit_func(void);
//#endif /* 0 */
```
### Conclusion
What's been covered here today is a very brief introduction to the preprocessor and what you do to make GCC do your work for you. I'll get around to covering some more complicated uses of the preprocessor/compiler in a future blog post.

For now, stay 💩y.
