# Changelog

## 1.3.1 2023-08-04

### Fixed
- Fixed regression introduced by game changing all logical operators into bitwise operators, notably breaking `nand` and `nor`.

## 1.3.0 2023-07-29

### Changed

- The AC will now target 20C whenever the helmet is closed. This is to stop people from thinking they're dying.

## 1.2.0 2022-08-27

### Changed
- Raise maximum internal pressure limit to 607.95, which is the point where the suit alarm would start beeping incessantly.
- The AC will now target -10C whenever the helmet is closed. This is to reduce water consumption.
- Suit now overdrives pressure to flush as quickly as possible. This will reduce time spent in vacuum during an autoflush.

## 1.1.0 2022-08-06

### Added
- Safety check to make sure the suit is not pressurized beyond safe limits

### Changed
- Previously, the program would not run for two ticks while auto-closing your helmet. This is no longer the case, and the program now runs every tick.
- Oxygen levels are now kept as low as possible to minimize the moles metabolized

### Fixed
- Toxin detection now matches how the game actually checks for toxins
- Release toggling off no longer lags one tick behind filters starting

## 1.0.0 2022-08-04

### Added
- Initial functionality
