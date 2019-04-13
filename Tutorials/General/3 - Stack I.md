# Stack I

In this tutorial, we'll be going over the concept of the stack. This concept will cover the information over the past two tutorials regarding the `ebp` and `esp` registers as well as the `push` and `pop` instructions. It is fundamental knowledge that is needed to understand function calls.

## Stack Layout

The stack is a special area of memory that is reserved for storing, preserving and passing data around by the code. It has the name "stack" because it is as the word describes, a "stack" of data. Imagine the stack as a pile of plates where the data are plates. 

```
+----------------+  Top
|     Plate 1    |
+----------------+
|     Plate 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Plate n    |
+----------------+  Bottom

 Stack of plates
```

The way you **store** (or **preserve**) the plate is by putting it on **top** of the stack, i.e. `push`ing. 

```
Pushing a plate onto the top

+----------------+
|     Plate 0    |  <- push
+----------------+
        |
        v
+----------------+  Top
|     Plate 1    |
+----------------+
|     Plate 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Plate n    |
+----------------+  Bottom

 Stack of plates
```

The way you **remove** a plate is by taking it **from the top**, i.e. `pop`ing. 

```
Popping a plate off the top

         ^
         |
+----------------+
|     Plate 0    |  <- pop
+----------------+

+----------------+  Top
|     Plate 1    |
+----------------+
|     Plate 2    |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|      ...       |
+----------------+
|     Plate n    |
+----------------+  Bottom

 Stack of plates
```

The stack is defined to "grow" (by `push`ing) towards **lower** memory address values, i.e. towards 0, whereas the bottom of the stack points towards the **higher** memory address values. And because we `push` and `pop` from the top, we will display the top of the stack at... the top... and the bottom at... the bottom... Makes sense, right? From my experience, it's commonly taught the other way around (flipped) which doesn't help at all. Consistency is key, so follow one convention. These tutorials will use the former convention.

## Function Prologues and Epilogues

In all of the examples we've seen, they all had the same sequence of instructions:

```asm
push ebp
mov ebp, esp

; ...

pop ebp
```

These two registers help create the current stack "frame". The `ebp` register is the "base pointer" and points to the bottom of this frame and the `esp` register is the "stack pointer" and points to the top of the frame. Together, they define a region of memory which is the function's frame or scope. Within this region, the function usually stores its local variables as well as other data such as preserving register values to free them up for use and stack cookies.

Let's go through the first instruction `push ebp`. Here is an example of a stack before the instruction:

```
Before pushing ebp

        +----------------+  Top of stack = 0x0
0x0     |      ...       |
        +----------------+
0x4     |      ...       |
        +----------------+
0x8     |      ...       |
        +----------------+  esp = 0xC
0xC     |     Value 1    |
        +----------------+
0x10    |     Value 2    |
        +----------------+
0x14    |      ...       |
        +----------------+
0x18    |      ...       |
        +----------------+
0x1C    |      ...       |
        +----------------+
0x20    |     Value n    |
        +----------------+  ebp = 0x20
0x24    |      ...       |
        +----------------+
0x28    |      ...       |
        +----------------+
0x2C    |      ...       |
        +----------------+  Bottom of stack = 0x2C
        
              Stack
```

We see `ebp` at the bottom and `esp` at the top. Together, we see that they form the function's frame. Each "segment" is 4 bytes. To `push` a value, there are two steps: move `esp` up one "segment" (4 bytes) and then move the value into the "newly created segment" (like pushing a plate) like so:

```
Pushing ebp

        +----------------+  Top of stack = 0x0
0x0     |      ...       |
        +----------------+
0x4     |      ...       |
        +----------------+  esp = 0x8 <-- move esp up a "segment" (new esp)
0x8     |   ebp = 0x20   |  <-- Insert the value of ebp here
        +----------------+  (old esp here)
0xC     |     Value 1    |
        +----------------+
0x10    |     Value 2    |
        +----------------+
0x14    |      ...       |
        +----------------+
0x18    |      ...       |
        +----------------+
0x1C    |      ...       |
        +----------------+
0x20    |     Value n    |
        +----------------+  ebp = 0x20
0x24    |      ...       |
        +----------------+
0x28    |      ...       |
        +----------------+
0x2C    |      ...       |
        +----------------+  Bottom of stack = 0x2C
        
              Stack
```

Now let's do the `mov ebp, esp` instruction. Remember that `mov` is a **copy**!

```
Move esp into ebp

        +----------------+  Top of stack = 0x0
0x0     |      ...       |
        +----------------+
0x4     |      ...       |
        +----------------+  esp = ebp = 0x8 <-- move ebp here (new ebp)
0x8     |   ebp = 0x20   |  
        +----------------+  (old esp here)
0xC     |     Value 1    |
        +----------------+
0x10    |     Value 2    |
        +----------------+
0x14    |      ...       |
        +----------------+
0x18    |      ...       |
        +----------------+
0x1C    |      ...       |
        +----------------+
0x20    |     Value n    |
        +----------------+  (old ebp here)
0x24    |      ...       |
        +----------------+
0x28    |      ...       |
        +----------------+
0x2C    |      ...       |
        +----------------+  Bottom of stack = 0x2C
        
              Stack
```

Right now, the frame is flat but in a later, we will go through an example of growing the frame to reserve space for data storage.

And finally, the last instruction `pop ebp`. `pop`ing is the reverse of `push` and is done like so: move the value, from where `esp` points, into the specified register in the instruction (`ebp`) and move `esp` down a "segment" (4 bytes) (like removing a plate). Here is what it will look like:

```
Popping ebp

        +----------------+  Top of stack = 0x0
0x0     |      ...       |
        +----------------+
0x4     |      ...       |
        +----------------+  (old esp and ebp here)
0x8     |   ebp = 0x20   |  <-- give this value to ebp
        +----------------+  esp = 0xC <-- move esp down a "segment" (original esp)
0xC     |     Value 1    |
        +----------------+
0x10    |     Value 2    |
        +----------------+
0x14    |      ...       |
        +----------------+
0x18    |      ...       |
        +----------------+
0x1C    |      ...       |
        +----------------+
0x20    |     Value n    |
        +----------------+  ebp = 0x20  <-- move ebp here (original ebp)
0x24    |      ...       |
        +----------------+
0x28    |      ...       |
        +----------------+
0x2C    |      ...       |
        +----------------+  Bottom of stack = 0x2C
        
              Stack
```

Notice that the `ebp` value on the stack remains there! It does not get cleared and will only change when something else is moved there to replace it!
