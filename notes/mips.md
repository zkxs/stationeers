# Things
- ScriptCommand.cs is an enum of all commands that can run
- ProgrammableChip.cs is the MIPS interpreter
- ICircuitHolder.cs is implemented by everything that an IC10 can be slotted into
- `mips instructions.txt` is an alphabetized list of instructions from ScriptCommand.cs for the purpose of diffing


# Limits
- 128 lines of code
- 52 characters per line (not including the \n at the end)
- You must yield or sleep at least once in 128 lines executed or a yield will be performed for you

# Registers
Normal registers: `r0`-`r15`.

Devices: `d0`-`d5`, as well as `db` for the device we're attached to.

`sp` is the stack pointer. It can also be used as `r16`.

`ra` is the return pointer set by certain jumps. It can also be used as `r17`. Typically used via `j ra`.

All of these registers are fully read/write.

# Extra stuffs
End of line comments with `#`

# Best Practices
- branching
  - if you're just doing assignment in your branching, it's guaranteed better for your line count to just do selects
  - if you're conditionally doing something side-effecty (like a yield) then obviously you have to branch
  - you can just preprocess your labels out, so don't let labels costing lines discourage you from branching
- 16 registers is actually a lot for a 128 line limited program
  - don't be afraid to reserve registers for a value, even if you could be using that register for more than one thing
  - if you have say, 4 tmp registers that still leaves you 12 reserved registers. 14 if you abuse sp and ra!
- use `seqz` for `not`

# Shutdown behavior
- On shutdown
  - r0-r15 are NOT reset
  - ra is NOT reset
  - sp is NOT reset
  - stack values are NOT reset
  - instruction counter is NOT reset
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

In summary: Shutdown pauses execution and doesn't reset anything, even the instruction counter! It will resume where it left off! Unplugging or reprogramming resets the instruction counter and `sp`, but all other register and the stack contents are NOT reset. Note that values persist to the *chip*, not the housing. You can also persist something to `db.Setting` if that's helpful.

# Housing tooltips
The game will use the last bound aliases as tooltips for the housing screws. This can be really confusing if you're relying on them, and moving different ICs around
as it's not trivial to *clear* these screw tooltips.

# Bizarre Tricks
- If you jump to a negated line number, e.g. `j -60`, it will perform a yield immediately after the jump. This is a weird quirk of how yields are represented in the mips interpreter.
  This also means you cannot use this trick for a `j 0` as 0 cannot be negative.

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
- sge
- sgez
- sgt
- sgtz
- sin
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
- sqrt
- sub
- tan
- trunc
