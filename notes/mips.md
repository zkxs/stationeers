# MIPS Notes

You can make end of line comments with `#`.

## Limits
- 128 lines of code
- 52 characters per line (not including the \n at the end)
- Maximum of 128 lines executed in a tick before a forced `yield`
- Ticks happen every 0.5 seconds
- You only have six device ports

## Registers
Normal registers: `r0`-`r15`.

`sp` is the stack pointer. It can also be used as `r16`.

`ra` is the return pointer set by certain jumps. It can also be used as `r17`. Typically used via `j ra`.

All of these registers are fully read/write.

Indirect referencing can be done with `dr0`, `rr0`, `rrr0`, etc.

## Devices

Devices: `d0`-`d5`, as well as `db` for the device we're attached to. You can also batch store/load using `sb`/`lb` and a prefab hash. Note that this doesn't require binding the devices to ports.

You may be wondering how to access `db` using `dr0`. Well, `db` is secretly `d2147483647`.

## Best Practices
### Branching
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

### Register Reuse

As you only have 16 registers and you often need a register for only a few lines, there's an obvious benefit for reusing the same register for different jobs throughout your program. While it might be tempting to aggressively do this to every register, 16 registers is actually quite a lot for a 128 line program. Don't be afraid to reserve registers for a value, even if you could be reusing that register for more than one thing. 4 temporary registers is more than enough for most of the math you'll be doing, and that still leaves you 12 reserved registers. 14 if you abuse `sp` and `ra`! 

### Not
Use `seqz r0 r0` for inverting `r0`. While you could do something like `nor r0 r0 r0` that will just confuse people.

### Exponents
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

### The Stack is a Trap
If you're familiar with stack-based VMs, you might be thinking the stack will be useful for doing math. This is not the case, as arithmetic instructions use registers—*not* stack values. You'd end up burning a bunch of instructions loading and unloading stuff from the stack.

If you're familiar with how call stacks work, you may think using the stack is a great way of keeping track of the return address and parameters for nested function calls, and you're not wrong. Here's an example:

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
Observe how this is ridiculous line count overhead. You're much better off aggressively inlining function calls where you can. (Also, I'm aware this could be tail-call optimized but that's beyond the scope of this example).

Where the stack *truly* excels is not as a stack at all, but as RAM or [ROM](#stack-as-rom). Compared to your 18 registers, the 512-word stack is *massive*. If you've got big arrays of data you're tossing around in a loop then your *only* option is to store it in the stack.


# Bizarre Platform Behavior

Now we get into the weird platform-specific stuff. While these are all pretty unintuitive, some of them can be abused to your benefit.

## Byte Counter

ICs show their file size on mouse over. Each character (including newlines) takes two bytes. This isn't really a useful metric, as you can bypass the column limit [trivially](#going-beyond-52-columns).

## Housing Tooltips

The game will use the last bound aliases as tooltips for the housing screws. This can be really confusing if you're moving different programs around as it's not trivial to *clear* these screw tooltips.

## Not Real Assembly

It looks like assembly and it smells like assembly, but it isn't real assembly. It never goes through an assembler. It's an interpreted language. This has some interesting side effects:

- Aliases and defines can be redefined at runtime.
- Whitespace, comments, and labels count against the line/column limit.
- Every line counts as an instruction, including empty lines, comments, and labels! This means they count against the 128 instruction-per-tick limit!

## Golfing

"Golfing" a program is tying to reduce the size of your source code. This is not needed for real assembly, as real assembly is going to use pass through an assembler which will, by its nature, remove things like aliases, labels, comments, and whitespace.

Typically you'd optimize real assembly to minimize CPU cycles or memory. However, the line count limit (while reasonably high for simple programs) is going to be a big limiting factor for larger projects.

Luckily, we can automate the stripping of unnecessary lines via a preprocessor. [icX](https://traineratwot.aytour.ru/wiki/icx) is great for stripping defines, aliases, comments, and whitespace. If you also want to strip out labels, I've made a [simple tool](https://github.com/zkxs/stationeers-ic10-preprocessor) for that.

The line count limit encourages awful practices like this

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

This awful block of code executes uses a stupid-fall through hack to do a jump table with one line per entry. This is just burning cycles for the sake of saving lines which is incredibly stupid, but this is the type of line count optimization you're incentivized to do.

## Abusing Batch Operations
Device ports are your biggest limitation, as you only have 6. Because batch operations (`lb`, `sb`) do not require a device port, they are perfect for abusing. If you can contrive a logic network to only have a single instance of a device type, you can just access it by PrefabHash using `lb` or `sb`. This may have some performance impact, but we're not incentivized to care about that.

If you have two devices of the same type on a network, you can bind one to `d0`, then use `lb` and arithmetic to deduce  the state of the second device without ever using `l` on it directly. For example, if `l r0 d0 Setting` give you a 1, and `lb r0 SOME_HASH Setting Sum` gives you a 10, then we can deduce the second device has a Setting of 9.

Finally, some devices have variants than can be abused to get the same behavior but a different PrefabHash. For example, there are two different types of switch that while having the same behavior, have distinct prefab hashes.

## Shutdown Behavior

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
l r1 d0 r1
```

This means you can access variables by index. This is very useful for say, a lookup table of each gas where you have 
- LogicType of the relevant ratio (e.g. `RatioOxygen`)
- Specific heat of that gas

This would allow you to iterate over the table and save *many* lines of code.

This applies to more than just LogicType (although LogicType is the most useful):

- LogicType
- LogicSlotType
- LogicReagentMode
- LogicBatchMethod
- Every enum type in Assembly-CSharp

## Tick Timing
- All ICs are executed in series at the end of an ElectricityTick
- The order in which they are executed is the reverse order in which their housings were registered.
- Housings are registered when added to a grid.
- ElectricityTick is ran at the end of a GameTick, notably *after* all atmospheric events have ticked.
- It is possible for ICs to programmatically determine their execution order relative to other ICs. ([Video example](https://www.youtube.com/watch?v=m9ZOCTS178o), [example code](../tests/ic10-tests/serial.manual.ic10)). This can be used to establish IC-to-IC communication in a deterministic way.

# Instructions

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

- peek
- pop
- push

## Misc
- alias
- hcf
  - halt and catch fire
- define
- label
  - a label, such as `foo:`. Used as a jump target.
- move
- sleep
  - sleep for `a` seconds
- yield
  - tells your script where to pause until the next tick. If omitted, one will be automatically performed after 128 lines execute in a tick.

## Branching
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
- j
- jal
- jr

## Set Register
- sap
- sapz
- sdns
- sdse
- select
- seq
- seqz
  - Most concise and readable way of performing a `not`
- sge
- sgez
- sgt
- sgtz
- sle
- slez
- slt
- sltz
- sna
- snaz
- sne
- snez

## Logic
- and
- nor
- or
- xor

## Math
- abs
- acos
- add
- asin
- atan
- atan2
- ceil
- cos
- div
- exp
- floor
- log
- max
- min
- mod
- mul
- rand
- round
- sin
- sqrt
- sub
- tan
- trunc


# Reverse Engineering
- ScriptCommand.cs is an enum of all commands that can run
- ProgrammableChip.cs is the MIPS interpreter
- ICircuitHolder.cs is implemented by everything that an IC10 can be slotted into
- `mips instructions.txt` is an alphabetized list of instructions from ScriptCommand.cs for the purpose of diffing
