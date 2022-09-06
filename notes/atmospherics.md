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

## Human Survivability

To not take damage:

- Temperature between -10C and 50C, or your lungs will burn.
- If total pressure is not between 20 kPa and 607.95 kPa you will take psychological damage from all the alarms and labored breathing sounds.
- Oxygen partial pressure of at least 16 kPa (with 100% healthy lungs. Unhealthy lungs need more O2.)
- At most 5 kPa partial pressure of NOS, or you'll be stunned.
- At most 1 kPa partial pressure of toxins (which include volatiles and pollutant) or your lungs will be poisoned.

To breathe optimally:

- Entity.cs
- You breathe more efficiently at lower oxygen partial pressures. Math time:
  - 0.0012 moles per breath, times an efficiency multiplier
  - MinimumOxygenPartialPressure = 16
  - efficiency = atmos.PartialPressureO2 / MinimumOxygenPartialPressure
    - bigger numbers are worse, because this is a mole consumption multiplier
    - so at 16 kPa of O2 your efficiency is 1
    - So the most efficient possible airmix would be 16 kPa O2 (plus at least 4 kPa filler to get to the 20 kPa pressure requirement)
  - so if we're breathing optimally we're doing 0.0012 moles per tick, or 0.0024 moles per second.

_entityMolePerBreath = 0.0012f

## Plant Survivability

To not take damage:

- At most 1 kPa partial pressure of toxins (which may vary from plant to plant, but typically includes volatiles and pollutant)
- at least 0.00003 moles of water available
- room temperature between 15C and 52.5C
- water temperature between 3C and 62.5C
- light doesn't matter
- CO2 pressure doesn't matter
- total pressure doesn't matter

For optimal growth:

- at least 0.00006 moles of water available
- room temperature between 30C and 40C
- at least 0.0012 moles of CO2 available
- fully lit
  - grow lamps and the powered hydroponics stations light fully
  - sunlight is based on solar scale of the world.
  - grow lamps take priority over the sun. This means on vulcan you will actually get a 1.5x growth multiplier if you turn *off* your growlamp in direct sunlight!
- water temperature doesn't matter
- total pressure doesn't matter (provided it contains enough CO2 for the plant to consume)
- toxin presence doesn't matter

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

## Sterling

Some variables:

- _hotInputAtmosphere
  - Atmosphere that exhanges heat heat with the hot/cold piston atmospheres inside the engine
  - Connects to the input pipe
- _hotSideAtmosphere
  - hot piston
- _coldSideAtmosphere
  - cold piston

Some values:

- Will explode at 11 MPa delta-P from internal tank to environment
- internal tank is 54L
- hot side is 5L
- cold side is 5L
- hotinput is 10L
- Max power output is 8 KJ/tick

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
   - make sure the input and the room are not remotely close to the same temperature.
   - Run the stirling at a higher working gas pressure
2. _workingGasEfficiency
   - just use H2, donezo
3. MachinePressureDifferentialEfficiency
   - make sure the input is much hotter than the room. This will eventually get capped.

Once the max power output of 8KJ/tick is reached, your only option to get more power from your heat is to run more stirlings in parallel.

## Turbine Generator

Power generation caps at a 50.6625kPa difference but grids equalize at 101.325kPa per tick. This means you could potentially rate limit things for more power.

Hey I'm back after reverse-engineering, and the way pressure is equalized across turbine generators is broken. Specifically, `AtmosphereEqualizeBothWays()` does not actually equalize. It just blindly moves pressure from the high-pressure side to the low pressure side. This means in a closed system with only a high pressure and a low pressure side, it will just bounce the pressure back and forth.

This bugged function is used in three places:

| place                                  | input          | output         | pressurePerTick | maxMoles | scale            | ramification                           |
|----------------------------------------|----------------|----------------|-----------------|----------|------------------|----------------------------------------|
| `AtmosphericItem.LeakAir():`           | internal atmos | external atmos | 101.325         | none     | LeakRatio, max 1 | only used for suit leaks               |
| `Furnace.HandleGasInput():`            | gas output     | internal atmos | ???             | none     | 1                | none, because it is  direction-checked |
| `TurbineGenerator.OnAtmosphericTick()` | a grid         | the other grid | 101.325         | none     | 1                | infinite power                         |
