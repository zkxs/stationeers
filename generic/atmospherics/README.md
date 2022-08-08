# Atmospherics

All the plans and software needed to spin up an atmospheric system.

## The Plan

- Main atmospherics area
  - Central waste processing for all your mystery gas
    - takes furnace output
    - takes ground ice
    - will be filtered via IC to prevent fuel mix from building up
    - if waste is very hot we can run sterlings off of it
  - Tanks for all six gases and all one liquids
    - These tanks do *not* need to be the same temperature
      - We can use math to mix properly across temperature differences. [For example, this](../mixer/).
      - If we do decide to equalize temps, we can just use heat exchangers.
  - Everything outdoors should be insulated so the pesky environment doesn't fuck with your temperature. I want complete future-proofed control over temperature without having to go re-pipe everything.
  - Do not use your main CO2/pollutant tank for coolant, as the fucking air conditioners can contaminate your coolant network.
- Distro
  - Airmix
    - Humans
      - \>20kPa - 607.95kPa total pressure.
      - \>16kPa O2.
      - <0.5kPa toxins.
      - <5kPa N2O
    - Plants
      - \> 7.7 kPa total
      - Trace CO2. It can be a small fraction of a mole. Maybe 0.0012 moles?
      - < 1 kPa of toxins. This is typically pollutant, but varies from plant to plant.
      - Environment temperature between 20-50C to grow. 30-40C is ideal.
      - Water temperature between 5-60C
      - ~0.4 moles of water per hour
    - Structures
      - If you want your base higher than 200 kPa relative pressure you cannot use
        - composite window
        - glass doors
        - iron walls
        - iron windows
      - If you want your base higher than 300 kPa relative pressure you cannot use
        - composite doors
        - composite walls
        - padded walls
        - padded windows
      - If you want your base higher than 500 kPa relative pressure you cannot use
        - manual hatch
      - If you want your base higher than 1 MPa relative pressure you cannot use
        - Airlock doors
        - Reinforced windows
      - If you want your base higher than 15 MPa canisters will be crushed
      - These are safe at any pressure
        - Blast doors
        - Iron frames
        - Steel frames
    - Conclusion
      - Base pressure should be within 200 kPa of environment pressure to allow for all structures.
        - Airlocks pull a vacuum, so they will require airlock doors on high pressure planets (Venus).
      - Temperature should be 35C (for plants)
      - Airmix should be 20 kPa O2, 0.5 kPa CO2, rest dilutant (N2).
  - One pipenet, connected to rooms via passive vents.
    - This allows easy pressure control without using logic on one billion active vents (each of which would cost 100W regardless of delta-P)
    - Might need a holding tank if it's needing constant filtering.
    - If *anything* breaks your whole base will drain. Doors won't help you. Perhaps add digital valves? Each one costs 25 Watts to keep open. But how to detect a breach?
  - The distro pipe will be maintained directly. This reduces expensive temperature correction we'd get from using holding tanks constantly.
    - Filters will engage to remove any gas that is in excess
    - Pumps will engage to add any gas that is lacking
    - AC will regulate distro temperature directly
- Walk-in fridge
  - Only needs N2, so we can use a single filter to remove everything else
  - One pump to add more N2 from supply if necessary
  - Separate juiced-up AC system to get as close to 0K as possible
    - the APC needs to be *outside* the fridge due to battery loss below 0C.

## ICs

### Distro Mix Controller

Manages O2, N2, CO2 levels. Each gas needs 1 filter and 1 pump, so that's all 6 device ports. The gas analyzer will be read via `lb`.

### Distro Toxin Controller

Manages H2, N20, X levels. Each gas needs 1 filter, so that's 3 device ports. We might as well still use `lb` for the gas analyzer.

With the remaining 3 ports we can do something nice, like a low filter alert. This might require using `db.Setting` from the distro mix controller...

### Distro Temperature Controller

Do we even need an IC for this? An AC should just be able to handle it.

### Waste Filter Controller

- All six device ports will be taken by the six filters
- If any gas's partial pressure exceeds an upper threshold, then filter it until it passes a lower threshold.

### Holding Tank Temperature Controller

- We don't need to be precise with this, so a direct AC is unnecessary.
- Due to a bunch of insulation we expect waste to get *hotter* over time, from machine and furnace output. Therefore we can just toggle a bunch of heat exchangers on and off with a digital valve if waste gets too hot.