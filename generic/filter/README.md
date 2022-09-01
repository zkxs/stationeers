# Distro Filtration Controller

A filtration unit controller designed for maintaining a distro mix.

The code: [filter.ic10](filter.ic10)

## Usage

You must define your minimum and maximum partial pressures for each of the six gasses.

While a gas's partial pressure is above the corresponding `MaxPartialPressure`, the filtration unit will be turned on.

While a gas's partial pressure is below the corresponding `MinPartialPressure`, a pump will be turned on. You do not need to bind a pump to a device port if `MinPartialPressure` is 0.

| gas | filter | pump |
| -- | -- | -- |
| gas 1 | d0 | d3 |
| gas 2 | d1 | d4 |
| gas 3 | d2 | d5 |

The program will automatically figure out what filter is doing which gas.
