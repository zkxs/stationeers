# Radiator Manager

Oh god this is so much worse than you'd think.

## Truly Cursed Solar Angles

The daylight sensor has three modes: Default, Horizontal, and Vertical. Default and Vertical are cursed because they only give positive numbers out for vertical. Horizontal is cursed because it's complicated. Empirical measurement time...

- I'm on the moon
- The sun rises in the east (90 degrees).

### Test 1

Daylight sensor in Horizontal mode goes on an east-facing wall, with the data port facing north.

| Event                           | Time | Horizontal | Vertical |
| ------------------------------- | ---- | ---------- | -------- |
| Sun directly beneath the ground | 0000 | -90        | 90       |
| Sun rising over east horizon    | 0900 | -90 -> +90 | 0        |
| Sun directly overhead           | 1200 | +90        | 90       |
| Sun setting over west horizon   | 1800 | +90 -> -90 | 180      |

### Test 2

Daylight sensor in Horizontal mode goes on an east-facing wall, with the data port pointing up.

| Event                           | Time | Horizontal | Vertical |
| ------------------------------- | ---- | ---------- | -------- |
| Sun directly beneath the ground | 0000 | 0          | 90       |
| Sun rising over east horizon    | 0900 | 0 -> -180  | 0        |
| Sun directly overhead           | 1200 | -180       | 90       |
| Sun setting over west horizon   | 1800 | -180 -> 0  | 180      |

### Test 3

Daylight sensor in Horizontal mode goes on south-facing wall, with the data port pointing up.

| Event                           | Time | Horizontal   | Vertical |
| ------------------------------- | ---- | ------------ | -------- |
| Sun directly beneath the ground | 0000 | 0            | 90       |
| Sun rising over east horizon    | 0900 | -90          | 90       |
| Sun directly overhead           | 1200 | -180 -> +180 | 90       |
| Sun setting over west horizon   | 1800 | 90           | 90       |

### Test 4

Daylight sensor in Horizontal mode goes on south-facing wall, with the data port pointing down.

| Event                           | Time | Horizontal   | Vertical |
| ------------------------------- | ---- | ------------ | -------- |
| Sun directly beneath the ground | 0000 | ±180         | 90       |
| Sun rising over east horizon    | 0900 | +90          | 90       |
| Sun directly overhead           | 1200 | 0            | 90       |
| Sun setting over west horizon   | 1800 | -90          | 90       |

### Test 4

Daylight sensor in Horizontal mode goes on south-facing wall, with the data port facing east.

| Event                           | Time | Horizontal   | Vertical |
| ------------------------------- | ---- | ------------ | -------- |
| Sun directly beneath the ground | 0000 | -90          | 90       |
| Sun rising over east horizon    | 0900 | ±180         | 90       |
| Sun directly overhead           | 1200 | +90          | 90       |
| Sun setting over west horizon   | 1800 | 0            | 90       |

I think I like these numbers.

## Recording Radiator Behavior

All measurements are in the **Test 4** coordinate frame.

Cases where the radiators are performing solar heating:

- -178 to 172 *weird morning spike*
- 60 to -2 *evening*

## What do?

| Can Solar Heat | Pipe Needs Cooling | Open radiator | Add 90 to angle |
| -------------- | ------------------ | ------------- | --------------- |
| false          | false              | false         | false           |
| false          | true               | true          | true            |
| true           | false              | true          | false           |
| true           | true               | true          | true            |
