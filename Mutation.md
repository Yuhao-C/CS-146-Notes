# Mutation

## Basic Mutation in Racket: `set!`

```scheme
(define x 3)
(set! x 4)
x
=> 4
```

## Example

```scheme
(lookup 'Rob)
=> false
(add 'Rob 3)
(lookup 'Rob)
=> 3
```

## Implementation in impure Racket

```scheme
(define address-book empty) ;; global variable
(define (add name number)
(set! address-book
  (cons (list name number) address-book))
```

**Global data**: good for defining constants going to be used repeatedly.

* But not great with mutations
* Any part of the program could change global
* Affects the entire program
* Hidden dependencies between different parts of the program
* Harder to reason about programs

## Application

* **Caching**: Saving the result of computation to avoid repeating
* **Memoization**: maintaining a list or table of cached values

### Consider

```scheme
(define (fib n)
  (cond
    [(= n 0) 0]
    [(= n 1) 1]
    [else (+ (fib (- n 1))
             (fib (- n 2)))]))
```

It is inefficient becasue recursive calls are repeated doing the same computation:

* `(fib 98)` called twice, `(fib 97)` called three times
* `(fib n)` is $\theta$`(f n)`
* To avoid repetition, we can keep a table of computed value
