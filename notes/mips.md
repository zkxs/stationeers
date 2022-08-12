# MIPS Notes

These are my personal MIPS notes. This is less of a MIPS tutorial and more documentation of the weird MIPS internals I've stumbled across.

If you're just getting started with MIPS, check out these resources:

- [MIPS Wiki page](https://stationeers-wiki.com/MIPS)
- [Advanced MIPS Wiki page](https://stationeers-wiki.com/Advanced_IC10_Programming)
- [Stationeering Tools IC Simulator](https://stationeering.com/tools/ic)

If you're already somewhat comfortable with "normal" MIPS and are ready to pull back the curtain and get some cold, hard **details** then this page might be useful to you.

## Limits

- 128 lines of code
- 52 characters per line (not including the \n at the end). Can be [bypassed](#going-beyond-52-columns).
- Maximum of 128 lines executed in a tick before a forced `yield`
- Ticks happen every 0.5 seconds
- Six device ports
- 18 1-word registers
- 512 word stack (which supports random access)
- The "words" are all doubles ([IEEE-754](https://en.wikipedia.org/wiki/IEEE_754) double-precision floating point numbers)

## Registers

Normal registers: `r0`-`r15`.

`r16` or `sp` is the stack pointer. It's read by `push`, `pop`, and `peek`. It's written by `push` and `pop`.

`r17` or `ra` is the return pointer set by certain jumps. Nothing automatically reads it, but and it's typically used via `j ra`.

All registers are fully read/write, even `sp` and `ra`.

Indirect referencing can be done with `dr0`, `rr0`, `rrr0`, etc.

## Devices

Devices: `d0`-`d5`, as well as `db` for the device we're attached to. You can also batch store/load using `sb`/`lb` and a prefab hash. Note that this doesn't require binding the devices to ports.

You may be wondering how to access `db` using `dr0`. Well, `db` is secretly `d2147483647`.

## Byte Counter

ICs show their file size on mouse over. Each character (including newlines) takes two bytes. This isn't really a useful metric, as you can bypass the column limit [trivially](#going-beyond-52-columns).

## Housing Tooltips

The game will use the last bound aliases as tooltips for the housing screws. This can be confusing if you're moving different programs around as it's not trivial to *clear* these screw tooltips.

# Best Practices
## Branching

- if you're just doing assignment in your branching, it's guaranteed better for your line count to just do selects. Compare a branching-based approach to an equivalent `select`-based approach.
  ```mips
  bge a b else # if a < b
    move result 100
    j end
  else:
    move result 0
  end:
  ```
  ```mips
  slt condition a b
  select result condition 100 0
  ```
  Observe how branching takes 4 instructions (I'm not counting the labels), while `select` takes 2.
- if you're conditionally doing something side-effecty (like writing to a device, yielding, etc) then obviously you have to branch
- you can just preprocess your labels out, so don't let labels costing lines discourage you from branching

## Register Reuse

As you only have 18 registers and you often need a register for only a few lines, there's an obvious benefit for reusing the same register for different jobs throughout your program. While it might be tempting to aggressively do this to every register, 18 registers is actually quite a lot for a 128 line program. Don't be afraid to reserve registers for a value, even if you could be reusing that register for more than one thing. 4 temporary registers is more than enough for most of the math you'll be doing, and that still leaves you 14 reserved registers.

## Not
Use `seqz r0 r0` for inverting `r0`. While you could do something like `nor r0 r0 r0` that will just confuse people.

## Exponents

- x<sup>2</sup>
  ```mips
  # pow(x, 2) = x * x
  # store result in x
  mul x x x
  ```
- x<sup>3</sup>
  ```mips
  # pow(x, 3) = x * x * x
  # store result in x
  mul tmp x x
  mul x x tmp
  ```
- use `exp` and `log` to calculate arbitrary powers.  x<sup>y</sup> = *e*<sup>y*ln(x)</sup>
  ```mips
  # calculate pow(x, y) = exp(y*log(x))
  # store result in x
  log x x
  mul x x y
  exp x x
  ```

## The Stack is a Trap

If you're familiar with stack-based VMs, you might be thinking the stack will be useful for doing math. This is not the case, as arithmetic instructions use registers—*not* stack values. You'd end up burning a bunch of instructions loading and unloading stuff from the stack.

If you're familiar with how call stacks work, you may think using the stack is a great way of keeping track of the return address and parameters for nested function calls. While you're not wrong, it's a lot of overhead. Here's an example:

```mips
alias tmp r0

# tmp=foo(1)
push 1
jal fn_foo
pop tmp
s d0 Setting tmp
j end


# Square the input, then add 1
fn_foo:
pop tmp # load parameter 1
mul tmp tmp tmp # square the input

push ra # back up ra

# tmp=bar(tmp)
push tmp
jal fn_bar
pop tmp

pop ra # restore ra

push tmp # return tp
j ra

# Add one to the input
fn_bar:
pop tmp # load parameter 1
# no need to back up ra as we call nothing
add tmp 1 # add one
push tmp # return tmp
j ra

end:
```
Observe how this creates ridiculous line count overhead. You're much better off aggressively inlining function calls where you can. (Also, I'm aware this could be tail-call optimized but that's beyond the scope of this example).

Where the stack *truly* excels is not as a stack at all, but as RAM or [ROM](#stack-as-rom). Compared to your 18 registers, the 512-word stack is *massive*. If you've got big arrays of data you're tossing around in a loop then your *only* option is to store it in the stack.

# Optimizing MIPS

In real life, time and memory are a program's two biggest limiting factors so you naturally want to optimize for reducing them. However, MIPS has very extreme artificial limitations on line count and connected devices. So our two optimization targets become using fewer lines and binding fewer devices.

## Optimizing for Line Count: Golfing

We can't talk about MIPS without talking about code golf. "Golfing" a program is tying to reduce the size of your source code. Normally, this is a purely [recreational activity](https://en.wikipedia.org/wiki/Code_golf), as the size of the source code doesn't typically matter in real life. As MIPS source code is interpreted directly and has a very low 128-line limit, you will *need* to golf your code to fit larger programs onto a single IC10.

Luckily, we can automate the stripping of unnecessary lines via a preprocessor. [icX](https://traineratwot.aytour.ru/wiki/icx) is great for stripping defines, aliases, comments, and whitespace. If you also want to strip out labels, I've made a [simple tool](https://github.com/zkxs/stationeers-ic10-preprocessor) for that.

The line count limit also encourages awful hacks like this:

```mips
move result 0
jr offset
move result 1
select result result result 2
select result result result 3
select result result result 4
select result result result 5
select result result result 6
# end of lookup table
```

This block of code uses a fall-through hack to create a jump table with one line per entry. This is just burning cycles for the sake of saving lines which is unfortunate, but this is the type of line count optimization you're incentivized to do.

## Optimizing for Device Ports: Abusing Batch Operations

Device ports are hugely limited, as you only have 6. Because batch operations (`lb`, `sb`) do not require a device port, they are ripe for abuse. If you can contrive a logic network to only have a single instance of a device type, you can just access it by PrefabHash using `lb` or `sb`. This may have some performance impact, but we're not incentivized to care about that.

If you have two devices of the same type on a network, you can bind one to `d0`, then use `lb` and arithmetic to deduce  the state of the second device without ever using `l` on it directly. For example, if `l r0 d0 Setting` give you a 1, and `lb r0 SOME_HASH Setting Sum` gives you a 10, then we can deduce the second device has a Setting of 9.

Finally, some devices have variants than can be abused to get the same behavior but a different PrefabHash. For example, there are two different types of switch that while having the same behavior, have distinct prefab hashes.

## Isn't Golfing Bad for Readability?

Yes. Small programs shouldn't be golfed so that you can maintain readability. For large programs you should automate your golfing. You edit the ungolfed "source" code, and your minification program generates "compiled" output that you *don't* edit by hand.

## Isn't Golfing Bad for Performance?

Yes. Optimizing for line count above all else is 100% going to increase your executed instruction count. Furthermore, `lb` and `sb` are more expensive than `l` and `s`. In other words: you'll be doing more CPU work each tick. But the game incentivizes you to do this. If the devs want to encourage players to write more performant code, they need to shift to [limitations that actually encourage performance](mips-incentives.md).

Also, regarding the line count limit, back in April 2021 the Stationeers developers said this ([source](https://discord.com/channels/276525882049429515/277443989794324480/828130969830817792)):

> We investigated having an IC with 256/512 lines of code. Unfortunately the current implementation means that having too many lines lags out the IC editor UI. So we're stuck with 128 lines pending a major refactor of the IC system.

# Bizarre Platform Behavior

Now we get into the weird platform-specific stuff that we can abuse to our benefit.

## Not Real Assembly

It looks like assembly and it smells like assembly, but it isn't real assembly. It never goes through an assembler. It's an interpreted language. This has some interesting side effects:

- Aliases and defines can be redefined at runtime.
- Whitespace, comments, and labels count against the line/column limit.
- Every line counts as an instruction, including empty lines, comments, and labels! This means they count against the 128 instruction-per-tick limit!

## Double Literals

Allowed:

- Doubles can have thousands separators `1,000.0`, and leading `+100` or trailing `100.0-` signs.
- `Infinity` and `-Infinity` are valid literals.

Not Allowed:

- Exponents such as `1e100` do not parse

Sort of Allowed:

- `NaN` parses, but most instructions treat a NaN literal as an error. `define`, however works. Which means you can `define NaN NaN` to get a working NaN literal. If you just need NaN in a register, you can do `sqrt r0 -1` or something similar.

## NaN Checking
NaN != NaN. This behavior isn't actually a MIPS thing, it's an IEEE-754 thing. This means if you need to check a number `n` for NaN, the best way to do it is:

```mips
# Check if n is NaN
sne isNan n n
```

## Shutdown Behavior

You might assume that state like registers and stack gets cleared when you unplug an IC. You'd assume wrong, then. ICs retain their state, and in fact their state is properly serialized to your world's save file.

Most of this behavior is consistent across different ways of shutting off an IC. I've bolded the inconsistent stuff:

- On shutdown (toggling it off or cutting power)
  - r0-r15 are NOT reset
  - ra is NOT reset
  - **sp is NOT reset**
  - stack values are NOT reset
  - **instruction counter is NOT reset**
- On unplug
  - r0-r15 are NOT reset
  - ra is NOT reset
  - sp is reset
  - stack values are NOT reset
  - instruction counter is reset
- On reprogram
  - r0-r15 are NOT reset
  - ra is NOT reset
  - sp is reset
  - stack values are NOT reset
  - instruction counter is reset

In summary: Shutdown pauses execution and doesn't reset anything, even the instruction counter! It will resume where it left off! Unplugging or reprogramming resets the instruction counter and `sp`, but all other registers and the stack contents are NOT reset. Note that values persist to the *chip*, not the housing. You can also persist something to the housing with `db.Setting` if that's helpful.

## Stack as ROM

Because the stack does not ever get cleared, you can have a separate set of "stack loader" programs to load data into the stack. You can then read this data from your main program using `sp`, `peek`, and `pop`. `pop` in particular is useful as it decrements `sp` without actually zeroing the stack contents. This means if you need to iterate over 100 values the best way to do it is to set `sp` to 100, then loop over `pop` while `sp` is nonzero. In other words: consider loading your table in reverse order. ([Example of a stack-based lookup table](../tests/ic10-tests/stack-table-example.manual.ic10)).

You can also do all sorts of interesting graph traversal behavior by storing stack pointers in your stack.

## Going Beyond 52 Columns

If you paste a program from your clipboard the 52 character line length limit does not apply. It'll just be impossible to edit the overly-long lines in-game. This trick cannot be used to bypass the 128-line limit.

## Free Yield After Jump

If you jump to the negative of a target line number, e.g. `j -60`, it will perform a yield immediately after the jump. This is a weird quirk of how yields are represented in the mips interpreter. This also means you cannot use this trick for a `j 0` as 0 cannot be negative. (Yes, I know an IEEE-754 float can represent -0.0, but it doesn't actually matter here for .NET reasons).

## LogicType is an Integer

Well, actually it's a byte. For example, `Setting` has the value `12`. If want to store the value of a LogicType into a register, you can use something like `LogicType.Setting`.

```mips
move r0 LogicType.Setting

# These all do the same thing
l r1 d0 Setting
l r1 d0 12
l r1 d0 r0
```

This means you can access variables by index. This allows you to iterate over a table and save *many* lines of code. Take for example a [lookup table of each gas](../tests/ic10-tests/stack-table-example.manual.ic10) where you have 
- LogicType of the relevant ratio (e.g. `RatioOxygen`)
- Specific heat of that gas

This trick applies to more than just LogicType (although LogicType is the most useful):

- LogicType
- LogicSlotType
- LogicReagentMode
- LogicBatchMethod
- Every enum type in Assembly-CSharp

## Tick Timing

All ICs are executed in **series** at the end of an ElectricityTick.

- **series** means they execute one after another. NOT at the same time.
- This means bidirectional communication within a single tick is impossible. Yes, even if you busy loop.
- Because IC execution happens in one big batch each tick and isn't "spread out" over time, ICs will always tick in perfect sync with each other. As long as you don't get auto-yielded the way ICs tick is completely deterministic.
- It is possible for ICs to programmatically determine their execution order relative to other ICs. ([Video example](https://www.youtube.com/watch?v=m9ZOCTS178o), [example code](../tests/ic10-tests/serial.manual.ic10)). This can be used to establish IC-to-IC communication in a deterministic way.
- The order in which ICs are executed is the reverse order from which their housings were registered. Housings are registered when added to a grid (a.k.a. when built). This order appears to persist across world save/load.
- ElectricityTick is ran at the end of a GameTick, notably *after* all atmospheric events have ticked.

# Instructions

Most of these are already reasonably well-documented on the [MIPS Wiki page](https://stationeers-wiki.com/MIPS) and the [Stationeering Tools IC Simulator](https://stationeering.com/tools/ic). I'm only going to add detail where the wiki is lacking or where there are weird edge cases.

## Device IO

- l
- lb
- lr
- ls
- s
- sb

## Stack

Can hold 512 values. A special `sp` register points to the next index to be pushed.

Valid indices range from 0 through 511. Reading/writing an invalid index will result in a stack overflow/underflow error.

- peek \<r\>
  - Read the stack value at index `sp - 1` into `r`.
- pop \<r\>
  - Read the stack value at index `sp - 1` into `r`, then decrement `sp`.
  - This does not zero the read value in the stack: it *only* decrements `sp`.
- push \<r\>
  - Push the value of `r` into the stack at index `sp`, then increment `sp`.

## Misc

- alias
- hcf
  - Halt and Catch Fire. This instruction is rarely documented, for good reason.
- define
- label
  - a label, such as `foo:`. Used as a jump target.
- move
- sleep \<x\>
  - sleep for `x` seconds. Can be fractional? It's going to get rounded to a tick somehow.
- yield
  - Tells your script to pause until the next tick. If omitted, one will be automatically performed after 128 lines execute in a tick.

## Logic

These are logical operations, *not* bitwise operations.

- and
- nor
- or
- xor

## Math

- abs
- add
- ceil \<r\> \<x\>
  - Assigns to `r` the smallest integral value greater than or equal to `x`. In other words, round towards positive infinity.
- div \<r\> \<x\> \<y\>
  - Floating-point division. `r` = `x / y`. 
- exp \<r\> \<x\>
  - *e*<sup>`x`</sup>. See [`Math.exp(x)`](https://docs.microsoft.com/en-us/dotnet/api/system.math.exp) for edge cases.
- floor \<r\> \<x\>
  - Assigns to `r` the largest integral value less than or equal to `x`. In other words, round towards negative infinity.
- log \<r\> \<x\>
  - Natural log. See [`Math.log(x)`](https://docs.microsoft.com/en-us/dotnet/api/system.math.log) for edge cases.
- max
- min
- mod \<r\> \<x\> \<y\>
  - Misleadingly, this does not perform a modulo operation. From Wikipedia:

    > In computing, the modulo operation returns the remainder or signed remainder of a division, after one number is divided by another (called the modulus of the operation).

    The .NET remainder operator (`%`) computes the remainder of a division. The result will have the sign of the dividend (left-hand operand).

    MIPS is doing something very different. If `x` is positive, then `mod r x y` has the same behavior as `r = x % y`. If `x` is negative, then `mod r x y` instead performs `r = (x % y) + y`.

    ```cs
    using System;
    
    public class ModuloTest
    {
        public static void Main() {
            // C# behavior
            Console.WriteLine(  csMod( 1.6,  1)); //  0.6
            Console.WriteLine(  csMod(-1.6,  1)); // -0.6
            Console.WriteLine(  csMod(-1.6, -1)); // -0.6
            Console.WriteLine(  csMod( 1.6, -1)); //  0.6
            
            // MIPS behavior
            Console.WriteLine(mipsMod( 1.6,  1)); //  0.6
            Console.WriteLine(mipsMod(-1.6,  1)); //  0.4 wat
            Console.WriteLine(mipsMod(-1.6, -1)); // -1.6 WHAT THE FUCK
            Console.WriteLine(mipsMod( 1.6, -1)); //  0.6
        }
        
        // performs a mod exactly as .NET does
        public static double csMod(double x, double y) {
            return x % y;
        }
        
        // performs a mod exactly as Stationeers MIPS does
        public static double mipsMod(double x, double y) {
            double r = x % y;
            if (r < 0) {
                r += y;
            }
            return r;
        }
    }
    ```

    `mod r x y` is clearly intended to allow you to loop `r` from 0 to `y` to generate a sawtooth wave. When `y` is positive, this is in fact how it behaves. Such behavior is useful for looping a number, for example looping a rotation to remain between 0 and 360 even if your input goes out of that range.

    However, when both `x` and `y` are negative `mod r x y` behaves in an unexpected, useless way. This appears to be due to an oversight in its implementation.

    I've illustrated the behavior of various flavors of remainder and repeat operations below:

    ![Graph showing (x % 1)](../img/Various%20Flavors%20of%20x%20mod%201.png)

    ![Graph showing (x % -1)](../img/Various%20Flavors%20of%20x%20mod%20-1.png)
- mul
- rand \<r\>
  - Assigns to `r` a random double that is greater than or equal to 0.0, and less than 1.0. A single static PRNG is used that is seeded based on time (milliseconds since .NET started).
- round \<r\> \<x\>
  - Assign to `r` the integer nearest `x`. If the fractional component of `x` is halfway between two integers, one of which is even and the other odd, then the even number is returned.
- sqrt
- sub
- trunc \<r\> \<x\>
  - Removes any fractional part of `x`, storing the result in `r`.

## Trigonometry

All trigonometry functions use radians for angles.

- acos
- asin
- atan
- atan2 \<r\> \<y\> \<x\>
  - performs a [`Math.Atan2(y, x)`](https://docs.microsoft.com/en-us/dotnet/api/system.math.atan2) and stores the result in `r`.
- cos
- sin
- tan

## Conditional Branching

- bap
- bapal
- bapz
- bapzal
- bdns
- bdnsal
- bdse
- bdseal
- beq
- beqal
- beqz
- beqzal
- bge
- bgeal
- bgez
- bgezal
- bgt
- bgtal
- bgtz
- bgtzal
- ble
- bleal
- blez
- blezal
- blt
- bltal
- bltz
- bltzal
- bna
- bnaal
- bnaz
- bnazal
- bne
- bneal
- bnez
- bnezal
- brap
- brapz
- brdns
- brdse
- breq
- breqz
- brge
- brgez
- brgt
- brgtz
- brle
- brlez
- brlt
- brltz
- brna
- brnaz
- brne
- brnez

## Unconditional Jumps

- j
- jal
- jr

## Set Register

- sdns
- sdse
- select
- seq
- seqz
- sge
- sgez
- sgt
- sgtz
- sle
- slez
- slt
- sltz
- sne
- snez

## Approximate Comparisons

Used for comparing *almost* equal values.

- sap
- sapz
- sna
- snaz

# Reverse Engineering
Useful for when you toss Stationeers into a decompiler.

- ScriptCommand.cs is an enum of all instructions that can be used
- ProgrammableChip.cs is the MIPS interpreter
- ICircuitHolder.cs is implemented by everything that an IC10 can be slotted into
- [mips instructions.txt](mips%20instructions.txt) is an alphabetized list of instructions from ScriptCommand.cs for the purpose of diffing across Stationeers versions. I can use this to see if an undocumented instruction gets added.
