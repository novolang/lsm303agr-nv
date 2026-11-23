# Changelog

## 0.1.3 — 2026-11-23

The package asks for embedded-hal-nv ^0.4.0 (was ^0.2.1) and devices-nv ^0.4.0 (was ^0.3.2), the minor of both trait packages that the other drivers and the board packages ask for, so one version of each serves a program that takes this package beside any of them.  Under the older ranges, a program that named this package and a driver on 0.4 stopped with "no version of 'embedded-hal-nv' satisfies every requirement".

No signature of the package changed; the suite passes without an edit.

The STM32F3DISCOVERY's `tests/imu_probe.nv`, which drives this driver and l3gd20-nv through the board's devices block, builds against the package: flash 39,076 B before and 40,944 B after, the growth being devices-nv 0.4.0's traits in the probe's image.

## 0.1.2 — 2026-10-28

The dependency requirement is raised: `devices-nv` from ^0.2.0 to ^0.3.2.  The newest version of each builds with no migration warning under the 0.22.0 compiler, and the older one drew E2050, E2052 or E2055 warnings into this package's build.  The tests and the magnetometer wiring carry the new `en` field of `DeviceWiring`, which devices-nv 0.3 added.  No signature and no behaviour changed.

## 0.1.1 — 2026-10-25

A call that passes a variable to a `var` parameter now writes the `var` marker, which the 0.21.1 compiler asks for (E2050).  The package needs Novo 0.21.1 or later.  No signature and no behaviour changed.

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
