# Stack II

In this tutorial, I'll be continuing from the previous _Stack I_ tutorial. We'll be going through how the stack operates with functions and its local variables, and how the `call` and `ret` instructions work.

## Example

Let's first look at an example with our own function. Consider the following code:

```c
int add(int a, int b) {
    int num1 = a;
    int num2 = b;
    
    return num1 + num2;
}

int main(void) {
    int sum = 0;
    
    sum = add(1, 2);
    
    return 0;
}
```

and its disassembly:

```asm
; int __cdecl add(int a, int b)
public add
add proc near

num1= dword ptr -8
num2= dword ptr -4
a= dword ptr  8
b= dword ptr  0Ch

push    ebp
mov     ebp, esp
sub     esp, 10h
mov     eax, [ebp+a]
mov     [ebp+num1], eax
mov     eax, [ebp+b]
mov     [ebp+num2], eax
mov     edx, [ebp+num1]
mov     eax, [ebp+num2]
add     eax, edx
leave
retn

add endp

; int __cdecl main(int argc, const char **argv, const char **envp)
public main
main proc near

sum= dword ptr -4
argc= dword ptr  8
argv= dword ptr  0Ch
envp= dword ptr  10h

push    ebp
mov     ebp, esp
sub     esp, 10h
mov     [ebp+sum], 0
push    2               ; b
push    1               ; a
call    add
add     esp, 8
mov     [ebp+sum], eax
mov     eax, 0
leave
retn

main endp
```

Let's start with the `main` function first and analyse its stack. Here it is before the first instruction:

```
Before push ebp

        ^  Towards top of the stack
        |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+  esp
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  ebp
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

The function then executes a `push ebp`:

```
push ebp

        ^  Towards top of the stack
        |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+  esp
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  ebp <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

and `mov ebp, esp`:

```
mov ebp, esp

        ^  Towards top of the stack
        |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+  esp = ebp
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

At this point, we are now setting up the stack frame for the `main` function. The next instruction is `sub esp, 10h`. Doing this grows the stack frame and now has a structure similar to that before executing the `mov ebp, esp` instruction.

```
sub esp, 10h

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      ...       |
+----------------+  ebp ----- Bottom of main's stack frame
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

Notice that the disassembly provided by IDA helpfully shows that the value for the `sum` symbol is `-4` (`sum= dword ptr -4`). This means that the instruction `mov [ebp+sum], 0` is equivalent to `mov [ebp-4], 0`. Where exactly is `[ebp-4]`? Well, it's in the area within `main`'s stack frame!

```
Local variable sum in main

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      sum       |  <-- local variable sum @ ebp-4
+----------------+  ebp ----- Bottom of main's stack frame
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

So when compiling the C code down to assembly, it has created this abstracted concept of the stack frame to store the local variable within the stack, specifically within the function's frame, and can now reference its location by using the `ebp` register as the base offset and then adding an offset to locate a specific variable, i.e. `-4` is added to `ebp` to reference the `sum` local variable. The `dword` adjective describes a 4-byte value, i.e. an `int` as we had defined in C. After this instruction is executed, the `sum` variable will be `0`.

```
mov sum, 0

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- mov sum, 0
+----------------+  ebp ----- Bottom of main's stack frame
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

The next two instructions are `push 2` and `push 1` where each `push` is a 4-byte `dword` value. These two are the arguments to the `add` function. If you've been paying attention, you may have noticed that this is actually the reverse order of how the `add`'s parameters were defined. I will explain the reason for this in a later tutorial so just keep that in mind for now.

```
push 2; push 1

        ^  Towards top of the stack
        |
+----------------+  esp
|     a = 1      |  <-- push 1 (argument 1)
+----------------+ 
|     b = 2      |  <-- push 2 (argument 2)
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

Now comes the `call add`. When a `call` instruction is executed, two things happen. The first is that it will reserve the value of the **next** instruction of `eip` in the stack, i.e. `add esp, 8`. The reason it does this is because it will need to keep track of where to continue from after leaving the `add` function. 

```asm
; ...

push    2               ; b
push    1               ; a
call    add             ; <-- move address of add into eip
add     esp, 8          ; <-- the address here is saved onto the stack

; ...
```

When the function returns with `ret`, it will pop the value from the current `esp` location into `eip`. Let's see the `call` in action with the stack:

```
call add

        ^  Towards top of the stack
        |
+----------------+
|      ...       |
+----------------+  esp
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1
+----------------+ 
|     b = 2      |  <-- argument 2
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame
|    Saved ebp   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

The second thing that happens is `eip` will be given the value of the `add` function. The `add` symbol is an address that describes the beginning of the `add` function. Let's move onto the `add` disassembly.

We see the same thing again with the 

```asm
push ebp
mov ebp, esp
sub esp, 10h
```

Let's do them in order:

```
push ebp

        ^  Towards top of the stack
        |
+----------------+  esp
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1
+----------------+ 
|     b = 2      |  <-- argument 2
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

Note that I've relabelled each saved `ebp` on the stack for clarity. Next instruction is `mov ebp, esp`:

```
mov ebp, esp

        ^  Towards top of the stack
        |
+----------------+  esp = ebp
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1
+----------------+ 
|     b = 2      |  <-- argument 2
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

and `sub esp, 10h`:

```
sub esp, 10h

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+  ebp ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1
+----------------+ 
|     b = 2      |  <-- argument 2
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

We've now created `add`'s stack frame just like `main`! And just like `main`, we have some local variables `num1` and `num2` at positions `ebp-8` and `ebp-4` respectively.

```
Local variables num1 and num2

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |  <-- num1 @ ebp-8
+----------------+ 
|      num2      |  <-- num2 @ ebp-4
+----------------+  ebp ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1
+----------------+ 
|     b = 2      |  <-- argument 2
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|     Value 1    |
+----------------+
|     Value 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

IDA also gives argument variables `a` and `b`, similar to `main` with its `argc`, `argv`, and `envp`. If you look at the stack and and count their positions (for both `add` and `main`), we can visualise and make sense of how it works out:

```
Function parameters

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |  <-- num1 @ ebp-8
+----------------+ 
|      num2      |  <-- num2 @ ebp-4
+----------------+  ebp ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1 @ add's ebp+8
+----------------+ 
|     b = 2      |  <-- argument 2 @ add's ebp+C
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|     Value 1    |
+----------------+
|      argc      |  <-- argc @ main's ebp+8
+----------------+
|      argv      |  <-- argv @ main's ebp+C
+----------------+
|      envp      |  <-- envp @ main's ebp+10
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

We can also deduce that `Value 1` on the stack must be the return value for `main`!

```
Saved eip before main

        ^  Towards top of the stack
        |
+----------------+  esp ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |  <-- num1 @ ebp-8
+----------------+ 
|      num2      |  <-- num2 @ ebp-4
+----------------+  ebp ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |  <-- argument 1 @ add's ebp+8
+----------------+ 
|     b = 2      |  <-- argument 2 @ add's ebp+C
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|   Saved eip    |  <-- Saved eip for main's ret
+----------------+
|      argc      |  <-- argc @ main's ebp+8
+----------------+
|      argv      |  <-- argv @ main's ebp+C
+----------------+
|      envp      |  <-- envp @ main's ebp+10
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

If we look at the stack view for `add`'s function in IDA, we can see the same layout:

```
-0000000000000010
-0000000000000010                 db ? ; undefined
-000000000000000F                 db ? ; undefined
-000000000000000E                 db ? ; undefined
-000000000000000D                 db ? ; undefined
-000000000000000C                 db ? ; undefined
-000000000000000B                 db ? ; undefined
-000000000000000A                 db ? ; undefined
-0000000000000009                 db ? ; undefined
-0000000000000008 num1            dd ?
-0000000000000004 num2            dd ?
+0000000000000000  s              db 4 dup(?)
+0000000000000004  r              db 4 dup(?)
+0000000000000008 a               dd ?
+000000000000000C b               dd ?
+0000000000000010
+0000000000000010 ; end of stack variables
```

where `r` is the saved `eip` value and `s` is the saved `ebp` value. The +/- inflection point is where `ebp` points and the top of the stack is where `esp` points. Note: `dd` represents "define `dword`" and `db` is "define `byte`", hence why it doesn't seem to align with what our stack looks like but if you count the bytes, it will turn out to be the same.

Now that we have this visual aid, you'll notice a pattern. The local variables are within the stack frame defined by `esp` and `ebp`. Below the stack frame lies, in order, the saved `ebp` value and then the saved `eip` value. After that, _if_ there are arguments to that function, they will exist there. Then, underneath that is the stack frame of the caller function.

Let's go through these instructions of `add`:

```asm
mov     eax, [ebp+a]
mov     [ebp+num1], eax
mov     eax, [ebp+b]
mov     [ebp+num2], eax
mov     edx, [ebp+num1]
mov     eax, [ebp+num2]
add     eax, edx
```

This is relatively straightforward. the first two instructions `mov eax, [ebp+a]` and `mov [ebp+num1], eax` simply transfer the parameter `a` into `eax` and then from `eax` into the local variable `num1`. The two instructions after that does the same thing but with the parameter `b` and local variable `num2`. Notice that when the next three instructions perform the addition of the two numbers, the result is stored within the `eax` register!

Finally, to conclude the `add` function, let's analyse the remaining instructions:

```asm
leave
retn
```

The first of these is `leave`. This is a new one but functionally, it performs the same as another set of instruction seen previously. We've seen it before as the following:

```asm
mov esp, ebp
pop ebp
```

 `leave` is usually used at the end of a function to destroy the stack frame. It also has a counterpart known as `enter` which is used at the _beginning_ of a function which replaces these three:

```asm
push ebp
mov ebp, esp
sub n, esp
```

where `n` is the size of the stack. However, because `enter` is slow, it is not used by compilers whereas `leave` is fine. 

Let's do the `leave` operation and analyse the stack. The first suboperation is the `mov esp, ebp`:

```
mov esp, ebp

        ^  Towards top of the stack
        |
+----------------+      ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |
+----------------+ 
|      num2      |
+----------------+  esp = ebp ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |
+----------------+ 
|     b = 2      |
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+      ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|   Saved eip    |  <-- Saved eip for main's ret
+----------------+
|      argc      |
+----------------+
|      argv      |
+----------------+
|      envp      |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

and `pop ebp`:

```
pop ebp

        ^  Towards top of the stack
        |
+----------------+      ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |
+----------------+ 
|      num2      |
+----------------+      ----- Bottom of add's stack frame
|  Saved ebp 2   |  <-- this value gets moved into ebp
+----------------+  esp
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |
+----------------+ 
|     b = 2      |
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|   Saved eip    |  <-- Saved eip for main's ret
+----------------+
|      argc      |
+----------------+
|      argv      |
+----------------+
|      envp      |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

Remember that a `pop` is the opposite of a `push` so `esp` moves down and points to the saved `eip`. The next instruction is `retn` and, coincidentally, `esp` points to saved `eip`.

```
ret

        ^  Towards top of the stack
        |
+----------------+      ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |
+----------------+ 
|      num2      |
+----------------+      ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+  esp
|     a = 1      |
+----------------+ 
|     b = 2      |
+----------------+      ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|   Saved eip    |  <-- Saved eip for main's ret
+----------------+
|      argc      |
+----------------+
|      argv      |
+----------------+
|      envp      |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

`ret` is the same as `pop eip` (this instruction combination doesn't actually exist because we are not allowed to control `eip` like this). So now, `eip` points to the instruction after `call add`. We shift our focus back to the `main` function now.

The next instruction on `eip` is `add esp, 8`. `add`ing moves `esp` downwards by 2 positions (remember that each is 4 bytes wide):

```
add esp, 8

        ^  Towards top of the stack
        |
+----------------+      ----- Top of add's stack frame
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|      num1      |
+----------------+ 
|      num2      |
+----------------+      ----- Bottom of add's stack frame
|  Saved ebp 2   |
+----------------+
|    Saved eip   |  <-- this value points to the next instruction in main
+----------------+
|     a = 1      |
+----------------+ 
|     b = 2      |
+----------------+  esp ----- Top of main's stack frame
|      ...       |
+----------------+
|      ...       |
+----------------+ 
|      ...       |
+----------------+
|     sum = 0    |  <-- main's local variable sum
+----------------+  ebp ----- Bottom of main's stack frame (saved ebp 2 points here)
|  Saved ebp 1   |
+----------------+
|   Saved eip    |  <-- Saved eip for main's ret
+----------------+
|      argc      |
+----------------+
|      argv      |
+----------------+
|      envp      |
+----------------+
|      ...       |
+----------------+
|     Value n    |
+----------------+  <-- Saved ebp 1 value points here
|      ...       |
+----------------+
        |
        v  Towards bottom of the stack
        
      Stack
```

Now we understand why we have `add esp, X`, so it can shift `esp` back down after the `push`es for `add`'s arguments.

After all that stack unwinding, we've restored `main`'s stack frame! Let's finish off this program with:

```asm
mov     [ebp+sum], eax
mov     eax, 0
leave
retn
```

Before the end of `add`, we summed the values and placed them into `eax`. The `eax` register has not been touched since then and so it will continue to hold the value of `num1 + num2` as we defined `return num1 + num2;` in the C code. In `main`, we also defined `sum = add(1, 2);` so the return value of `add` is given to the `sum` local variable as seen with `mov [ebp+sum], eax`. Finally, `return 0;` is fulfilled with `mov eax, 0` before destroying `main`'s stack frame.

In the entirety of this stack gymnastics, I've not removed/cleared any values from the stack. As I've stated in the last tutorial, the values will remain on the stack until something else is moved into their positions.

The end! If you don't understand it in one reading, I suggest that you do it again. If you have a debugger available with you, try stepping through while reading the tutorial to try and make sense of what's happening.
