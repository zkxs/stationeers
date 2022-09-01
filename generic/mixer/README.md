# Gas mixing
- Gas mixing is done by volume, but you actually care about mixing by moles
- So you can either equalize temperature, or just do math

## The Game Code

From data mining:

```csharp
float Ratio1 = OutputSetting;
float Ratio2 = 100f - OutputSetting;

float transferMoles1 = (PressurePerTick * (Ratio1 / 100.0) * OutputNetwork.Atmosphere.Volume / (InputNetwork.Atmosphere.Temperature * 8.3143997192382813));

float transferMoles2 = (PressurePerTick * (Ratio2 / 100.0) * OutputNetwork.Atmosphere.Volume / (InputNetwork2.Atmosphere.Temperature * 8.3143997192382813));
```

To translate that into math:

```
s = setting / 100
n1 = P * s * V / (T1 * R)
n2 = P * (1 - s) / 100 * V / (T2 * R)
```

## Algebra Time

Solving both equations for `R / (P * V)` yields

```
R / (P * V) = s / (T1 * n1)
R / (P * V) = (1 - s) / (T2 * n2)
```

Setting those equal to each other yields:

```
s / (T1 * n1) = (1 - s) / (T2 * n2)
```

Solving for `s` yields

```
s = (T1 * n1) / (T1 * n1 + T2 * n2)
```

Finally, translating `s` back to setting yields

```
Setting = (T1 * n1 * 100) / (T1 * n1 + T2 * n2)
```

## Making it into IC10 Code

- Doing any 1:X hardcoded mix (e.g. 1:2 O2:H2) can be optimized down to 4 instructions using just 1 register. This is because multiplication by 1 is a no-op.
- Doing a variable mix (e.g. X:Y) can be optimized down to 5 instructions using 2 registers. This is being done in [mixer.ic10](mixer.ic10).
