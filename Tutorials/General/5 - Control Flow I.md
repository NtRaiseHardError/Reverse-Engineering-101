# Control Flow I

This tutorial will cover the `if` statements. It will discuss the different comparison operations as well as how the instruction pointer can be controlled to execute the appropriate instructions.

## Example I

Consider the following code:

```c
#include <stdio.h>

int main() {
    int a = 1;
    int b = 2;

    if (a < b) {
        puts("a < b");
    }
    
    if (a == b) {
        puts("a == b");
    }
    
    if (a > b) {
        puts("a > b");
    }

    return 0;
}
```

and its disassembly:

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

var_10= dword ptr -10h
var_C= dword ptr -0Ch
var_4= dword ptr -4
argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

lea     ecx, [esp+4]
and     esp, 0FFFFFFF0h
push    ebp
mov     ebp, esp
push    ecx
sub     esp, 14h
mov     [ebp+var_10], 1
mov     [ebp+var_C], 2
mov     eax, [ebp+var_10]
cmp     eax, [ebp+var_C]
jge     short loc_8048442
sub     esp, 0Ch
push    offset aAB      ; "a < b"
call    _puts
add     esp, 10h

loc_8048442:
mov     eax, [ebp+var_10]
cmp     eax, [ebp+var_C]
jnz     short loc_804845A
sub     esp, 0Ch
push    offset aAB_0    ; "a == b"
call    _puts
add     esp, 10h

loc_804845A:
mov     eax, [ebp+var_10]
cmp     eax, [ebp+var_C]
jle     short loc_8048472
sub     esp, 0Ch
push    offset aAB_1    ; "a > b"
call    _puts
add     esp, 10h

loc_8048472:
mov     eax, 0
mov     ecx, [ebp+var_4]
leave
retn

main endp
```

The disassembly shows the local variables as generic `var_X` labels. To figure out which one is `a` and which is `b`, we can simply follow thea assignments of the `1` and `2` values. Let's rename `var_10` to `a` and `var_C` to `b`.

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

a= dword ptr -10h
b= dword ptr -0Ch
var_4= dword ptr -4
argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

lea     ecx, [esp+4]
and     esp, 0FFFFFFF0h
push    ebp
mov     ebp, esp
push    ecx
sub     esp, 14h
mov     [ebp+a], 1
mov     [ebp+b], 2
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jge     short loc_8048442
sub     esp, 0Ch
push    offset aAB      ; "a < b"
call    _puts
add     esp, 10h

loc_8048442:
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jnz     short loc_804845A
sub     esp, 0Ch
push    offset aAB_0    ; "a == b"
call    _puts
add     esp, 10h

loc_804845A:
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jle     short loc_8048472
sub     esp, 0Ch
push    offset aAB_1    ; "a > b"
call    _puts
add     esp, 10h

loc_8048472:
mov     eax, 0
mov     ecx, [ebp+var_4]
leave
retn

main endp
```

Examining the structure of the disassembly, we see that there are some `loc_XXX`. These are labels that define addresses and they are primarily used with `jmp` instructions. These instructions are equivalent to the `goto` statement which simply changes the value of `eip`.

Let's analyse the instruction flow of this program.

```asm
mov     [ebp+a], 1
mov     [ebp+b], 2
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jge     short loc_8048442
sub     esp, 0Ch
push    offset aAB      ; "a < b"
call    _puts
add     esp, 10h

loc_8048442:
```

The first two instructions give our local variables their values `1` and `2`. The third instruction moves `a`'s variable into `eax`. The `cmp` instructions is then used to _compare_ `eax` with `b`. `cmp` works by subtracting the `src` operand from the `dest` operand but does **not** save the value. 

So why do a subtraction if the value isn't saved to the operands? The resulting effect is echoed into what's known as the _EFLAGS_. These flags indicate the status of certain operations, for example, if the result is zero (ZF), if there's a carry (CF), if there's a negative sign (SF). The `jcc` set of instructions (conditional `jmp`s) will either jump or not jump `eip` depending on whether certain flags are set or not.

Back to the `cmp` instruction, it affects multiple flags but the one we are concerned with are the OF and SF. The `jge` conditional jump will read these two flags and jump only if `SF = OF` (overflow flag). Essentially, if the subtraction becomes negative, `1 - 2 < 0` then the condition is **not** satisfied and therefore will **not** jump. And since the subtraction is indeed negative, the jump is **not** taken. This is how these instructions work. If this is too confusing then it may be much simpler to just compare it like: "is 1 greater than or equal to 2?" and of course, the answer is no, hence, the jump is **not** taken. If the `jge` is **not** taken, then it will run down into the `puts` function and print `a < b`.

Let's take a look at the next `if` statement:

```asm
loc_8048442:
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jnz     short loc_804845A
sub     esp, 0Ch
push    offset aAB_0    ; "a == b"
call    _puts
add     esp, 10h

loc_804845A:
```

Here, we assign `a` to `eax` again. Why? Because x86 Intel assembly does not support the `cmp [mem], [mem]` pattern so this must be done. Comparing `eax` to `b`, this time it uses the `jnz`. This jump will occur only if `ZF = 0`. This means that if `1 - 2 != 0`, the jump will happen. Of course, `1 - 2 < 0` as we previously answered so it is **not** zero. The `ZF` flag is **not** set and so `eip` will jump immediately to `loc_804845A`, skipping the `puts` function.

I will leave the final `if` statement for the reader to look through and repeat the above steps as it is quite trivial.

## Example 2

Consider the following code:

```c
#include <stdio.h>

int main() {
    unsigned int a = 1;
    unsigned int b = 2;

    if (a < b) {
        puts("a < b");
    }
    
    if (a == b) {
        puts("a == b");
    }
    
    if (a > b) {
        puts("a > b");
    }

    return 0;
}
```

and its disassembly:

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

a= dword ptr -10h
b= dword ptr -0Ch
var_4= dword ptr -4
argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

lea     ecx, [esp+4]
and     esp, 0FFFFFFF0h
push    dword ptr [ecx-4]
push    ebp
mov     ebp, esp
push    ecx
sub     esp, 14h
mov     [ebp+a], 1
mov     [ebp+b], 2
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jnb     short loc_8048442
sub     esp, 0Ch
push    offset aAB      ; "a < b"
call    _puts
add     esp, 10h

loc_8048442:
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jnz     short loc_804845A
sub     esp, 0Ch
push    offset aAB_0    ; "a == b"
call    _puts
add     esp, 10h

loc_804845A:
mov     eax, [ebp+a]
cmp     eax, [ebp+b]
jbe     short loc_8048472
sub     esp, 0Ch
push    offset aAB_1    ; "a > b"
call    _puts
add     esp, 10h

loc_8048472:
mov     eax, 0
mov     ecx, [ebp+var_4]
leave
lea     esp, [ecx-4]
retn

main endp
```

Examining this disassembly reveals some small differences. Some `jcc` instructions here are new. They are in fact the _unsigned_ instruction counterparts. With the _signed_ `jcc` instructions, they are usually named with "greater than" or "less than" whereas the _unsigned_ instructions are named "above" or "below" instead. Here we have `jnb` and `jbe` which are _jump not below_ and _jump below or equal_. Inversely, they can also be `jae` and `jna` respectively, which are _jump above or equal_ and _jump not above_. The _signed_ `jcc`s usually deal with `ZF`, `SF`, and `OF` flags whereas the _unsigned_ `jcc`s are controlled by `CF` and `ZF` flags. As you can imagine, the _unsigned_ comparisons no longer deal with negative results thus the lack of use of the `SF` and `OF` flags.

The disassembly is left for the reader to understand.
