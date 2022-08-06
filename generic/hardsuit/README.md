# Hardsuit Controller

Slap this into any old hardsuit and it'll manage things for you.

## Features

1. If your suit needs flushing, it will beep at  you while autoflushing.  An autoflush is slower, but more precise, than a manual flush. Feel free to manually flush too!
2. If the environment is appears unsafe, it closes and locks your helmet. Not all types of unsafe environment can be detected. Keeping your helmet closed is on you!
3. It will micromanage your filter, greatly prolonging its lifespan. Filtering will temporarily fuck with release and pressure settings to avoid dumping O2 to waste due to a stationeers bug.
4. It will turn the AC off if the external temperature is safe.  Otherwise, it will set your AC to the most extreme but still safe value.
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

