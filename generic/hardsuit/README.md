# Hardsuit Controller

Slap this into any old hardsuit and it'll manage things for you.

## Features

1. If your suit needs flushing, it will beep at  you while autoflushing.  An autoflush is slower, but more precise, than a manual flush. Feel free to manually flush too!
2. If the environment is appears unsafe, it closes and locks your helmet. Not all types of unsafe environment can be detected. Keeping your helmet closed is on you!
3. It will micromanage your filter, greatly prolonging its lifespan. Filtering will temporarily fuck with release and pressure settings to avoid dumping O2 to waste due to a stationeers bug.
4. It will turn the AC off if the external temperature is safe. Otherwise, it will set your AC to the most extreme but still safe value.
5. It will turn off air flow when your helmet is open.

## Assumptions

If any of these are false, the hardsuit controller program may behave in undesirable ways.

1. Your tank is 100% oxygen
2. You inhale oxygen
3. You exhale CO2
4. Your suit is loaded exclusively with CO2 filters
5. CO2 is not toxic in any concentration (the devs may change this soon)
6. O2 is not toxic in any concentration (the devs may change this soon)
7. You pressurize your breathable rooms between 1 and 2 atmospheres
8. A 1 atmosphere breathable room contains at least 22% oxygen
9. You would prefer a brief stint in vacuum to being poisoned

## Limitations

1. The hardsuit cannot read the external gas mix directly. It can only read external pressure and temperature.
2. When the hardsuit helmet is open, its gas ratio sensor doesn't work as expected. It appears that the helmet only mixes with the environment as it is closed. This makes it impossible to auto-close the helmet if the room is contaminated.
3. Logic cannot use the helmet's "flush" button, and its behavior cannot be replicated perfectly. The only alternative is to set internal pressure to 0 while filtering is on, but this is slower than a manual flush.
4. If the external temperature is hotter than the internal temperature, your suit will dump some unfiltered atmosphere to waste. This is due to a stationeers bug. Here's what's going on:

   1. The external environment heats the suit (and therefore causes pressure to increase above the setpoint)
   2. The suit dumps any overpressure to waste, but only if the filters are on.
   3. The suit's AC cools the suit (and therefore causes pressure to decrease below the setpoint)
   4. The suit refills missing pressure from the oxygen tank, but only if the release is on.

   As you can see, in hot environments this will cause you to churn gas. This can be mitigated by some silly code that does weird, contrived things to the internal pressure target and release setting when the filters are running.

   The developers think this bug is interesting and makes hot planets more difficult, so they are leaving it in (for now).

## Bugs

### The "Three Clicks" bug

For some reason my filtering stuff is happening across two ticks:

1. Filter turns on
2. Release turns off and target is set to 202 kPa
3. time passes
4. Filter turns off, release turns on, target set to 101 kPa

Things that are **not** causing this:
- `suit.Filtration = internalPressureHigh || flushing || filtering`

## Suit Trivia

### Helmet
- The light uses 5 W per tick, or 10 W per second from the equipped suit's battery.
- Flushing does not consume power, and works with no cell. This means if you're out of battery you can manually flush in order to not die. Just flush whenever you get oxygen low or too hot/cold.
- Normal helmets actually have all the same logic values as a hardsuit helmet, and can be controlled perfectly fine via d0.

### IC Power Cost
- The IC uses 2.5 W per tick, or 5 W per second.
- Does it still use power if the suit is not equipped?

### Air Release
- Air release does not consume power, and functions even when there is no cell installed.

### Filters

- Filters use 10 W per tick, or 20 W per second.
- This means for an IC to break even, it has to run your filters 75% of the time maximum.
- Filters will still run even if the suit is not occupied.

### Air Conditioner

- The air conditioner behaves as a heat pump in cooling mode, meaning it has more than 100% efficiency. It costs less than 1 joule to remove 1 joule of heat.
- The air conditioner heats electrically in heating mode, meaning it has exactly 100% efficiency. It costs 1 joule of battery to add 1 joule of heat.
- There is a very small power benefit to running the AC as close to the environment temperature as is safe.
  - Convection scales linearly with delta-T. As you can run your suit at 0-40C this means you have 40 degrees to play with.
  - Contrived example: if the environment is 80C, you'll use 50% less power with a 40C suit vs a 0C suit.
  - Real example: if the environment is 620C, you'll use 3% less power running at 40C vs 20C.
- However, your
- You want to turn the AC off if the external atmosphere is a survivable temperature.
- There's no benefit to toggling the AC on and off if the environment is not a survivable temperature.
- The AC does not run if the suit is not occupied.

### Gas Sensors

While the suit has its own sensors, they suck. This is because the helmet and suit have separate
atmospheres that mix over time. The helmet atmosphere is the actual important one that you breathe
from, but the filtering is done in the suit. This means as you're filtering the helmet always lags
behind the suit. What this means is you need to read the helmet gas ratios to actually know when
you can stop filtering.

### Debugging

You can set the temperature/pressure settings to out of range values. They will display the set value
even though functionally they are still clamped. This is useful, as you can abuse the settings as
debug displays.
