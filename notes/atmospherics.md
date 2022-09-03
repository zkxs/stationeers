# Atmospherics Notes

## Constants
| Constant                    | Value                              |
|-----------------------------|------------------------------------|
| Idea gas constant           | 8.31439971923828 (kPa*L)/(K\*mol) |
| One atmosphere              | 101.325 kPa                        |
| Zero Celsius                | 273.15 K                           |
| Armstrong limit             | 6.3 kPa                            |

## Important Equations

**Ideal Gas Law**  
PV=nRT

**Definition of Heat Capacity**  
Q=ncT

### Derived Equations

**Heat in terms of pressure and volume**  
QR=PVc

## Jetpack

- It emits moles of gas, therefore you might as well supercool your canister
- Any gas will do, so use whatever is plentiful and nontoxic

## Oxygen

- Your suit AC is going to have to fix whatever temperature your oxygen tank is, so try to get it to around your suit's target temperature. It's not worth it to try and maximize moles by cooling it.

## Breathing

- You need 20 kPa total pressure to not get hurt
- Entity.cs
- But lets do the math and find how how good this is.
  - 0.0012 moles per breath, times an efficiency multiplier
  - MinimumOxygenPartialPressure = 16
  - efficiency = atmos.PartialPressureO2 / MinimumOxygenPartialPressure
    - bigger numbers are worse, because this is a mole consumption multiplier
    - so at 16 kPa of O2 your efficiency is 1
    - So the best possible airmix would be 16 kPa O2 4 kPa filler to get to the 20 kPa pressure requirment.
  - so if we're breathing optimally we're doing 0.0012 moles per tick, or 0.0024 moles per second.
  - What is the volume of a GasMask?
  - What is the volume of a suit?

## Autoignition

Active vents autoignite at 300C, like most devices. That's far too low for use in a pressure cooker room. However, the second condition for autoignition is 10MJ of energy in the room, which cannot be reached unless the room is above 368.55 kPa. So as long as you keep the pressure below that point it doesn't matter how hot the room gets: it won't spontaneously combust (unless sparked).

### Math Zone

If you're curious where the hell I got that magical "368.55 kPa" number from, here's the math:

`T=Q/(nc)` comes from `Q=ncT`  
`P=nRT/V` comes from `PV=nRT`  
substituting `T=Q/(nc)` into `P=nRT/V` and simplifying gets us `P=(QR)/(Vc)`  

- `Q` = 10 MJ *(autoignition required energy)*
- `R` = 8.314399719 (kPa*L)/(K*mol) *(ideal gas constant)*
- `V` = 8000 L *(volume of a grid)*
- `c` = 28.2 J/(mol\*K) *(specific heat of CO2, which is the worst-case gas as it can hold the most energy per mole)*

Substituting those values into `P=(QR)/(Vc)` yields 368.55 kPa, assuming the atmosphere is 100% CO2. If it's not you can handle slightly higher pressures. For example, for a 50% CO2 50% X mix your autoignition pressure would be 392.19 kPa

### TLDR

So the TLDR is: keep your furnace room between 101.325 and 368.55 kPa to prevent both radiative cooling and autoignition. That room can now be any temperature and structures won't burn unless sparked.

### Things that won't catch on fire

- the beefier doors
  - blast doors (also pressure immune, which is a nice bonus)
  - airlock doors (don't go over 1 MPa)
  - glass door (WTF, but it can only handle a 200 kPa delta, so maybe don't use this)
- passive vents
- atmospheric devices (pumps, valves, etc)
- heavy cable (technically they can burn, but 10,000C is way too high to reach typically)
- frames
- hardsuits
- most atmospheric devices
  - pipes
  - pumps
  - pipe analyzer
  - filtration units
  - etc
- gas sensor
- igniter
- jetpacks
- items inside your backpack
- items inside your uniform
- items inside your suit
- items inside your toolbelt

### Things that *will* catch on fire

- one way valves
- most electronics
  - light cable
  - Active vents
  - APC
  - buttons and switches
  - etc
- walls
- windows
- weaker doors
  - composite door
  - manual hatch
  - airlock gate
  - interior doors
- non-hard suits
- hardsuit backpack
- your glasses
- anything held in your hand
- cells in drills, grinders, or welders **NO MATTER HOW DEEPLY THEY ARE NESTED IN INVENTORIES**
- chutes
- one-way valves

## Sterling

- at least 3000 kPa delta-P for optimal
- wtf bruh. do I need to actively cool the output net?
  - looks like NO? wtf.
  - So I'm just forced to pump from output back to input it for that 3MPa delta-P that I need

Some variables:
- _hotInputAtmosphere
  - Atmosphere that exhanges heat heat with the hot/cold piston atmospheres inside the engine
  - Connects to the input pipe
- _hotSideAtmosphere
  - hot piston
- _coldSideAtmosphere
  - cold piston

mixing simply equalizes two atmospheres  
here's what it does:
  1. fill internal atmos from the attached tank
  2. mix _hotSideAtmosphere and internal atmos
  3. mix _coldSideAtmosphere and internal atmos
  4. mix _hotInputAtmosphere and InputNetwork (the input pipe)
  5. transfer heat from _hotInputAtmosphere to _hotSideAtmosphere. The result is UNUSED.
  6. mix _hotInputAtmosphere and OutputNetwork (the output pipe)
  7. transfer energy from environment to _coldSideAtmosphere. The result is UNUSED.
  8. energyDelta = Abs(_hotSideAtmosphere.energy - _coldSideAtmosphere.energy) (note that nuking the sign here is WEIRD)
  9. _machineEnvironmentEfficiency = 1 // WTF WHY IS THIS ALWAYS 1!?!?!?
  10. _hotSideAtmosphere.Pressure - _coldSideAtmosphere.Pressure is used to calculate MachinePressureDifferentialEfficiency
      the best MachinePressureDifferentialEfficiency occurs if _hotSideAtmosphere is 3000 kPa higher than _coldSideAtmosphere
  11. _workingGasEfficiency comes from the can gas. You want LOW specific heat, so H2 is the best.
  12. EnergyAsPower = Min(MaxPower, energyDelta * _workingGasEfficiency * _machineEnvironmentEfficiency * MachinePressureDifferentialEfficiency);
  13. Remove EnergyAsPower joules of energy from _hotSideAtmosphere

so we care about maximizing three things:
1. energyDelta
   - make sure the hot side and the cold side are not remotely close to the same temperature. There is no limit on how good this can be.
2. _workingGasEfficiency
   - just use H2, donezo
1. MachinePressureDifferentialEfficiency
   - just make sure input is some "mystery value" higher pressure than output.
   - This "mystery value" is going to be greater than 3000 kPa in practice due to the gas mixing fuckery happening FIRST.

so to achieve this the INTENDED way:
- yeet hot gas from some source into the input side
- use the room to cool the cold side
  - cooling source?
- something to lower the pressure of the output net
  - the output pipe is still plenty hot, so just pump it back to the input with whatever compressor is the most power efficient (oh god why do I suspect it will be a fucking active vent)

note that the funny energyDelta Abs() WEIRDLY means we can run the thing in reverse to generate power. This has some WILD ramifications:
- something to lower the pressure of the output net
  - the output pipe is still plenty COLD (energy destruction helps), so just pump it back to the input as per normal
- use something to cool the input
- use something to heat the room
  - make sure that the internal atmos doesn't get too high pressure, or it will go boom
  - this actually does a delta-P check, so if the room is ALSO high pressure... then well, you're fine.


## Turbine Generator

Power generation caps at a 50.6625kPa difference but grids equalize at 101.325kPa per tick. This means you could potentially rate limit things for more power.

Hey I'm back after reverse-engineering, and the way pressure is equalized across turbine generators is broken. Specifically, `AtmosphereEqualizeBothWays()` does not actually equalize. It just blindly moves pressure from the high-pressure side to the low pressure side. This means in a closed system with only a high pressure and a low pressure side, it will just bounce the pressure back and forth.

This bugged function is used in three places:

| place                                  | input          | output         | pressurePerTick | maxMoles | scale            | ramification                           |
|----------------------------------------|----------------|----------------|-----------------|----------|------------------|----------------------------------------|
| `AtmosphericItem.LeakAir():`           | internal atmos | external atmos | 101.325         | none     | LeakRatio, max 1 | only used for suit leaks               |
| `Furnace.HandleGasInput():`            | gas output     | internal atmos | ???             | none     | 1                | none, because it is  direction-checked |
| `TurbineGenerator.OnAtmosphericTick()` | a grid         | the other grid | 101.325         | none     | 1                | infinite power                         |
