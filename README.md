# lsm303agr-nv

The LSM303AGR is ST's eCompass module: a three-axis accelerometer and a
three-axis magnetometer in one package, each reached at its own I2C
address.  This package drives both halves and presents the part as
[devices-nv](https://novo-lang.org/packages/devices-nv)'s
`Accelerometer`, `Magnetometer` and `Identifiable`.

## What it is

An **accelerometer** measures acceleration along three axes.  At rest
it reads the acceleration of gravity, 1 g, on the axis that points up,
so it also tells which way the board is tilted.  The part measures
within a **full scale** of ±2, ±4, ±8 or ±16 g, and in its
high-resolution mode each axis is a 12-bit count, **left-justified** in
two bytes: the count sits in the top 12 bits.  One step is 0.98, 1.95,
3.9 or 11.72 mg at the four scales.  The driver answers milli-g.

A **magnetometer** measures the magnetic field along three axes.  The
Earth's field at the surface is between about 25 and 65 microtesla, and
iron or current near the part adds its own.  The part's step is fixed
at 1.5 milligauss, 150 nanotesla, and the driver answers nanotesla.
**Offset cancellation** takes the sensor's own offset out of each
sample, and the driver turns it on.

The accelerometer answers at I2C address 0x19 and the magnetometer at
0x1E.  Each half has an identification byte: **WHO_AM_I_A** is 0x33 and
**WHO_AM_I_M** is 0x40.  For a read of more than one register, bit 7 of
the register address makes the address advance after each byte.  Each
half raises a **data-ready** flag when a sample is complete, and can
drive it onto a pin: the accelerometer onto INT1, the magnetometer onto
its INT_MAG/DRDY pin.

The LSM303AGR replaced the LSM303DLHC on later boards, at the same two
addresses and with other registers behind them; lsm303dlhc-nv drives the
older part.

## Install

```
novo pkg add lsm303agr-nv
```

## Example

A board names the part in the `devices` block of its declaration, by the
accelerometer's address, as the STM32F3DISCOVERY's does:

```novo norun:fragment
    devices
        ecompass  Lsm303agr  from "lsm303agr-nv"  on i2c0  addr 0x19  irq accel_int1
```

and a program reads both halves through their traits:

```novo norun:needs-pkg
use dev_accel.{ Accelerometer }
use dev_mag.{ Magnetometer }
use lsm303agr

@tier(embedded)
fn main() [hw]
    let e = bsp.board.ecompass()
    // ±2 g at 100 samples a second; the magnetometer at 10.
    if e.begin()
        match e.accel_read()
            // Close to 1000 with the board flat and face up.
            Some(a) => hal.uart.write(str.from_int(a.z))
            None    => hal.uart.write("no sample")
        match e.mag_read()
            Some(m) => hal.uart.write(str.from_int(m.x))
            None    => hal.uart.write("no sample")
```

## What the package contains

| Module | What is in it |
| --- | --- |
| `lsm303agr` | `Lsm303agr`, the driver: `attach`, `begin`, `irq_level`, `mag_set_rate_hz`, the register access `accel_reg`, `set_accel_reg`, `mag_reg` and `set_mag_reg`, and devices-nv's `Accelerometer`, `Magnetometer` and `Identifiable`; and the same operations as functions over any `I2cBus`, with the conversions |

## How to choose an entry point

A program on a board uses `Lsm303agr`, which the board's device table
builds.  A program that holds its own bus, or a test with a double of
the part, calls the module's functions, which take the bus and the
wiring as arguments.

## The rules a user needs

1. `begin` must run before a reading: both halves leave power-on
   powered down or idle.  It answers `false` for a part whose
   identification bytes are not 0x33 and 0x40.
2. A reading is converted at the scale it was taken in: the driver reads
   the scale and the sample in one transaction.
3. The ±16 g step is 11.72 mg, not 16 g over 2^11.
4. The driver keeps the accelerometer in high-resolution mode; a rate it
   offers only in low-power mode, 1620 Hz, reads as 0.
5. `device_id` answers both identification bytes as one number,
   0x3340: WHO_AM_I_A in the high byte, WHO_AM_I_M in the low one.

## What is not included

- The FIFO, the click and free-fall detection, the interrupt
  generators' thresholds and the temperature sensor: no trait of
  devices-nv names them yet; the register access reaches them.
- The SPI interface: the boards that carry the part wire it to I2C.

## Related packages

- [devices-nv](https://novo-lang.org/packages/devices-nv) names the
  traits this driver implements, the wiring record and the register
  helpers it uses.
- [embedded-hal-nv](https://novo-lang.org/packages/embedded-hal-nv)
  names the `I2cBus` and `GpioOut` traits and the board handles.
- lsm303dlhc-nv drives the LSM303DLHC, the part this one replaced.
- l3gd20-nv drives the gyroscope beside it on the STM32F3DISCOVERY.

## Tests

`novo test tests` runs the module's functions over a double of both
halves' register files: the samples at each scale, the rates, the
identity, the bring-up and an address no part answers; and each
method of the type `Lsm303agr` over the host's board handles, which are
absent hardware that refuses every transfer.  The driver on hardware is
measured by the STM32F3DISCOVERY's suite in the novo-lang repository,
`orbit/bsp/st/stm32f3/stm32f3discovery/tests/imu.sh`, which reads gravity
and the Earth's field on the board and the coverage of this module
there.

## Licence

Apache-2.0.
