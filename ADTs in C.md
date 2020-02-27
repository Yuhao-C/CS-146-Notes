# ADTs in C

- [ADTs in C](#adts-in-c)
  - [Sequence ADT](#sequence-adt)

---

## Sequence ADT

Operations:

- `empty-sequence`
- `insert(s, i, e)` insert e into s at index i
- `size(s)` number of elements in s
- `remove(s, i)` remove item from s at index i
- `index(s, i)` produce the value of the element at index i in s

Linked List:

- +: easy to grow
- -: slow to inde

Array:

- +: fast indexing
- -: fixed size, hard to grow

We need fast indexing.

Approach - partially filled heap array

```C
// file: Struct.h
struct Sequence {
  int size; // how many elements
  int *theArray;
  int cap;// capacity of the array
  struct Sequence emptySeq();
  int seqSize(struct Sequence s);
  void add(struct Sequence* s, int i, int e);
  void remove(struct Sequence* s, int i);
  int index(struct Sequence s, int i);
  void freeSeq(struct Sequence* s);
};
```
