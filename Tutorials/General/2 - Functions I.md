# Introduction

In this tutorial, we'll be going over the basic structure of functions and the classic Hello World example. We'll be covering what was explained in the previous tutorial so make sure you've caught up with that.

## Example

Consider the following code:

```c
#include <stdio.h>

int main(void) {
    printf("Hello world!\n");
    
    return 0;
}
```

and its disassembly:

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

push    ebp
mov     ebp, esp
push    offset aHelloWorld ; "Hello world!\n"
call    _printf
add     esp, 4
mov     eax, 0
pop     ebp
retn

main endp
```

We can identify the `return 0` assembly right away that we learned in the previous tutorial as well as the other same instructions. The only difference here, in comparison with the previous tutorial's disassembly, is the `push offset aHelloWorld`, `call _printf` and `add esp, 4`. It's obvious from the `_printf` symbol and the `call` instruction that this is the `printf` function call from the source and by deduction, the `push offset aHelloWorld` is the parameter.

I'd like to point out that reverse engineering is not only the ability to read assembly but it's also the skill to be able to draw conclusions from given context. Recognising the differences between this and the previous tutorial's disassembly as well as making the connection from the source and its dissasembly to understand how the `printf` function and its argument work is important critical thinking.

Finally, there is also the `add esp, 4` which you may or may not have connected to the function call. Its purpose is to "undo" the `push offset aHelloWorld`. That's all that you need to know for now as it will be covered in a later tutorial.

The `aHelloWorld` symbol is actually an address that IDA has automatically analysed as a string and relabeled to something appropriate. It simply points to the start of the string as described by the definition of a (string/char) pointer in C as shown here:

```asm
.rodata:080484C0 aHelloWorld     db 'Hello world!',0Ah,0
```

The address exists within the `.rdata` or `.rodata` section/segment of the executable file. This section is "read-only" data so it cannot be written to. This is similar to that of a global variable (that cannot be modified).

## Decompilation

So let's attempt to decompile the dissassembly back into C. First, we know that this is a `main` function from the symbols shown by IDA.

```c
int main(int argc, char *argv[], char *envp[]) {

}
```

Notice that I have retained the same signature provided by IDA. Although this is not what we originally defined in our source, it is still technically valid and has the same effect since we do not utilise any of the function's arguments.

Next, we have the `return 0` so let's add that in.

```c
int main(int argc, char *argv[], char *envp[]) {
    return 0;
}
```

From above, we noticed that there is a call to the `printf` function and we also came to the conclusion that it had a parameter of `"Hello world!\n"` from analysis of the source. I also mentioned that the `aHelloWorld` symbol was similar to that of a (non-modifiable) global variable.

```c
char *string = "Hello world!\n";

int main(int argc, char *argv[], char *envp[]) {
    printf(string);

    return 0;
}
```

One last thing that we are missing, and was also omitted from the disassembly, is the `stdio.h` header! Since the compilation from C to assembly is lossy, we lose some information. But knowing that the `printf` function was used, we can make the safe assumption that the original source had the `stdio.h` header or something similar that also has that function.

This is our final decompilation:

```c
#include <stdio.h>

char *string = "Hello world!\n";

int main(int argc, char *argv[], char *envp[]) {
    printf(string);

    return 0;
}
```

And that's how it's done! The decompiled and original source are pretty much almost the same.

## Summary

Key takeaways from this tutorial:

* Function calls are performed by the `call` instruction,
* Function parameters are passed via the `push` instruction,
* "Undoing" the function parameter `push` is done with the `add esp, X` instruction,
* Read-only strings are stored in a global, read-only section of the executable file,
* We can rediscover lost information such as headers by inferring from the functions present in the disassembly.
