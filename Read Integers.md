# Read Integers

## First Attempt

```C
#include <stdlib.h>

int getIntHelper(int acc) {
  char c = getchar();
  if (c < '0' || c > '9') return acc;
  return getIntHelper(acc * 10 + c - '0');
}
int getInt() {
  return getIntHelper(0);
}
```

The range of input for `getchar();` is `0~255`. Need to indicate to caller if we cant read a character. No way to do that if we can only return a char (then every return value has meaning). So return type is int so that we may return a value outside the range of a char to indicate unsuccess reads.

> ***Problem***: Our current `getInt()` eats up an extra character from the input stream. Does C have a fn like racket's peekChar? No... :(
>
> ***Solution***: We have `ungetc()` - stuffs the char back into the input stream. Does

```C
int peekChar() {
  int c = getchar();
  // Often EOF will be -1
  return c == EOF ? EOF : ungetc(c, stdin);
}
```

## Second Version

```C
#include <stdio.h>
#include <ctype.h> // character predicates

int getIntHelper(int acc) {
  int c = getChar();
  return isdigit(c) ?
    getIntHelper(acc * 10 + c - '0')
    (ungetc(c, stdin), acc));
    // comma operator - a, b = evaluate a, b
    // the result of this expr is b's reusult
    // similar to (begin a b) in racket
}

int getInt() {
  return getIntHelper(0);
}
```

> *What if there is whitespace before our number? Quite likely we'd like to get taht number ignoring the leading whitespace*

## Third Version

```C
#include <stdio.h>
#include <ctype.h>

void skipws() { // a void fn return nothing
  int c = getchar();
  if (isspace(c)) {
    skipws();
  } else {
    ungetc(c, stdin);
  }
}

int getIntHelper(int acc) {
  int c = getChar();
  return isdigit(c) ?
    getIntHelper(acc * 10 + c - '0')
    (ungetc(c, stdin), acc));
}

int getInt() {
  skipws();
  return getIntHelper(0);
}
```
