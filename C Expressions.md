# C Expressions

- [C Expressions](#c-expressions)
  - [Statements](#statements)
    - [Blocks - Groups of Statements](#blocks---groups-of-statements)
    - [Functions](#functions)
      - [Function calls](#function-calls)
      - [Function Declaration](#function-declaration)
  - [Header](#header)
  - [Types](#types)

***

- infix operations `1 + 2`
- function calls `f(a, b, c)`
- Operator prececence - follow mathematical conventions
- `printf("%d\n", 5)`
  - `5` substituted for `%d`
  - `%d` displays a base-10 integer value
  - produces the number of characters printed

***

## Statements

- `printf("%d\n", 5);` - (semicolon makes an expr a statement)
- `5+4;` - computes 5+4, does nothing with result
value of expr is ignored
- expressions are evaluated only for their side effects
- `return 0;` - produce the value 0 as the result of the function, return control immediately to the caller
- `;` - empty statement (do nothing)
  
### Blocks - Groups of Statements

```C
{
  stmt1;
  stmt2;
  ...
  stmtn;
}
```

### Functions

```C
int f(int x, int y) {
  printf("x = %d, y = %d\n", x, y);
  return x + y;
}
```

#### Function calls

- `f(3, 4)` - expression - evaluates to 7
- `f(3, 4);` - statement - expr is evaluated but answer is never used

> *A programs is a sequence of functions, starting with main()*

#### Function Declaration

- C enforces declaration before use - can't use a fn/variable/etc. until you tell C about it.
- One Solution: put f first. Works, but overkill, not necessary
- C only requires declaration (statement of existence) before use. So, we can simply write before main int f(int, int); // fn prototype or fn header

***

## Header

```C
#include<stdlib.h>
```

- `#include` is not part of the C language
- It directs to the C preprocessor (which runs before compilation)
- It is like macro expansion in Racket
- `#include<file.h>` copys the contents of `file.h` to the program
- `stdio.h` contains declarations for `printf` and other I/O functions
- A *linker- takes care of linking function declarations from header files to their definitions automatically
  - if you write your own module, you need to tell the linker about it

***

## Types

Everything in our memory (where these value are stored) are just numbers. Characters are restricted form of integers.

- `int` - varies but typically 32 bits
- `char` - 8 bits (256 distinct values)
- `'0'` - the character 0, numerically 48 according to ASCII
- `'9'` - the character 9, numerically 57 according to ASCII
