# Gas mixing
- Gas mixing is done by volume, but you actually care about mixing by moles
- So you can either equalize temperature, or just do math
- Setting = (T1 * n1 * 100) / (T1 * n1 + T2 * n2)
- Doing any 1:X hardcoded mix (e.g. 1:2 O2:H2) can be optimized down to 4 instructions using just 1 register. This is because multiplication by 1 is a no-op.
- Doing a variable mix (e.g. X:Y) can be optimized down to 5 instructions using 2 registers.
