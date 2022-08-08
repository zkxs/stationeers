# Miscellaneous Notes

## Netstat

```
tcp        0      0 *:27500                 *:*                     LISTEN      steamcmd   194013734   15839/rocketstation
tcp        0      0 *:55344                 *:*                     LISTEN      steamcmd   194014639   15839/rocketstation
udp        0      0 *:27015                 *:*                                 steamcmd   194011948   15839/rocketstation
udp        0      0 *:54390                 *:*                                 steamcmd   194014636   15839/rocketstation
udp6       0      0 [::]:27500              [::]:*                              steamcmd   194010997   15839/rocketstation
```

## Connection
```
172.22.0.10:27500
example.com:27500
login foo
```

## Jetpack
- It emits moles of gas, therefore you might as well supercool your canister
- Any gas will do, so use whatever is plentiful and nontoxic

## Oxygen
- Your suit AC is going to have to fix whatever temperature your oxygen tank is, so try to get it to around your suit's target temperature. It's not worth it to try and maximize moles by cooling it.

## Constants
| Constant                    | Value                              |
|-----------------------------|------------------------------------|
| Idea gas constant           | 8.31439971923828 (kPa*L)/(J\*mole) |
| One atmosphere              | 101.325 kPa                        |
| Zero Celsius                | 273.15 K                           |
| Armstrong limit             | 6.3 kPa                            |

## Breathing
- You need 20 kPa to not get hurt
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

## Base Design
- Mandatory
  - Furnace room
  - Workshop
  - Plant room
  - Atmos area
- Optional
  - Crew quarters
  - Kitchen
  - Garage
  - Control room
  - Observation area
  - Comms tower

## Hydration
- You take 0.1 dehydration damage while your hydration is 0
- You heal 0.1 dehydration damage while your hydration is nonzero
- Dehydration damage is constant and does not scale
- Hydration usage scales on several things
  - World hunger rate (slider in options)
  - World dehydration rate (0.1, 0.2, 0.35 for easy, normal, stationeer respectively)
  - Temperature
    - Hydration drop scales linearly from temperature.
    - Okay this is interesting, as it's not being clamped to a positive number.
    - Naturally, we want to know where the zeros are. For easy, normal, stationeer it is -10C, -20C, -35C respectively.
    - So if you go below those temperatures you'll start *gaining* hydration.
    - Note that once you turn it negative you actually want a larger hunger slider as it will multiply your bugged hydration gain.
    - This *will* burn your lungs with linearly scaling damage starting at -10C.
- This means if you want to minmax your water, only be hydrated when you are as cold as possible.

## Venus Plan

- We're going to play on "stationeer" difficulty, because difficulty is the point.

### The Bear Necessities

#### Drinking Water

1. starting equipment tank
2. trading
3. composter?

#### Safe environment to eat/drink

1. Make a vacuum chamber
   1. must not use walls or they'll break
   2. how to get in?
      1. just don't use glass doors or iron walls and you'll be fine
2. Pressurize the chamber above 20kPa with something nontoxic
3. Cool the chamber below 80C

#### Oxygen

1. work around suit O2 dumping bug
2. plants (CO2 is plentiful)


2. pull vacuum
   1. you can briefly open helmet in vacuum to eat/drink
3. water bottle filling
4. power
   1. solars are the same as moon
5. smelt steel
6. Obtain hydrogen

What is `public List<SpawnGas> ExspelledGas;` in AdvancedComposter?

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


## turbine generator
Power generation caps at a 50.6625kPa difference but grids equalize at 101.325kPa per tick. This means you could potentially rate limit things for more power.

Hey I'm back after reverse-engineering, and the way pressure is equalized across turbine generators is broken. Specifically, `AtmosphereEqualizeBothWays()` does not actually equalize. It just blindly moves pressure from the high-pressure side to the low pressure side. This means in a closed system with only a high pressure and a low pressure side, it will just bounce the pressure back and forth.

This bugged function is used in three places:

| place                                  | input          | output         | pressurePerTick | maxMoles | scale            | ramification                           |
|----------------------------------------|----------------|----------------|-----------------|----------|------------------|----------------------------------------|
| `AtmosphericItem.LeakAir():`           | internal atmos | external atmos | 101.325         | none     | LeakRatio, max 1 | only used for suit leaks               |
| `Furnace.HandleGasInput():`            | gas output     | internal atmos | ???             | none     | 1                | none, because it is  direction-checked |
| `TurbineGenerator.OnAtmosphericTick()` | a grid         | the other grid | 101.325         | none     | 1                | infinite power                         |
