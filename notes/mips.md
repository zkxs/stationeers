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

## Best Practices
- branching
  - if you're just doing assignment in your branching, it's guaranteed better for your line count to just do selects. Compare a branching-based approach to a `select` based approach.
    ```mips
    # branching solution

    alias condition r0
    alias result r1
    # rest of code...

    beqz condition else
    move result 100
    j end
    else:
    move result 0
    end:

    # rest of code...
    ```
    ```mips
    # select solution

    alias condition r0
    alias result r1
    # rest of code...

    select result condition 100 0
    
    # rest of code...
    ```
    Observe how branching takes 4 instructions (I'm not counting the labels), while `select` takes 1. Even if you can roll your condition calculation into your branch (e.g. using a `blt`), it's still 4 lines vs 2.
  - if you're conditionally doing something side-effecty (like writing to a device, yielding, etc) then obviously you have to branch
  - you can just preprocess your labels out, so don't let labels costing lines discourage you from branching
- 16 registers is actually a lot for a 128 line limited program
  - don't be afraid to reserve registers for a value, even if you could be using that register for more than one thing. If you have say, 4 tmp registers that still leaves you 12 reserved registers. 14 if you abuse `sp` and `ra`!
- use `seqz r0 r0` for inverting `r0`.


# Bizarre Platform Behavior

Now we get into the weird platform-specific stuff. While these are all pretty unintuitive, some of them can be abused to your benefit.

## Byte Counter

ICs show their file size on mouse over. Each character (including newlines) takes two bytes. This isn't really a useful metric, as you can bypass the column limit trivially.

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

Luckily, we can automate the stripping of things like empty lines, comments, and labels via a preprocessor.

The line count limit encourages awful practices like this

```mips
alias result r0
alias offset r1
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
Device ports are your biggest limitations, as you only have 6. Because batch operations (`lb`, `sb`) do not require a device port, they are perfect for abusing. If you can contrive a logic network to only have a single instance of a device type, you can just access it by PrefabHash using `lb` or `sb`.

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

Because the stack does not get cleared, you can have a separate set of "stack loader" programs to load data into the stack. You can then read this data from your main program using `sp`, `peek`, and `pop`. `pop` in particular is useful as it decrements `sp` without actually nuking the stack contents. This means if you need to iterate over 100 values the best way to do it is to set `sp` to 100, then loop over `pop` while `sp` is nonzero. In other words: consider loading your table in reverse order. You can also do all sorts of interesting graph traversal behavior by storing stack pointers in your stack.

## Going Beyond 52 Columns

If you paste a program from your clipboard the 52 character line length limit does not apply. It'll just be impossible to edit the overly-long lines in-game.

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
