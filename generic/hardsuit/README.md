# Hardsuit Controller

Slap this into any old hardsuit and it'll manage things for you. I think the feature list really sells it.

The minified ic10 code: [hardsuit.ic10](hardsuit.release.ic10). Copy/paste this into the game.

The source code before minification: [hardsuit.icX](hardsuit.icX). Look at this if you actually want to read the code.

Steam workshop: [runtime's Hardsuit Controller](https://steamcommunity.com/sharedfiles/filedetails/?id=2845816603)

## Features

Compared to an IC-less hardsuit:

1. ~50x more filter lifespan
2. ~1.5x longer oxygen lifespan
3. 11 watts less power usage
4. Only needs CO2 filters: bad atmosphere will be automatically flushed. The suit will beep while it is autoflushing.
5. Closes and locks your helmet in some (but not all) unsafe environments. See [limitations](#limitations).
6. Mitigates a Stationeers bug where hot environments cause your oxygen tank to gradually empty into waste.

## Assumptions

If any of these are false, the hardsuit controller program may behave in undesirable ways:

1. Your tank is 100% oxygen
2. You inhale oxygen (or nothing)
3. You exhale CO2 (or nothing)
4. Your suit is loaded exclusively with CO2 filters
5. CO2 is not toxic in any partial pressure or concentration (the devs may change this soon)
6. O2 is not toxic in any partial pressure or concentration (the devs may change this soon)
7. You pressurize your breathable rooms between 1 and 2 atmospheres
8. A 1 atmosphere breathable room contains at least 22% oxygen
9. You would prefer a brief stint in vacuum to being poisoned

## Limitations

1. The hardsuit cannot read the external gas mix directly. It can only read external pressure and temperature.
2. When the hardsuit helmet is open, its gas ratio sensor doesn't work as expected. It appears that the helmet only mixes with the environment as it is closed. This makes it impossible to auto-close the helmet if the room is contaminated.
3. Logic cannot use the helmet's "flush" button, and its behavior cannot be replicated perfectly. The only alternative is to set internal pressure to 0 while filtering is on, but this is slower than a manual flush.
4. If the external temperature is hotter than the internal temperature, your suit will dump some unfiltered atmosphere to waste. This is due to a Stationeers bug. Here's what's going on:

   1. The external environment heats the suit (and therefore causes pressure to increase above the setpoint)
   2. The suit dumps any overpressure to waste, but only if the filters are on.
   3. The suit's AC cools the suit (and therefore causes pressure to decrease below the setpoint)
   4. The suit refills missing pressure from the oxygen tank, but only if the release is on.

   As you can see, in hot environments this will cause you to churn gas. This can be mitigated by some silly code that does weird, contrived things to the internal pressure target and release setting when the filters are running.

   The developers think this bug is interesting and makes hot planets more difficult, so they are leaving it in (for now).

## Suit Trivia

### Breathing

Below an oxygen partial pressure of 24 kPa the moles of oxygen your consume are related linearly to the partial pressure. This means that if oxygen is kept at the absolute minimum of 16 kPa vs anything above 24 kPa, you will consume 1.5 times less oxygen. In other words, this IC will make your oxygen tank last ~1.5x longer than you're used to.

### Helmet

- The light uses 5 J per tick, or 10 W from the equipped suit's battery.
- Flushing does not consume power, and works with no cell. This means if you're out of battery you can manually flush in order to not die. Just flush whenever you get oxygen low or too hot/cold.
- Normal helmets actually have all the same logic values as a hardsuit helmet, and can be controlled perfectly fine via d0.

### IC Power Cost

- The IC uses 2.5 J per tick, or 5 W.
- IC still uses power even if the suit is not equipped, and cannot be turned of short of unplugging it.

### Air Release

- Air release does not consume power, and functions even when there is no cell installed.

### Filters

- Filters use 10 J per tick, or 20 W.
- This means for an IC to break even, it has to run your filters 75% of the time maximum. In practice a filter management IC will run ~2% of the time. Not only will your filters last 50 times longer, they use only ~4 W average. The ~16W filter power savings means the IC effectively *saves* 11 W to install.
- Filters will still run even if the suit is not occupied.

### Air Conditioner

- The air conditioner behaves as a heat pump in cooling mode, meaning it has more than 100% efficiency. It costs less than 1 joule to remove 1 joule of heat.
- The air conditioner heats electrically in heating mode, meaning it has exactly 100% efficiency. It costs 1 joule of battery to add 1 joule of heat.
- There is a very small power benefit to running the AC as close to the environment temperature as is safe.
  - Convection scales linearly with delta-T. As you can run your suit at 0-40C this means you have 40 degrees to play with.
  - Contrived example: if the environment is 80C, you'll use 50% less AC power with a 40C suit vs a 0C suit.
  - Real (vulcan) example: if the environment is 620C, you'll use 3% less AC power running at 40C vs 20C.
- You should turn the AC off if the external atmosphere is a survivable temperature.
- There's no benefit to toggling the AC on and off if the environment is not a survivable temperature.
- The AC does not run if the suit is not occupied.

### Gas Sensors

While the suit has its own sensors, they suck. This is because the helmet and suit have separate
atmospheres that mix over time. The helmet atmosphere is the actual important one that you breathe
from, but the filtering is done in the suit. This means as you're filtering the helmet always lags
behind the suit. Therefore, you need to read the gas ratios from the *helmet* to actually know when
you can stop filtering.

### Debugging

You can set the temperature/pressure settings to out of range values. They will display the set value
even though functionally they are still clamped. This is useful, as you can abuse the settings as
debug displays.
