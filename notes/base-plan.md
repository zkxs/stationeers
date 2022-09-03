# Base Planning

## Base Design

### Rooms

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
  - Maintenance tunnels
  - Mining area
  - Rocket silo

### Layout
- Walls
  - Outer walls MUST be frames
  - interior walls can be composite/padded
  - a 1 block void between the frames and interior walls in some places for crawlspace
  - interior floor divisions will be more wall, wall side down
- Substation
  - station batteries
  - large transformers
- Solar area
  - must be on the substation roof
  - solars cannot use transformers or we'd need stupid logic replication, so they're going to be all heavy cable
  - solars either need a little glass house OR to be heavy or they'll get fucked up
    - would have to use padded windows on Omega Vulcan
  - if the roof gets full, then stop making solars and do something more interesting
  - checkerboard that shit
    ```
    S.S.S.
    .S.S.S
    S.S.S.
    ```
- Wind turbines can be wherever
  - not too far from substation though
- Maint tunnels spanning the base
  - contains the main heavy power conduit
  - if we need more than 100kW then you get to run another fucking cable
  - wire net for room gas analyzers
- Lobby
  - suit storage units
  - tank fillers?
- Workshop
  - the 4 infinity lathes, each with a stacker on the output
  - bonus stacker
  - recycler -> centrifuge -> a ball pit
- Initial shitter shack
  - just large enough to grow some plants in, so maybe 2x2x1
  - all my gear can be outside
  - manual doors for the airlock
- Actual first room: workshop zone, 6x6
  1. roof: glass solar booth, uses cheapest windows and equalizes with outside via passive vents
  2. layer of frames
  3. top story: substation
  4. layer of walls
  5. workshop
  6. layer of walls
  7. maintenance zone
  8. frames
- Second room: Hydroponics Zone. Reserve part for a ladderwell/elevator shaft 8x6
  1. layer of frames
  2. top story: habitation area, freezer
  3. layer of walls
  4. hydroponics land 1
  5. layer of walls
  6. Maintenance zone
  7. Frames
- Third room: luxurious realm. Attach to the ladderwell, 8x8
  1. layer of frames
  2. hydroponics land 2, stairs up from hydroponics land 1
  3. layer of walls
  4. lobby
  5. layer of walls
  6. maintenance zone
  7. frames

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