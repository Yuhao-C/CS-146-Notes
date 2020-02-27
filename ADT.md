# *ADT*s (**A**bstract **D**ata **T**ype) in C

C doesn't have modules - C has files

## Implement ADT Sequence in C

### Operations

- empty sequence
- `insert(s, i, e)`: insert int `e` at index `i` in `s`
  - precondition: `0 <= i <= size(s)`
- `size(s)`: number of elements in `s`
- `remove(s, i)`: remove item from index `i`
  - precondition: `0 <= i <= size(s) - 1`
- `index(s, i)`: returns `i`th element of `s`
  - precondition: `0 <= i <= size(s) - 1`

Want: no limits on size - sequence can grow as needed

### Implementation options

- linked list
  - easy to grow
  - slow index
- array
  - fast index
  - hard to grow

### Approach: partially-filled heap array

```c
// sequence.h

struct Sequence {
  int size; // how many items are in use
  int *theArray;
  int cap; // how much can we hold?
};
struct Sequence emptySeq();
int seqSize(struct Sequence s);
void add(struct Sequence *s, int i, int e);
void remove(struct Sequence *s, int i);
int index(struct Sequence s, int i);
void freeSeq(struct Sequence *s);
```

```c
// sequence.c

#include "sequence.h"
struct Sequence emptySeq() {
  struct Sequence res;
  res.size = 0;
  res.theArray = malloc(10 * sizeof(int));
  res.cap = 10;
  return res;
}
int seqSize(struct Sequence s) {
  return s.size;
}
void add(struct Sequence *s, int i, int e) {
  for (int n = s->size; n > i; --n) {
    s->theArray[n] = s->theArray[n - 1];
    // both -> and [] are postfixes
  }
  ++s->size; // postfix before prefix, i.e. ++(s->size)
}
void remove(struct Sequence *s, int i) {
  --s->size;
  for (int n = i; n < s->size; ++n) {
    s->theArray[n] = s->theArray[n + 1];
  }
}
int index(struct Sequence s, int i) {
  return s.theArray[i];
}
```

- `#include`
  - `<stdio.h>` - angled brackets: in "standard place" (some dir in the file system)
  - `"sequence.h"` - quotes: in current dir

```c
// main.c

#include "sequence.h"
int main() {
  struct Sequence s = emptySeq();
  add(s, 0, 4);
  add(s, 1, 7);
  // OK, but not immune to tampering/forgery

  s.size = 8; // tampering!

  struct Sequence t;
  t.size = 10;
  t.cap = 20;
  // forgery!
}
```

Can we prevent this?

- keep the details of `struct Sequence` hidden
- declare, but not define, the `struct`?

### Prevent tampering/forgery

```c
// sequence.h

struct Sequence;
struct Sequence emptySeq();
...
```

```c
// sequence.c

#include "sequence.h"
struct Sequence {
  int size, cap;
  int *theArray;
};
...
```

```c
// main.c

#include "sequence.h"
int main() {
  struct Sequence s; 
  // ✗: compiler doesn't know enough about Sequence (needs size!)
  ...
}
```

Fix

```c
// sequence.h

struct SeqImpl;
typedef SeqImpl *Sequence; // A Sequence is a pointer to a SeqImpl
Sequence emptySeq();
int seqSize(Sequence s);
void add(Sequence s, int i, int e);
void remove(Sequence s, int i);
int index(Sequence s, int i);
void freeSeq(Sequence s);
```

```c
// sequence.c

#include "sequence.h"
struct SeqImpl {
  int size, cap;
  int *theArray;
};
...
```

```c
// main.c

int main() {
  Sequence s; // pointer - ✓ OK!
  ...
}
```

### What happens when the array is full?

```c
void increaseCap(Sequence s) {
  if (s->size == s->cap) {
    s->theArray = realloc(s->theArray, 2 * s->cap * sizeof(int));
    s->cap *= 2;
  }
}
void add(Sequence s, int i, int e) {
  increaseCap(s);
  ...
}
```

#### Helper function

`increaseCap()`

- `main` should not be calling this
- how do we prevent it?
  - leave it out of the header file

But

```c
// main.c

#include "sequence.h"
void increaseCap(Sequence s); // What if main declares its own header?
...
```

Solution

```c
// sequence.c

static void increaseCap(Sequence s) { ... }
```

`static` function
- means only visible in this file
- prevents other files from having access, even if they write their own header

#### `realloc()`

- increases a block of memory to a new size
- if necessary, allocates a new, larger block and frees the old block (data copied over)

How big should we make it?

- one bigger? (must assume each call to realloc causes a copy, $O(n)$)

If we have a sequence of adds (at the end, so no shuffling cost)  
number of steps is $n+(n+1)+(n+2)+...+(n+k)$  
$O(n^2)$ total cost  
$O(n)$ per `add`

What if, instead, we double the size?  
Each add still $O(n)$ worst case.

But...

#### Amortized Analysis

places a bound on a **sequence** of operations, even if an **individual** operation can be expensive.

If an array has a `cap` of $k$ and is empty

| Operation | `cap` | Steps |
| --------- | ----- | ----- |
| $k$ inserts cost of $1$ each   | $k$  | $k$ steps    |
| $1$ insert costs $k+1$         | $2k$ | $k+1$ steps  |
| $k-1$ inserts cost of $1$ each |      | $k-1$ steps  |
| $1$ insert costs $2k+1$        | $4k$ | $2k+1$ steps |
| $2k-1$ inserts cost $1$ each   |      | $2k-1$ steps |
- $1$ insert costs $4k+1$, ($4k+1$ steps) - `cap` now $8k$
- ...
- $2^{j-1}k-1$ inserts cost $1$ each ($2^{j-1}k-1$ steps)
- $1$ insert costs $2^jk+1$, ($2^jk+1$ steps) - `cap` now $2^{j+1}k$

Total number of insertions:  
$k+1+(k-1)+1+(2k-1)+1+...+(2^{j-1}k-1)+1$  
$= k+1+k+2k+...+2^{j-1}k$  
$= k+1+k(1+2+...+2{j-1}$  
$= k+1+k(2^j-1)$
$= k+1+2^jk-k$
$= 2^jk+1$

Total number of steps:  
$k+(k+1)+(k-1)+(2k+1)+(2k-1)+(4k+1)+...+(2^{j-1}k-1) + (2^jk+1)$  
$= k+2k+4k+...+2^jk+2^jk+1$  
$= k(2^{j+1}-1) + 2^jk+1$  
$= 2^{j+1}k+2^jk-k+1$  
$= 2*2^jk+2^jk-k+1$  
$= 3*2^jk-k+1$

So the number of steps per insertion is:  
$$
\frac{3*2^jk-k+1}{2^jk+1} ≈ \frac{3*2^jk}{2^jk} = 3
$$

## Application of Vectors

### ADT Map/Dictionary (Mutable version)

- `(make-map)`: no parameter
  - precondition: `true`
- `(add): params: Map M, key k, value v
  - pre: `true`
  - post: if $\exists v'$ such that $(k, v') ∈ M$, then $M ← (M ∖ (k, v')) ∪ \{(k,v)\}$ else $M ← M ∪ \{(k, v)\}$