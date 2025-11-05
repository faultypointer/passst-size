+++
title = "Into the x86_64 land"
date = 2025-11-05
draft = true
[taxonomies]
tags = ["x86_64", "assembly"]
+++
We are going to learn assembly programming, more specifically x86_64 assembly. Why? Because that's the architecture my machine uses. As to Why
learn assembly in the fisrt place, I have no answer to that. So let's get right into it.

## But First

We need (imagine a drumroll here that is ..................... this long)*the setup*. We need assembler that can convert the assembly source into
machine code that can be executed. There are many assembler in the market: [nasm](https://www.nasm.us/), [gas](https://en.wikipedia.org/wiki/GNU_Assembler) and
many others. I've decided to go with nasm because it uses intel syntax which I like over the AT&T syntax.

We also probably need a debugger for which we have [gdb](https://sourceware.org/gdb/).

That is it for the tools we are going to need (except for things like editor and lsp if you want and other obvious things).

Finally lets start writing some assembly code.

## After this quick lesson on the basics

We need to have a general understanding of the x86_64 architecture before we write our first assembly code.

### Registers

I assume you know what registers are. If not tough luck.

There are 16 general purpose registers (GPR) in x86_64: RAX, RBX, RCX, RDX, RDI, RSI, RBP, RSP, R8-R15. You can use the lower 32-bit of these registers when working with 32-bit values.
In such case, you can use specify them using: EBX, ECX, EDX, EDI, ESI, EBP, ESP, R8D-R15D. There are also 16-bit and 8-bit variants with naming AX, BX, ... R8W-R15W for 16-bit and
AL, BL, ... R8B-R15B for 8-bit ones.
*The D here mean doubleword, W means word and B means byte.*

There are other registers as well for example flag registers, simd registers but we will not be taking about them now.

### Calling Convention

I am not going to explain about calling conventions now because I expect you to know what they are [*and its totally not because I am not confident about my explanation*]. All I'm goind to say is
that calling convention are a set of rules followed by programs mostly about where to put arguments to a function. The convention vary based not only on architecture and os but also based on
compilers as well. The calling convention we will follow is the one used by many c compilers for my system described [here](https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf#section.3.2).

- the parameters are passed in registers rdi, rsi, rdx, rcx, r8, r9. floating point parameters in xmm0 to xmm7
- additional parameters that don't fit in those registers are passed in stack
- the registers rbx, rsp, rbp, r12-r15 are callee-saved, while the rest of the registers are caller-saved
- so if a function wants to call another function and it has some value in caller preserved registers it wants after the function call, it should save those value before calling another function
- all floating point registers are caller preserved
- return values are stores in rax (or rdx:rax when necessary) or xmm0 (or xmm1:xmm0) for floating points
- if the return type doesn't fit in rdx:rax, the caller must pass pointer to the storage to where the return value should be placed in register rdi. the callee must return the same poiner in rax

```c
int threesum(int a, int b, int c) {
  return a + b + c;
}

float very_imp_function(int a, int b, float c, int d) {
  return a + b + d + (float)c;
}
```

```asm
0000000000000000 <threesum>:
   0: add    edi,esi
   2: lea    eax,[rdi+rdx*1]
   5: ret

0000000000000006 <very_imp_function>:
   6: movaps xmm1,xmm0
   9: add    edi,esi
   b: add    edi,edx
   d: pxor   xmm0,xmm0
  11: cvtsi2ss xmm0,edi
  15: addss  xmm0,xmm1
  19: ret
```

## I think it is time

We are gonna write hello world in assembly. We still don't have complete knowledge about how to print "Hello World" in the terminal using x86_64 assembly. But we are gonna figure those
things out as we go.

First of all, we need a place to write the assmebly code. Since we are using nasm, create a file called with `.nasm` extention [*which isn't necessary but cmon*].

Like c or any other compiled language, if we are going to make out program an executable, we need to provide it with an entry point. The linked we will later use (`ld`) to convert out
object file to an executable expects the entry point to be `_start` by default. So lets define a _start entry point in out program.

```asm
_start:
  ; I don't know any instruction yet!!
```

Now how can we write things to our terminal. If we were using c, we would probably call `printf` and pass the string "Hello, World" to that function. But we are doing assembly so we can't use printf. Or can we?
We absolutely can, but that is a topic for future part. Today we are getting a small taste of syscalls. In linux, there is a syscall for writing to a file descriptor called `write`. You can read about it in details using man pages: `man 2 write`.
It has the following defination:

```c
ssize_t write(int fd, const void buf[count], size_t count);
```

So it expects a file descriptor as its first argument, buffer from which to write to the file as second argument, and the total bytes from buffer to write. It returns the number of bytes
actually written on success (which may be less than the `count` passed as argument). On failure, it returns -1;

### But how do we actuall invoke the system call

The instruction to invoke a syscall in assembly is rightly named `syscall` which takes no operands. There isn't a single system call so how does one let the os know which system call I
want to invoke is? [*BTW if you are wondering how many syscall are there in linux for x86_64 architecture, the answer is too many.*]

This is basically the steps required to invoke a syscall using assembly:

- mov the syscall number of the syscall you want to invoke in register `rax`
- mov the arguments the syscall expects in appropriate registers
- use the `syscall` instruction to invoke the syscall
- the return value from syscall will be in register `rax`

With this we are ready to print the hello world message to the terminal. First of all, we need a file descriptor. Before you ask what a file descriptor is, [here](https://en.wikipedia.org/wiki/File_descriptor) is the wikipedia
article. The short[*and probably incorrect*] version is that it is a unique integer identifying an open file. We can use the standard output (stdout) to write to the terminal. Its file
descriptor is 1.

```asm
_start:
  mov rax, 0x1; the syscall number of write is 1
  mov rdi, 0x1; move the file descriptor of stdout (1) to rdi
  mov rsi, buffer containg the hello world message
  mov rdx, how many bytes from the buffer we want to be written
  syscall
```

Now we need a buffer storing the message (Hello World) that we want to print. To do that we need to learn a little bit about nasm and its program structure. The program is divided into sections. The `.text` section contains the code, the `.data` section
contains the initialized data. There are also `.bss` and `.rodata` section for allocating uninitialized space and for defining constant data respectively.

You can use [pseudo instructions](https://nasm.us/docs/3.01/nasm03.html#section-3.2) to declate initialized and uninitialized data.

With this, we are one step closer to printing hello world.

```asm
section .text
_start:
  mov rax, 0x1; the syscall number of write is 1
  mov rdi, 0x1; move the file descriptor of stdout (1) to rdi
  lea rsi, message
  mov rdx, length


section .data
  message: db "Hello, World", 10
  length: equ $ - message
```

A couple of things to note here. If you followed the link to pseudo instruction and read, you would know that `db` is used to define bytes. It accepts string constants too. The `10` is for the newline character.

The `equ` defines a symbol (in this case `length`) to a given constant value. The constant value is the value obtained after the evaluation of expression `$ - message`. The `$` is assembly position at the beginning of the line containing `$` from which the position of `message` is subtracted to get the length of the message.

If you were reading extra carefully, you might have noticed that an instruction changed in the above program. The `mov rsi, message` changed to `lea rsi, message`.

For this program, both instruction does the same thing but they are very different. We will get to those two and other instructions in a bit, but first lets assemble and link our program.

To compile the program with nasm, we can run the following command:

```bash
nasm -f elf64 hello.nasm
```

This creates an object file called `hello.o` which we can link using the linker (`ld  hello.o`) to make it into an executable.

No need to worry. It is just a warning saying it can't find the entry point `_start` (which is default for `ld`). You may be thinking, "What is `ld` smoking and can I get some too? I have defined the `_start` right there in the program". We still need to export it so that `ld` can see the thing. That is the work of the `global` directive.

We have finally completed the hello world in assembly. Lets us also exit from the program using `exit` syscall because why not?

```asm
section .data
  message: db "Hello World", 10
  len: equ $ - message

section .text

_start:
  mov rax, 0x1;
  mov rdi, 0x1; 
  lea rsi, message;
  mov rdx, len;
  syscall

  mov rax, 0x3c
  mov rdi, 0x1
  syscall
```

You might think that the syscall also use the same calling convention as the one we previously discussed, and you would be wrong. The registers used by the syscalls are: `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9`. You can find the syscall with their numbers and the arguments they expect in [this website](https://x64.syscall.sh/).

Phew! That was something but we got through and printed out hello world message in the terminal. Now its time to say bye until the next part is what I would say if I didn't think the next part would be out in atleast a month or two if ever. But I know myself better than that. So as a parting gift until next time [*if there is a next time*], I wanna leave with something more than just a hello world.

We haven't learned about SIMD registers or looping or anything like that so there isn't a lot we can do here. What we are gonna make is something very simple. We are making a cryptographically secure random number generator. Or atleast a bootleg version of it.

## Lets get familiar with some instructions first

We have used three instruction till now: `lea`, `mov` and `syscall`. We also have seen some more but lets not talk about them right now.

The `mov` instruction moves(copies) the value from source to destination. For example: `mov rax, rdx` moves the value from register `rdx` to `rax`. This is called register addressing.
As we have already seen, we can also use an immediate value as a source. This is called immediate addressing.
The one last addressing we need to worry about is the memory addressing. With it, we can use memory location as source or destination operand.
The general equation for addressing memory is: \[ base + index * scale + displacement \]. Here, the base and index can be any general purpose register, index can be one of \[1, 2, 4, 8\] and displacement can be any 8-bit, 16-bit or 32-bit value.

Let us assume we have an integer(32-bit) array. The address of the array(as well as that of first element) is stored in register rax.
To move the first element of array into register rdx, we can do the following: `mov rdx, [rax]`.

If we want to move the third element into `r8` register: `mov r8, [rax + 8]`. Since the address is byte addressable and our integer elements are each 4 bytes in size, we have to add 4 to get the next element.
What if we want to increment the $i^{th}$ element by 4? No worries.

```asm
mov rbx, 0x5; lets assume i is 5 for this instance
add [rax + rbx * 4], 0x4
```

Yes, we can use memory address as destination in other instruction as well. Speaking of which, `add` adds the value of source and destination and stores the result in destination.

What do you think `lea rsi, [rax + rbx * 4]` does? The `rbx` is still 5.
It moves the **adddress** of the $5^{th}$ element of the array into register `rsi`. `lea` computes the effective address of source operand and stores it into the destination operand.

Lets quickly go over the basic and simple instructions. We have already seen the `add` instruction. There is a `sub` instruction for substraction.
`and`, `or`, `nor`, `xor` performs bitwise and, or, not or xor of two operands respectively and stores the result in destination operand.

The `cmp` instruction compares two operands and changes the flag as if the source operands was subtracted from the destination operand.
Speaking of

### Flags

The RFLAGS register is a 64 register with its upper 32 bits reserved. The lower 32 bits contain groups of status, control and system flags.
The only flags we need to care about for now [*well we don't really need to care about any in this part*] are the following:

- Carry Flag: set if arithmetic operation generates a carry or a borrow out of MSB, cleared otherwise
- Zero Flag: set if the result is zero, cleared otherwise
- Signed Flag: equal to the MSB of result

The condition jump instruction uses those flags to jump to speficied location based on some condition.
There are different kinds of conditional jumps and they are ofter used with `cmp` to jump to an instruction in some location based on
the ordering of two values.
For example, `JE`: jump if zero flag is equal to zero. (so when used after `cmp` if two operands were equal)
You can check out all the conditional jump instructions [here](https://www.felixcloutier.com/x86/jcc).

There is also an unconditional jump(`jmp`) that just jumps to the instruction at the location.

## Cryptographically Secure Random Number Generator

So, how are we gonna be doing this? If you expected some actual algorithms like [cha cha slide](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant) or xoshiro(<https://en.wikipedia.org/wiki/Xorshift>) or any other then be ready to be disappointed.
Or you can take this as an opportunity and implement those algorithm in assembly yourself. For now, we are gonna do things simple.
What we will do is read few bytes from "/dev/urandom". We also don't have anything that can use that random number so we will just print it.

Let me outline what exactly what we will do. Maybe you can try to do it yourself before looking at the code.

- we need to open the file "/dev/urandom" using the `open` syscall
- use the fd returned by `open` to read the bytes to some memory location
- write the random (probably with some accompaning text) to standard out

About that last point, we could write a simple function to convert the integer to ascii so that write can print it out but again we haven't touched loops yet so we lets opt out for something simpler.

Remember when I said `printf` was a topic for the future part. Well, I lied. We are going to use it to print the random number.
So How do we call printf? What we can do is tell nasm "hey, I know I havent defined this `printf` thing but just trust me it exists". The `extern` directive is used to do that.
Then we can pass the library that actually defines the printf to `ld` when linking and we are done.

Here is the full code in all of its glory.

```asm
global _start
extern printf

section .data
  filename: db "/dev/urandom", 0
  message: db "Random Number: %u", 10, 0
  open_failure: db "Failed to open /dev/urandom", 10
  open_failure_len: equ $ - open_failure
  read_failure: db "Failed to read from file", 10
  read_failure_len: equ $ - read_failure

section .bss
  random_number: resd 1

section .text 

_start:
  mov rax, 0x2
  lea rdi, filename 
  mov rsi, 0x0; O_RDONLY
  syscall

  cmp rax, 0x0; 
  jl failed_to_open

  mov rdi, rax
  mov rax, 0x0
  lea rsi, random_number
  mov rdx, 0x4
  syscall

  cmp rax, 0x4
  jne failed_to_read

  lea rdi, message
  mov esi, [random_number]
  xor rax, rax
  call printf

  mov rax, 0x3c
  mov rdi, 0x0
  syscall



failed_to_open:
  lea rdi, open_failure
  mov rdx, open_failure_len
  jmp exit_with_error

failed_to_read:
  lea rdi, read_failure
  mov rdx, read_failure_len
  jmp exit_with_error


exit_with_error:
  mov r8, rdi
  mov rax, 0x1
  mov rdi, 0x1
  mov rsi, r8
  ; rdx should be set by whatever calls this
  syscall 

  mov rax, 0x3c
  mov rdi, 0x1
  syscall
```

The only thing I should mention is that the `0` at the end of `message` is because the format string `printf` expects is a null (0) terminated string.

We will discuss about `call` instruction as well as `ret` and other things related to it in a future part, but for now you can see that
`printf` follows the calling convention. We will also discuss about other aspects of calling convention like aligning stack in the future.

With that, I'm off.
