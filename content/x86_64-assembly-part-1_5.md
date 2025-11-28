+++
title = "Into the x86_64 land Part 1.5: Memory Layout of Struct"
date = 2025-11-28
[taxonomies]
tags = ["x86_64", "assembly", "into-x86_64-land"]
+++
## Alignment
A memory address `a` is said to be `n`-byte aligned if it is divisible by `n`. For example, write a c program like so:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    double *d = (double *)malloc(sizeof(double));
    printf("Address: %p\n", d);
    return 0;
}
```

If you compile and run this program, you will get an address like `0x561d3a642310`. This number is divisible by 1, 2, 4, 8 and 16 as well. It is not divisible by 32 tho. So the address is 16-byte aligned, 8 byte aligned, etc but not 32 byte aligned.

Based on this, there are two ways of accessing data from memory. 

An aligned data access is one where the data being accessed is n-byte in size located at the address `a` which is n-byte aligned (i.e `a` is divisible by `n`).

The other kind of access is Misaligned data accesses where n-byte data is accessed from address that is not n-byte aligned.

## Okay. Who cares?
Your compilers care and so does the cpu and so should you.

The processor doesn't access the memory in byte size, it access the memory in a fixed size chunks (like 2-byte, 4-byte, 8-byte, 32-byte or even 64-byte) called memory access granularity.

With an unaligned memory access, you can run into problems such as poor performance, program crashes, needing to restart your machine, memory corruption etc.
You can check [this article](https://developer.ibm.com/articles/pa-dalign/) for more detail. The article also describes the layout of a structure in memory so you don't even have to come back to this blog.

## Array
How are elements in array located in memory? Let's write a simple c program.
```c
#include <stdio.h>

int main() {
    double arr[] = {1.0, 2.0, 3.0, 4.0, 5.0};
    printf("Address of arr: %p\n", &arr);
    printf("Address of first element arr: %p\n", &arr[0]);
    printf("Address of second element arr: %p\n", &arr[1]);
    printf("Address of third element arr: %p\n", &arr[2]);
    printf("Address of forth element arr: %p\n", &arr[3]);
    printf("Address of last element arr: %p\n", &arr[4]);
    printf("Size of array: %ld\n", sizeof(arr));
    return 0;
}
```

The output: 
```bash
Address of arr: 0x7ffdeac77f50
Address of first element arr: 0x7ffdeac77f50
Address of second element arr: 0x7ffdeac77f58
Address of third element arr: 0x7ffdeac77f60
Address of forth element arr: 0x7ffdeac77f68
Address of last element arr: 0x7ffdeac77f70
Size of array: 40
```
The elements of the array are stored linearly, the next one starting right after the previous one ends. The elements are of type `double` and thus needs to be 8-byte aligned. The address of `arr`(and also the address of first element) is infact 8-bytes aligned. The address of the second element is the address of first element + the size of the first element (8), thus it is also 8-byte aligned. It is the case for all the elements of array.

So if the address of first element (that is starting address of array) is n-byte aligned (n being the size of an element of array), then every element of the array will be n-byte aligned since the address of every element of array is n-byte aligned address + k * n which is an n-byte aligned address. 

This works because the elements of array all have the same type and thus the same size.

Let's see what happens when its an struct with different element types. 

## Struct
Let's define two structs, a function to initialize the struct and a main function to print the address of elements of the struct.

```c
struct Position {
    int x, y;
};

struct MoreComplex {
    struct Position path[1];
    int num;
    double cost;
    bool is_first;
};

struct MoreComplex *struct_layout() {
    struct MoreComplex *mc =
        (struct MoreComplex *)malloc(sizeof(struct MoreComplex));
    mc->path[0].x = 3;
    mc->path[0].y = 5;
    mc->num = 1;
    mc->cost = 134.45;
    mc->is_first = false;
    return mc;
} 

int main() {
    struct MoreComplex *mc = struct_layout();
    printf("Address of mc: %p\n", mc);
    printf("Address of mc->path: %p\n", &mc->path);
    printf("Address of first element of mc->path: %p\n", &mc->path[0]);
    printf("Address of mc->path[0].x: %p\n", &mc->path[0].x);
    printf("Address of mc->path[0].y: %p\n", &mc->path[0].y);
    printf("Address of mc->num: %p\n", &mc->num);
    printf("Address of mc->cost: %p\n", &mc->cost);
    printf("Address of mc->is_first: %p\n", &mc->is_first);
    printf("Size of *mc: %lu\n", sizeof(*mc));
    return 0;
}
```

Let's go over the output first.
```bash
Address of mc: 0x55c07ebda310
Address of mc->path: 0x55c07ebda310
Address of first element of mc->path: 0x55c07ebda310
Address of mc->path[0].x: 0x55c07ebda310
Address of mc->path[0].y: 0x55c07ebda314
Address of mc->num: 0x55c07ebda318
Address of mc->cost: 0x55c07ebda320
Address of mc->is_first: 0x55c07ebda328
Size of *mc: 32
```

The first thing we see is that the address of the first field(path) is the same as the address of our struct instant(mc). So like array, struct also doesn't contain any metadata or header information. Because of that the address of field x of first `Position` in path field of `mc` is still the same as the address of `mc` even when its inside a struct which is inside an array which is itself a field of another struct.

Lets start with the `Position` struct. The address of its field `x`, an integer (4-byte aligned) is 0x55...310. The next field of `Position` is also an integer and its address is also 4-byte aligned as it starts right after the `x` (4-byte aligned address + 4 byte = 4-byte aligned address).

The next field of `mc` is `num`, an integer, and since it starts right after `mc->path` that is the last(and first and only in this case) element of `mc.path` that is `mc->path[0].y`, its address is also 4-byte aligned.

We finally get to something interesting. The next element is `mc->cost`, a double. The address it can be at is 0x55...318 + 4 (size of `mc->num`) = 0x55...31c. But it doesnt start at 0x55...31c it starts at 0x55...320. Why? Because the double is 8-bytes aligned but the address 0x55...31c is not. The next address after 0x55...31c that is 8-byte aligned is 0x55...320.

The final element `mc->is_first` starts where `mc->cost` ends, 0x55...328. Since its 1-byte aligned it can start there and it ends at 0x55...329. But it we look at the size of our struct, it ends at 0x55...310 + 0x20(32) = 0x55...330. There is no element after `is_first` that we need to align. So why do we need the extra 7 bytes at the end of struct.

Well according to System V ABI, we have the following rules for struct alignment
- Structures assume the alignment of their most strictly aligned component
- The size of any object is always a multiple of the object‘s alignment

Our struct has an alignment of 8 bytes as that is the largest alignment of any of its element. So its size has to be a multiple of 8. 
0x55...329 - 0x55...310 = 0x19. It is not a multiple of 8. The smallest number larger than 0x19 that is a multiple of 8 is 0x20 or 32 in decimal. Thus we have 32 as the size of our struct.


## Packed Struct

We can use the gcc's `packed` attribute to for the compiler to remove any paddings. Doing this may make the fields of struct unaligned and thus making memory access slower. It maybe beneficial in case of 
embedded systems where size is more of an issue that speed.

In the above program add the `packed` attribute to `MoreComplex` struct like so:

```c
struct MoreComplex {
    struct Position path[1];
    int num;
    double cost;
    bool is_first;
} __attribute__((packed));
```

```bash
Address of mc: 0x55e0f88cd310
Address of mc->path: 0x55e0f88cd310
Address of first element of mc->path: 0x55e0f88cd310
Address of mc->path[0].x: 0x55e0f88cd310
Address of mc->path[0].y: 0x55e0f88cd314
Address of mc->int: 0x55e0f88cd318
Address of mc->cost: 0x55e0f88cd31c
Address of mc->is_first: 0x55e0f88cd324
Size of *mc: 21
```
As you can see, there are no paddings here.

If you don't want unaligned memory but still want to save space, you can rearrange the fields of your struct so that the field with highest alignment is first, then element with second highest alignment and so on.

For example, doing this with our `MoreComplex` struct, 
```c
struct MoreComplex {
    double cost;
    struct Position path[1];
    int num;
    bool is_first;
};
```

We can save 8 bytes of space without sacrificing the performance.

```bash
Address of mc: 0x55b02b517310
Address of mc->path: 0x55b02b517318
Address of first element of mc->path: 0x55b02b517318
Address of mc->path[0].x: 0x55b02b517318
Address of mc->path[0].y: 0x55b02b51731c
Address of mc->int: 0x55b02b517320
Address of mc->cost: 0x55b02b517310
Address of mc->is_first: 0x55b02b517324
Size of *mc: 24
```


With that, I'm off.
