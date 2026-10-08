# Changelog

## 0.1.0 (2026-10-08)

The first release: a driver for ST's LSM303AGR, a three-axis
accelerometer and a three-axis magnetometer in one package, on an I2C
bus.

- `Lsm303agr`, the type a board's device table builds from the
  accelerometer's address, implementing devices-nv's `Accelerometer`,
  `Magnetometer` and `Identifiable` (identity 0x3340, both WHO_AM_I
  bytes), with its own methods for the bring-up, the magnetometer's
  rate, the INT1 line's level and register access on both halves.
- The register access as functions over any `I2cBus`, which the type's
  methods call.
- The host suite runs the functions over a double of both halves'
  register files and the type's methods over the host's board handles.
