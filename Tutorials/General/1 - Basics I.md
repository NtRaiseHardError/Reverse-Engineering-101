# Basics I

## Introduction

Reverse engineering has a pretty high skill floor and as a result of that, the entry point can be quite rough for beginners. The sight of assembly can be quite intimidating and kill motivation really quickly. So to start off this journey of a mile, we'll start by taking the first step by familiarising something that is easy on the mind and abstract enough to understand but as low level as we can get. Yes, I'm talking about the lovely C language. In juxtaposition, we will compile the source into assembly and compare the two side by side.

## Example 1

Consider the following code:

```c
int main(void) {
    return 0;
}
```

Simple and straight-forward - it returns 0. Let's compile it and see what it looks like as assembly:

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

push    ebp
mov     ebp, esp
mov     eax, 0
pop     ebp
retn

main endp
```

This is the output given by IDA. Because I've compiled with symbols, IDA can recognise that this is the `main` function and has shown the definition `int __cdecl main(int argc, const char **argv, const char **envp)`. But let's not get distracted by the surrounding information for now and instead focus on the generated assembly...

First of all, reading Intel assembly can be confusing if you are not used to it. The way you read it is `<instruction> <dest> <src>`. Keep that in mind! These three-letter "words", `ebp`, `esp`, `eax`, are called "registers". Imagine them as sort of like variables as they will hold values/data.

It is fundamental to know that return values of functions are done via the `eax` register. If a function has a return value, i.e. doesn't have a `void` return type, it will give the return value to `eax` before returning. Here, our code was told to `return 0` and so, in the generated assembly, we can see `mov eax, 0` where `eax` will be given the value 0. Just a heads up, the `mov` instruction may be "move" but it is actually a **copy**. Remember that! So the `mov ebp, esp` instruction means that it is **copying** the value from the `esp` register into the `ebp` register. 

At the end, we can see the `retn` (or sometimes just `ret`) instruction, telling the function to return.

Easy, right? Well, we haven't gone through what the other instructions are, but we will ignore them for the time being. Meanwhile, let's look at one more example.

## Example 2

Consider the following code:

```c
int main(void) {
    return 42;
}
```

Here is the generated assembly in IDA:

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

push    ebp
mov     ebp, esp
mov     eax, 42
pop     ebp
retn

main endp
```

We can see that this is _almost_ exactly the same as the code above with the exception of one number: `42`. Now that we're returning 42 instead, it is reflected in the assembly of `mov eax, 42`. And indeed, the return value shall be 42.

# Summary

Two things to take away from this is:

* The `eax` register holds the return value _if_ the function has a defined return type (not `void`),
* The `mov` instruction is a **copy** meaning that the source will retain the value after executing the instruction.
