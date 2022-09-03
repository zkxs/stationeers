# Miscellaneous Notes

## Hydration (OUTDATED)
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
