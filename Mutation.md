# Mutation

- [Mutation](#mutation)
  - [Mutation in Racket](#mutation-in-racket)
    - [Example](#example)
    - [Implementation in impure Racket](#implementation-in-impure-racket)
    - [Application](#application)
      - [Consider](#consider)
  - [Mutation in C](#mutation-in-c)
    - [Assignment](#assignment)
    - [Global Variables](#global-variables)
    - [Repetition](#repetition)
      - [recursion](#recursion)
      - [while loop](#while-loop)
      - [Common Pattern](#common-pattern)
      - [Updating Counters](#updating-counters)
  - [Intermediate Mutation (Racket)](#intermediate-mutation-racket)
  - [Intermediate Mutation (C)](#intermediate-mutation-c)
    - [Structure in C](#structure-in-c)
    - [Pointer](#pointer)
  - [Advance Mutation](#advance-mutation)
    - [Mutating structures and lists](#mutating-structures-and-lists)
    - [Semantics](#semantics)
    - [Rethinking define](#rethinking-define)
    - [Vector (Racket)](#vector-racket)
      - [Memory Allocation](#memory-allocation)
    - [Array (C)](#array-c)
    - [Pointer Arithmetic](#pointer-arithmetic)
  - [(Closer to) Real Memory Layout](#closer-to-real-memory-layout)
    - [Stack](#stack)

***

## Mutation in Racket

```scheme
(define x 3)
(set! x 4)
x
=> 4
```

### Example

```scheme
(lookup 'Rob)
=> false
(add 'Rob 3)
(lookup 'Rob)
=> 3
```

### Implementation in impure Racket

```scheme
(define address-book empty) ;; global variable
(define (add name number)
(set! address-book
  (cons (list name number) address-book))
```

**Global data**: good for defining constants going to be used repeatedly.

- But not great with mutations
- Any part of the program could change global
- Affects the entire program
- Hidden dependencies between different parts of the program
- Harder to reason about programs

### Application

- **Caching**: Saving the result of computation to avoid repeating
- **Memoization**: maintaining a list or table of cached values

#### Consider

```scheme
(define (fib n)
  (cond
    [(= n 0) 0]
    [(= n 1) 1]
    [else (+ (fib (- n 1))
             (fib (- n 2)))]))
```

It is inefficient becasue recursive calls are repeated doing the same computation:

- `(fib 98)` called twice, `(fib 97)` called three times
- `(fib n)` is $\theta$`(f n)`
- To avoid repetition, we can keep an associated list of already computed value

```scheme
(define fib-table empty)

(define (memo-fib n)
  (define result (assoc n fib-table))
  (cond
    [result => second]
    ;; [cond => f] -> if cond is true, then evaluate (f cond)
    [else
     (define fib-n
       (cond
         [(<= n 1) n]
         [else (+ (memo-fib (- n 1))
                  (memo-fib (- n 2)))]))
     (set! fib-table (cons (list n fib-n) fib-table))
     fib-n]))
```

- Now fib-table is a global variable. Can we hide it?

```scheme
(define memo-fib2
  (local
    [(define fib-table empty)
     (define (memo-fib n)
       (define result (assoc n fib-table))
       (cond
         [result => second]
         [else
          (define fib-n
            (cond
              [(<= n 1) n]
              [else (+ (memo-fib (- n 1))
                       (memo-fib (- n 2)))]))
          (set! fib-table (cons (list n fib-n) fib-table))
          fib-n]))]
    (λ (n) (memo-fib n))))
```

- We can also keep the computed value in the accumulator

```scheme
(define (accum-fib-helper n acc)
  (define result (assoc n acc))
  (cond
    [result acc]
    [else
     (define next-acc (accum-fib-helper (- n 1) (accum-fib-helper (- n 2) acc)))
     (cons (list n (+ (second (assoc (- n 1) next-acc))
                      (second (assoc (- n 2) next-acc))))
           next-acc)]))

(define (accum-fib n)
  (second (assoc n (accum-fib-helper n (list (list 0 0) (list 1 1))))))
```

***

## Mutation in C

### Assignment

- assignment operator: `=`

```C
// mutate x to have the value 5
// equivalent to (set! x 5) in racket
x = 5
```

- `x = 4` sets x to 4, produces 4

```C
int main() {
  int x = 3;
  printf("%d\n", x); // print 3
  printf("%d\n", x = 4); // print 4
}
```

- advantage

```C
x = y = z = 7; // sets x, y, z to 7
```

- disadvantage that often leads to errors

```C
int main() {
  int x = 5;
  if (x = 4) { // error: should use "==" instead of "="
    printf("%d\n", x); // print 4
  }
}
```

- can leave variables uninitialized and assign later

```C
int main() {
  int x; // uninitialized
  x = 4; // assignment
}
```

```C
int x;
if (x == 0) { // Will this run or not?
              // don't know. x's value undefined
}
// Typically its value will be whatever was in the memory before it
```

***

### Global Variables

```C
int c = 0;

int f() {
  int d = c;
  c = c + 1;
  return d;
}

int main() {
  printf("%d\n", f()); // print 0
  printf("%d\n", f()); // print 1
  printf("%d\n", f()); // print 2
}
```

> ***Careful***:
> `printf("%d\n %d\n %d\n", f(), f(), f());`
>
> could produce: `0 1 2` or `2 1 0` or `1 0 2` ......

Can we hide c within f, as we did in fib-table?

```C
int f() {
  static int c = 0; // c is a global variable
  int d = c;
  c = c + 1;
  return d;
}
```

Global constants can still be helpful! We can force a variable to be constant in C

```C
const int passingGrade = 50;
```

***

### Repetition

#### recursion

```C
void sayHiNTimes(int n) {
  if (n > 0) {
    printf("hi!\n");
    sayHiNTimes(n-1);
  }
}
```

***

#### while loop

```C
void sayHiNTimes(int n) {
  while (n > 0) {
    printf("hi!\n");
    n = n = 1;
  }
}
```

>a loop - the body of a loop is a group executed repeatedly so long as the condtion remains true.

*Loops and recursions are mathematically equivalent.*

***

#### Common Pattern

```C
(initialize vars)
while (condition) {
  (body)
  (update)
}
```

or

```C
for (init; condition; update) {
  (body)
}
```

***

#### Updating Counters

- short hands

```C
c += 1; // c = c + 1;
c -= 2; // c = c - 2;
c *= 10; // c = c * 10;
c /= 2; // c = c / 2;
```

- incrementing and decrementing by 1

```C
// prefix: produces the new value of i
// ++i;
// --i;
int i = 1;
printf("%d", ++i); // prints 2

// postfix: produces the old value of i
// i++;
// i--;
int j = 1;
printf("%d", j++); // prints 1
printf("%d", j); // prints 2
```

>Postfix operation implies you must remember the old value. So, if you don't care about value produced, prefer to use prefix

***

## Intermediate Mutation (Racket)

What if we want to work with multiple address books?

```scheme
(define work '(("manager" 12345)
               ("director" 123456)))
```

Solution:

```scheme
(define home '())

(define (add-entry abook aname anumber)
  (set! abbok (cons (list aname anumber) abook)))

(add-entry home "Neighbour" 34567)

> home
> '() ; No Change!
```

- Code doesn't work
- Not immediately clear how to make it work
- substitution model does not explain the behavior
- Effectively, we are doing `(set! '() (cons (list "Neighbour" 34567) '()))`
- We can not change the meaning of an empty list
- To make this work, we need to use Lambda
- Do the same thing to creat a struct with one field called a box:
  - Two operations:
    - get the value in the box
    - set the value in the box

```scheme
(define (make-box v)
  (lambda (msg)
    (cond
      [(equal? msg 'get) val]
      [(equal? msg 'set)
       (lambda (newVal) (set! val newVal))])))

(define (get b) (b 'get))

(define b1 (make-box 7))

> (get b1)
> 7

> ((set b1) 5)
> (get b1)
> 5
```

Boxes are built in to racket

*Syntax:*

```scheme
exp = (box exp)
    | (unbox exp)
    | (set-box! exp exp) ; first argument doesn't need to be an id

(define work (box '(Personel 123456)))
(define (add book name number)
  (set-box! book (cons (list name number) (unbox book))))
```

*Semantics:*

```scheme
(box v) ; v is a value
=> (define _u v) ; u is a fresh name number
;; Convention: when we write an _ before a var, it means the var's value is not looked up during expr evaluation unless its after unbox

(unbox _n) ;; find (define _n v)
=> v
```

*e.g.*

```scheme
(define box1 (box 4))
(unbox box1)
(set-box! box1 true)
(unbox box1)

;; step 1
=> (define _u1 4)
=> (define box1 _u1)
=> (unbox box1)
=> (set-box! box1 true)
=> (unbox box1)

;; step 2
=> (define _u1 4)
=> (define box1 _u1)
=> (unbox _u1)
=> (set-box! box1 true)
=> (unbox box1)

;; step 3
=> (define _u1 4)
=> (define box1 _u1)
=> 4
=> (set-box! box1 true)
=> (unbox box1)

;; step 4
=> (define _u1 4)
=> (define box1 _u1)
=> 4
=> (set! _u1 true)
=> (unbox box1)

;; step 5
=> (define _u1 4)
=> (define box1 _u1)
=> 4
=> (set! _u1 true)
=> (unbox _u1)

;; step 6
=> (define _u1 4)
=> (define box1 _u1)
=> 4
=> (set! _u1 true)
=> true
```

## Intermediate Mutation (C)

Suppose we want to write a function

```C
void inc(int x) {
  x = x + 1;
}

int main() {
  int x = 1;
  inc(x);
  printf("%d", x); // prints 1, we want it to print 2
}
```

Racket solution is to put it in a box. What is a box in C?

### Structure in C

```C
struct Posn {
  int x;
  int y;
};

int main() {
  struct Posn p;
  p.x = 3;
  p.y = 4;
  printf("%d, %d", p.x, p.y); // prints 3, 4
}

// or
int main() {
  struct Posn p = {3, 4};
}

// but not
int main() {
  struct Posn p;
  p = {3, 4};
}
```

*Does it work like boxes in Racket?*

```C
void swap(struct Posn p) {
  int temp = p.x;
  p.x = p.y;
  p.y = temp;
}

int main() {
  struct Posn p = {3, 4};
  swap(p);
  printf("%d, %d", p.x, p.y); // still prints 3, 4
}
```

**The Problem**: C (and also Racket) pass parameters by a machanism know as call by value. The function operates on a copy of the values of the arguments, not the arguments themselves.

### Pointer

```C
void inc(int x) {
  ++x; // only the local copy of x is being mutated
}
```

So, something speical about boxes

- Not equal to the value it holds
- But can tell you how to get the value
- unbox = find the value

Finding a value? Where is that value located?

- in memory (RAM)
- Every value in memory has an address
- given the address we can find the value

So address could function as boxes. Instead of passing the value, pass an address

```C
void inc(int x) {
  x = x + 1;
}

int main() {
  int x = 1;
  inc(&x); // & is the "address-of" operator
  printf("%d", x);
}
```

A few things are wrong:

- Tpes don't match anymore. We must update the parameter's type to be "int* p" - a pointer to an int
- Body is also wrong, Now: `x = x + 1` means that add 1 to the address (whatever that means) and overwrite our local copy of the address with that value.

Correct Version:

```C
void inc(int* x) {
  *x += 1;
}

// or
void inc(int* x) {
  ++*x;
}

// wrong
void inc(int* x) {
  *x++; // = *(x++) address is incremented
}
```

Consider

```C
void swap(struct Posn p) {
  int temp = p.x;
  p.x = p.y;
  p.y = temp;
} // only the local copy is mutated
```

Now we can fix this using the pointer

```C
void swap(struct Posn* p) {
  int temp = *p.x; // wrong! postfix before prefix
  *p.x = *p.y;
  *p.y = temp;
  // *(p.x) = *(p.y) and they are not pointers, also p doesn't have x and y filed because it is a pointer
}

// correct version
void swap(struct Posn* p) {
  int temp = (*p).x;
  (*p).x = (*p).y;
  (*p).y = temp;
}

// or
void swap(struct Posn* p) {
  int temp = (*p).x;
  p->x = p->y; // p->x = (*p).x
  p->y = temp;
}
```

Now we can use a more sophisticated input function `scanf`

- `scanf("%d", &x);`
- reads x as a decimal integer
- skips leading whitespace
- returns the number of arguments successfully read

```C
int main() {
  int x, y;
  int n = scanf("%d %d", &x, &y); // whitespace means skipping any amount of whitespace, including zero
  printf("%d", n); // prints 2;
}
```

## Advance Mutation

### Mutating structures and lists

- In scheme: can mumate the part of a cons with set-car! and set-cdr!
- In racket: cons fileds are inmmutable, for mutable pairs we use `mcons`, `mset-car!`, and `mset-cdr!`
- For structs: provide the option `#:mutable`

```scheme
(struct posn (x y) #:mutable)
(define p (make-posn 3 4))
(set-posn-x! p 5)
(posn-x p) => 5
```

### Semantics

- before: `(make-posn v1 v2)` is a value
- now: `(make-posn v1 v2)` cannot simply be a value if its mutable. It has to behave more like a box.
- it turns out that in racket a struct itself is not boxed, but its fields are automatically boxed.

So we can rewrite `(posn v1 v2)` as

```scheme
(define _val1 v1) ; _ = no expansion
(define _val2 v2)
(define p (posn _val1 _val2))
(posn-x p) ;; find the defn for _val1 and fetch the value
(set-posn-x! p v) ;; find (define _val1 v1) and replace it with (define _val1 v)
```

Generalized to any mutable structs, mcons

Consider

```scheme
(define lst1 (cons (box 1) empty))
(define lst2 (cons 2 lst1))
(define lst3 (cons 3 lst1))

(set-box! (first (rest lst2)) 4)
(unbox (first (rest lst3))) ;; return 4

;; lst2 and lst3 shares the same tail
;; (first (rest lst2)) and (first (rest lst3)) refers to the same object
```

### Rethinking define

```scheme
(define x 3)
(set! x 7)
```

x is not just a value, its something we can mutate. x must denote a location and it's the location that contains the value.

In C, consider:

```C
int main() {
  int x = 1;
  int* y = &x; // y stores the address of x
  int* z = y; // z stores y which is the address of x
  *y = 2; // take the value of y and treat it as an address
  *z = 3; // take the value of z and treat it as an address
  printf("%d, %d, %d", x, y, z); // prints 3, 3, 3
}
```

In this case, *y, *z are aliases of x.

It can be subtle, or seemingly unintuitive

```C
void f(int *x, int *y) {
  *y = *x + 1;
  if (*x == *y) {
    printf("Hello!\n");
  }
}

int main() {
  int z = 1;
  f(&z, &z);
}

// This would print "Hello!"
```

### Vector (Racket)

- used much like the traditional array
- unlike out arrays, they can store data of any size
- length of memory is chosen by the program
- width is any value (can be infinitely large)

```scheme
(define x (vector 'blue true "you")) ; 3 item vector
(define u (make-vector 100)) ; create a vector of length 100
(define z (build-vector 100 sqr)) ; similar to build-list
(define y (make-vector 100 5)) ; vector of length 100 with value 5
(vector-ref x 7); value of x at index 7
(vector-set! dy 7 42); set the value of x at index 7 to 42
```

Main advantages of vectors over lists

- vector-ref and vector-set! take O(1)

Disadvantages

- size is fixed
- difficult to add or remove elements
- vector-set! tends to force an imperative style

```scheme
(define (my-build-vector n f)
  (define res (make-vector n))
  (define (mbv-h i)
    (if (= i n)
        res
        (begin
          (vector-set! res i (f i))
          (mbv-h (+ i 1)))))
  (mbv-h 0))
```

Vectors lend themselves well to an imperative style. for, for/vector macros in Racket allow this.

```scheme
(define (my-build-vector n)
  (define res (make-vector n))
  (for ([i n]) ;; for macro
    (vector-set! res i (f i))
  res))

(define (my-build-vector n f)
  (for/vector ([i n]) (f i))) ; for/vector macro```
```

Sum the values in a vector

```scheme
(define (sum-vector n)
  (define acc 0)
  (for ([i (vector-length n)] (set! acc (+ acc (vector-ref n i))))))
```

This is not prue functional

- use mutatoin
- to the outside observer, this for appears to be pure functional
- mutation can't be called detected outside of the function

Provides a good method for managing the world with mutation: encapsulate it in a pure functional interface

#### Memory Allocation

Memory slots only hold fixed sizes, so how is racket storing in its simulation of arrays things like unbounded integer and strings.

Recall:

```scheme
(define (mutate-posn p)
  (set-posn-x! p (+ 1 (posn-x p))))

(define p (posn 3 4))
(mutate-posn p)
(posn-x p) => 4
```

VS

```C
void mutate(struct Posn p) {
  p.x = p.x + 1;
}

int main() {
  struct Posn p = {3 ,4};
  mutate(p);
  printf("%d\n", p.x); // 3
}
```

Racket structs are not like C structs

- The struct is copied, but changes to the fields persisted
- The fields of a racket struct are boxed, they are just pointers

Similarly, the items in a racket vector are addresses that point at the actual contents (which can be of any size)

Also, since Racket is dynamically typed, there must be some way the interpreter figures out the types of these things. So this must also include type information. We'll consider that later.

### Array (C)

- primitive data structure
- a slice of memory
- a sequence of consective locations

```C
int main() {
  int grades[10]; // array of 10 ints
  for (int i = 0; i < 10; ++i) {
    scanf("%d", &grades[i]);
  }
  int acc = 0;
  for (int = 0; i < 10; ++i) {
    acc += grades[i];
  }
  prinf("%d", acc);
}
```

- `a[i]` access the i-th element of the array
- int grades[10];
  - valid indices are 0~9
- What happens if you access outside the boundary? Will it stop you?
  - Program may or may not crash
  - if not, data may be corrupted (no way to know)

Can you give the size implicitly?

```C
int main() {
  int grades[] = {0, 0, 0, 0};
  // amount of memory grades occupies is 16 bytes, 4 each for 4 ints
  printf("%zd\n", sizeof(grades) / sizeof(int)) // prints 4
}
```

Functions on arrays

```C
int sum(int arr[], int size) {
  int res = 0;
  for (int i = 0; i < size; ++1) {
    res += arr[i];
  }
  return res;
}

int main() {
  int myArr[100];
  int total = sum(myArr, 100); // myArray is shorthand for &myArray
}
```

Passing an entire array into a function would be very expansive. So passing arrays by value, C does not copy them.

Most confusing rule of all in C: The name of an arry is shorthand for a pointer to its first element.

myArray is shorthand for &myArray

Therefore, `sumArray(myArray, 100)`, passes a pointer, not the whole array into the fn.

But sum was told to expect an array not a pointer! Why not `sumArray(int *arr, int size)`.

In fact, `int *arr` and `int arr[]` are identical when it comes to parameter declarations.

```C
int sumArray(int *arr, int size) {
  int res = 0;
  for (int i = 0; i < size; ++i) {
    res += arr[i];
  }
  return res;
}
```

### Pointer Arithmetic

Let T be a type

```C
T arr[10];
sizeof(arr) == 10 * typeof(T)
```

- `arr` is shorthand for `&arr[0]`
- `*arr` is shorthand for `arr[0]`

Expression produces a pointer to `arr[1]`:  `&arr[1] == arr + 1`
Expression produces a pointer to `arr[2]`:  `&arr[2] == arr + 2`

Numerically, `arr + n` produces a pointer whos value is `arr + sizeof(T) * n`

If `arr + k` is shorthand for `&arr[k]`, then `arr[k] == *(arr + k)`

Therefore, sumArray is equivalent to

```C
int sumArray(int *arr, int size) {
  int res = 0;
  for (int i = 0; i < size; ++i) {
    res += *(arr + i);
  }
}

//or
int sumArray(int *arr, int size) {
  int res = 0;
  for (int *cur = arr; cur < arr + size; ++cur) {
    res += *cur;
  }
}
```

```C
int arr[] = {0, 1, 2, 3, 4};
printf("%d\n", 2[arr]); // &(2 + arr) same as arr[2]
```

Any pointer can be thought of as pointing at the beginning of an array. Same syntax for referencing elements in array through a ptr or array.

So are arrays and ptrs the same thing? NO!

```C
int f(int arr[]) {
  printf("%zd\n", sizeof(arr));
}

int main() {
  int arr[10];
  f(arr); // prints 8 (size of a pointer)
  printf("%zd\n", sizeof(arr));  // prints 40 (size of the array)
}
```

Compiler:

- `myArray[i]` (in main)
  - fetch myArray location in the environment
  - add i * sizeof(int)
  - fetch the value from our store (RAM)

- `arr[i]` (parameter of the function)
  - look up arr's location in the environment
  - fetch arr's value from the store (myArray's address)
  - add i * sizeof(int)
  - fetch the value from our store (RAM)

`myArray[i]` and `arr[i]` look the same but do slightly different things.

We saw that a Racket struct `(struct posn (x y))` is like  a C strut whose fields are pointers. How can we achieve this in C?

```C
struct Posn {
  int *x;
  int *y;
}

int main() {
  struct Posn p;
  *p.x = 3;
  *p.y = 4;
  // p.x and p.y are not initialized. So they point at arbitrary locations. Ideally, this crashes.  
}
```

So when we define p with `struct Posn p`. spaces are reserved for out pointers x and y, but we didn't reserve any space for the ints they should point at.

So, we must do so

```C
int main() {
 int a; // no good!
 int b; // no good!
 struct Posn p = {&a, &b};
 *p.x = 3;
 *p.y = 4;
}

// why not make a function like Racket make-posn
struct Posn makePosn(int x, int y) {
  int a = x;
  int b = y;
  struct Posn p = {&a, &b};
  return p;
}

int main() {
 struct Posn p = makePosn(3, 4);
 printf("%d\n", *p.x, *p.y); // *p.x and *p.y are dangling pointers
}
```

Each function gets its own piece of memory, specifically, it gets a piece of memory called the stack - we call each function piece a stack frame. After a function is done its work - it no longer needs space, so its space is reclaimed. The variable `a` and `b` no longer exist, but their value remains on that memory address until other operations claims that address and mutate the value.

This is called a **Dangling Pointer**: a pointer that points at memories that has been reclaimed

We know that Racket achieve this. It allocates space for the posn (for the 2 pointers). It also allocates space for the 2 things they point at. We need to do the same in C.

```C
#include <stdlib.h>

struct Posn {
  int* x;
  int* y;
}

struct Posn makePosn(int x, int y) {
  Struct posn ret;
  ret.x = malloc(sizeof(int));
  ret.y = malloc(sizeof(int));
  *ret.x = x;
  *ret.y = y;
  return ret;
}
```

## (Closer to) Real Memory Layout

Applies to C and racket

RAM

- Code
- Static
- Heap
- Stack (at the bottom)

Static area

- where global/static vars are stored
- its lifetime is the entire lifetime of the program

Stack

- an ADT with Last In First Out(LIFO)
- can only ever remove the last thing put in
  - push - an item onto the top of a stack
  - pop - an item off the top of a stack
  - top - peek at the top element of the stack
  - empty - is the stack empty

Racket lists are stacks!

- push = cons
- pop = cdr
- top = car
- empty = empty?

### Stack

Programs Stack stores local variables

Example:

```C
int fact(int n) {
  int res = 0;
  if (n == 0) return 1;
  res = fact(n-1);
  return n * res;
}
```

Each function calls gets a stack frame

- local variables are pushed onto the stack
- also return addresses - where the function goes when it returns
- each invocation of a function gets its own stack frame

When a function returns its stack frame is popped

- all local variables are released
- not typically erased
- "top-of-stack" pointer is moved to top of next frame

```C
int main() {
  struct Posn p = makePosn(3, 4);
  // other operations
  printf("%d %d", p.x, p.y); // does not print "3 4"
}
```

We solved this problem with `malloc`

- `malloc(n)` gives us a pointer to a cointiguous block of memory in the heap of n bytes
- heap has arbitrarily long lifetime (length of the program by default)
- if heap allocated data never goes away, eventually tour program will run out of memory. Even if you are not using most of the data anymore.

We know Racket's `make-posn` does the same thing. Racket's solution is a runtime process called a garbage collector that detects data that's no longer in use and releases it.

e.g.

```scheme
(define (f x)
  (local [(define p (posn 3 4))] ;; not needed after f returns
  ......  
))
;; automatically released
```

C solution: Heap memory is freed when you say so.

```C
int *p = malloc(...);
// operations ...
free(p);
```

Failing to free all allocated memory is called a memory leak

Programs that leak memory will crash if run long enough.

Consider

```C
int *p = malloc(sizeof(int));
free(p);
*p = 7; //will this crash?
```

Probably not:

- `free(p)` does not change p
- p still points at that memory
- but p is not pointing to a valid location - that location may be assigned again to another pointer by another malloc call.
- called a dangling pointer - BAD

Better Solution: After `free(p)`, assign it to a guaranteed non-valid location (force dereferencing p to crash)

```C
int *p = malloc(sizeof(int));
free(p);
p = NULL;
```

NULL

- not really part of the C language
- defined as a constant equal to `0`
- could equally say `p = 0`;

Dereferencing NULL

- undefined behaviou
- program may crash

if malloc failes to allocate memory, it returns NULL

Consider

```C
int* f() {
  int x = 4;
  return &x;
}

int g() {
  int y = 5;
  return y;
}

int main() {
  int* p = f(); // dangling pointers
  g();
  printf("%d\n", *p);
}
```

***NEVER*** return a pointer to a local variable

If you want to return a pointer it should be to static, heap allocated, or non-local stack data.

```C
int* pickOne(int* x, int* y) {
  return ____ ? x : y;
}

int main() {
  int *p = malloc(sizeof(int));
  int x = 5;
  *p = 10;
  int* g = &x;
  int* z = pickOne(p, g);
}
```

When to use a heap:

- for data that should outlive the fn that creates it
- for data whose size is not know at compile time
- for large local arrays

```C
int numSlotsNeeded;
scanf("%d", &numSlotsNeeded);
int* p = malloc(sizeof(int) * numSlotsNeeded); // an array on the heap of size numSlotsNeeded
// can access p[0] ... p[numSlotsNeeded-1]
// dynamic array (heap allocated)
...
free(p)
```

Programs usually have more heap memory available than stack memory

```C
int recursiveFn(int n) {
  int tempArray[10000]; // eats up our stack memory
  // use heap array instead
  ...
  recursiveFn(n-1);
}
```

C array mimic Racket vectors. Can we get the behaviour of racket lists?

`(cons x y)` => produces a pair

Racket is dynamically typed - list items can have different types - stored as pointers in the list

C is statically typed - list items would need to have the same type - so no real need for the values to be pointers

```C
// linked list
struct Node {
  int data;
  struct Node* next;
}

strcut Node* cons(int data, struct Node* lst) {
  struct Node* result = malloc(sizeof(Node));
  result->data = data;
  result->next = lst;
  return result;
}

int length(struct Node* lst) {
  if (!lst) return 0;
  return 1 + length(lst->next);
}

// f is a function pointer to a function that takes one int parameter and returns a int
struct Node* map(int (*f)(int), struct Node* lst) {
  if (!lst->next) {
    return NULL;
  }
  return cons(f(lst->data), map(f, lst->next));
}

int main() {
  struct Node* l = cons(4, cons(3, cons(2, 0)));

  // WRONG! Only frees the first node
  free(l);

  //Correct Version
  for (struct Node* p = l; p != NULL) {
    struct Node* temp = p->next;
    free(p);
    p = temp;
  }

  //Or recursively
  freeList(l);
}

void freeList(struct Node* lst) {
  if (lst == NULL) {
    return;
  }
  freeList(lst->next);
  free(lst);
}
```
