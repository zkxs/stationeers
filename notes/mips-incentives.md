# How To Incentivize Performant Code

This is a suggestion for how the developers could shift the incentive from golfing to writing performant code.

1. Raise the line limit significantly, as a low line limit encourages bad practices such as:
   - Using multiple ICs to solve a single problem. As you get 128 instructions per tick per IC the devs should be making it *easier* to solve problems with one IC, not *harder*.
   - Doing the same work with less branching, thereby executing more instructions per tick.
   - Stripping documentation and whitespace out of your code, thereby making the player experience worse.
2. Give players a housing option with more device ports. A low device port limit encourages bad practices such as:
   - Using multiple ICs to solve a single problem (see above)
   - Using more expensive batch store/load instructions
3. Give instructions a "cycle" cost roughly proportional to how expensive the instruction is. For example, a comment could be 0 cycles, an `add` could be 1 cycle, an `l` could be 4 cycles, an `lb` could be 16 cycles. This fits with the assembly theme of the MIPS language.
4. Change the "128 lines per tick" limit to a "128 *cycles* per tick" limit.

This proposed solution would:
- Encourage solving problems using fewer, less expensive instructions
- Encourage solving problems using fewer ICs
- Stop discouraging code readability
