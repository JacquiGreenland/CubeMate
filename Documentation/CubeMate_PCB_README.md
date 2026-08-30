# CubeMate Controller PCB

## About this board

The CubeMate controller PCB is a custom two-axis controller designed specifically for CubeMate. It provides the electronics required to drive the right ascension (RA) and declination (DEC) axes using OnStep, while keeping the controller simple, modular and easy to repair or replace.

The design is intentionally based around readily available modules rather than permanently soldered controller or driver ICs. In the reference build, the ESP32-S3 and both TMC2209 stepper drivers are socketed so that any of them can be replaced without removing the complete PCB from service.

The board can be assembled with surface-mount parts, but through-hole alternatives are provided for the resistor and capacitors so that a fully hand-soldered build is also possible. This guide assumes that the builder is already comfortable with basic electronic assembly, component polarity, continuity checking and soldering.

This document covers the controller PCB itself. Mechanical installation of the completed PCB into CubeMate is covered by the main CubeMate assembly guide.

## Main components

A complete controller uses:

| Component | Qty | Notes |
| --- | ---: | --- |
| Waveshare ESP32-S3 Zero | 1 | Main OnStep controller. Some suppliers also describe this small-format board as an ESP32-S3 Mini. |
| TMC2209 V3.0 stepper driver module | 2 | One driver for RA and one for DEC. Socketing is recommended. |
| 100 uF, 35 V electrolytic capacitor | 2 | One on the motor-power supply line for each TMC2209. SMD footprint plus through-hole alternative pads are provided. |
| Mini 360 buck converter module | 1 | Regulates the incoming supply to 3.3 V for the ESP32-S3. |
| 1 kohm resistor | 1 | Part of the shared single-wire TMC2209 UART interface. SMD footprint plus through-hole alternative pads are provided. |
| JST-XH 4-pin board connector | 2 | One stepper-motor connector per axis. |
| Status LED | 1 | OnStep status indication. Mounted on the underside of the PCB. |
| 1 x 4 GPIO header | 1 | Exposes four otherwise-unused ESP32 GPIO connections for future expansion. |
| 5.5 x 2.1 mm horizontal PCB DC jack | 1 optional | Centre-positive in the reference build. Direct-wired power can be used instead. |
| USB-to-TTL serial module | 1 optional | Reference build uses a ZY-CP2102 module. The PCB can also accommodate a CH340E-based module. |
| Pin headers / sockets | As required | Used for replaceable modules and detachable panel wiring. |

Exact supplier links can be added to this document where useful.

## Board orientation

Almost all components are fitted to the **top/component side of the PCB**, which is the side **without the CubeMate logo**. This includes:

- the ESP32-S3 Zero;
- both TMC2209 drivers;
- the Mini 360 buck converter;
- the two 100 uF capacitors;
- the 1 kohm UART resistor;
- the JST-XH motor connectors;
- the optional serial module;
- the power connector or power wiring;
- the expansion headers.

The **status LED is the exception**. It is mounted on the **underside/logo side** of the PCB.

When the completed board is installed in CubeMate, the populated side faces down into the enclosure and the underside faces the top cover. A small hole in the top cover allows the status LED to be seen without exposing the observer to a bright direct LED.

## Power input

The reference PCB can accept power through an optional horizontal **5.5 x 2.1 mm centre-positive DC barrel jack**. The same PCB pads may instead be used for directly soldered wires.

Using the barrel jack makes the internal power lead easy to disconnect when the PCB or top plate needs to be removed. A directly wired arrangement is equally valid and can use any suitable inline connector if removability is required.

The Mini 360 regulator itself accepts approximately **4.75 to 23 V DC**. The reference CubeMate build is operated from a **19 V** supply, using a repurposed laptop power supply fitted with a standard 5.5 x 2.1 mm plug. A good-quality **12 V** supply with sufficient current is also suitable. In practice, a stable supply in roughly the **15 to 19 V** range is a good choice for the reference system.

The power supply must also be appropriately rated for the two stepper motors and their TMC2209 drivers.

### Important: set the Mini 360 before installing the ESP32

> **Do not install or power the ESP32-S3 Zero until the Mini 360 output has been adjusted and measured at 3.3 V.**

The Mini 360 is adjustable. Before fitting the ESP32-S3 Zero, power the board and use a multimeter to set and verify the regulator output at **3.3 V**.

Once the output has been confirmed, power the board off before installing the ESP32-S3.

## TMC2209 stepper drivers

CubeMate uses two **TMC2209 V3.0** modules, one for each axis.

### Socketing

Socketing the drivers is strongly recommended. The PCB can accept directly soldered modules, but sockets make a failed or damaged driver much easier to replace.

The same approach is recommended for the ESP32-S3 Zero.

### Driver orientation

The TMC2209 modules have a required orientation. The orientation is not explicitly labelled on the silkscreen, but the footprint is visually keyed.

At one end of the TMC2209 module, the normal pin rail turns the corner and includes **two additional pins across the end of the module**. The corresponding pair of holes is present in the PCB footprint. Match this geometry before inserting the driver into its socket.

If a driver is soldered directly to the PCB, the complete footprint geometry makes the intended orientation apparent, but orientation should still be checked before soldering.

### Current / Vref adjustment

> **Do not assume that the default TMC2209 current setting is correct for your motors.**

Each TMC2209 must be adjusted for the stepper motor with which it is being used. Use the onboard potentiometer and the normal TMC2209 Vref/current-setting procedure for the particular driver module and motor.

There is no single correct setting for every CubeMate build because it depends on the motor specification.

**TMC2209 current-setting calculator/reference:** [LINK REQUIRED]

Incorrect driver adjustment can produce surprisingly erratic motor behaviour, so this should be treated as a normal part of commissioning the controller.

## TMC2209 UART interface

The two TMC2209 drivers use a **shared, addressable single-wire UART bus**. Both drivers connect to the same ESP32 UART/GPIO bus line and are distinguished by their configured TMC2209 addresses.

The PCB implements the normal one-wire TMC2209 UART arrangement, including the **1 kohm resistor in the ESP32 RX/TX UART path**.

Driver addressing and the corresponding GPIO assignment are defined by the OnStep configuration supplied with the CubeMate controller.

## Stepper motor connectors

Each axis uses a **4-pin JST-XH connector** wired directly to its TMC2209 motor outputs.

### Motor plug pin order

Viewing the **JST-XH plug from the top**, with the locating boss facing **downward**, the connection order from left to right is:

```text
2B   2A   1A   1B
```

On many common four-wire stepper motors this corresponds to:

```text
Black   Green   Blue   Red
```

Wire colours are **not universal**. Always confirm the two motor windings and the manufacturer's wiring information if the motor uses a different colour scheme.

## External serial interface

The ESP32-S3 Zero already has an onboard USB-C interface, and Wi-Fi control is also available through OnStep. However, the reference CubeMate design provides a separate USB-to-TTL interface for reliable wired serial control from a Raspberry Pi, PC or similar host.

This was added because the onboard ESP32-S3 Zero USB-C serial interface has not proved sufficiently reliable for routine serial use in the reference build.

The PCB is designed to accommodate two styles of serial module:

- a CH340E-based USB-to-TTL module with an onboard micro-USB connector; or
- the **ZY-CP2102 USB-TTL serial UART module** used in the reference CubeMate build.

The reference instructions below describe the ZY-CP2102 arrangement.

### TTL-side connections

CubeMate uses only:

```text
GND
TX
RX
```

on the TTL side of the serial module.

The module fits directly into the PCB position, so the required TX/RX routing is already handled by the PCB layout.

### ZY-CP2102 voltage-selector pads

The reference ZY-CP2102 module has solder selectors for **5 V** and **3.3 V** operation.

For the CubeMate reference build, **both selector pads are left unbridged and cleaned of solder**.

The serial module is **not powered from the CubeMate controller PCB**. It becomes powered only when its USB side is connected to a host such as a PC or Raspberry Pi.

This arrangement avoids feeding host USB power into the main controller power system through the serial module.

### USB-side connections

The USB side of the CP2102 module presents:

```text
VCC
GND
D+
D-
```

These are connected to the panel-mounted USB fly lead according to the wiring of the particular panel connector or fly lead selected for the build.

Do not assume that every USB fly lead uses the same wire colours. Verify its pinout before soldering.

### Mounting the serial module

The CP2102 module can be mounted in either of two ways:

1. **Surface mounted** using the castellated edges of the module and the extended pads on the CubeMate PCB.
2. **Through-hole mounted** using pin headers or a similar removable connection.

The PCB also includes a second set of holes beside the CP2102 position. These holes are intentionally **not electrically connected**. They can be used to pass a pin header through the PCB and mechanically anchor the serial-module / USB-fly-lead connection.

In the reference build, this arrangement provides both mechanical support and a removable connector for the USB lead from the CubeMate top panel.

## Status LED

The LED is intended as the **OnStep status LED**.

Its GPIO assignment and behaviour are controlled by the supplied OnStep pin map and firmware configuration, rather than being fixed by this document. A builder modifying the firmware may therefore repurpose the LED if desired.

The LED is mounted on the underside/logo side of the PCB so that it faces the viewing hole in the CubeMate top cover when the PCB is installed.

## Expansion GPIO header

Four otherwise-unused ESP32 GPIO connections are exposed on a small header for future expansion.

They can potentially be used for features such as:

- switches or limit sensors;
- additional status indications;
- environmental sensors;
- other experimental or future CubeMate functions.

Check the supplied OnStep pin map and firmware configuration before assigning any of these pins to new hardware.

## Surface-mount and through-hole options

The reference PCB uses surface-mount versions of the capacitors and resistor, but matching **through-hole pads are also provided**.

This means the board does not require a hot-air station or hot plate to build. A builder who prefers conventional hand soldering can populate the alternative through-hole footprints instead.

The larger components - including the JST-XH connectors, sockets, headers, optional barrel jack and Mini 360 module - are conventional through-hole/module-mounted parts.

The serial module can likewise be surface mounted using its castellated pads or fitted using through-hole headers.

## Suggested assembly approach

There is no mandatory soldering sequence for the CubeMate PCB.

### If using surface-mount components

A convenient sequence is:

1. Populate and reflow the surface-mount components first.
2. Fit sockets for the ESP32-S3 and TMC2209 modules.
3. Fit the Mini 360 and any remaining module headers.
4. Fit the JST-XH connectors, power connector and other larger through-hole parts.
5. Fit the underside status LED.
6. Add the serial-module mounting and USB fly-lead connection in the preferred form.

This is only a practical sequence, not an electrical requirement.

### If using through-hole components

If the alternative through-hole capacitor and resistor pads are being used, the parts can simply be fitted in whatever order is most convenient. There is no particularly critical assembly sequence.

## Recommended commissioning checks

Before installing the controller in CubeMate:

1. Inspect the PCB carefully for solder bridges, poor joints and incorrect component orientation.
2. Check continuity and verify that the power rails are not shorted.
3. Leave the ESP32-S3 Zero out of its socket.
4. Apply power and adjust the Mini 360 output to **3.3 V**.
5. Power off and install the ESP32-S3 Zero.
6. Confirm that both TMC2209 modules are installed in the correct orientation.
7. Set each TMC2209 Vref/current limit for its particular stepper motor using the normal TMC2209 procedure.
8. Confirm the JST-XH motor wiring before connecting the steppers.
9. If using the reference ZY-CP2102 module, confirm that the 5 V and 3.3 V selector pads are both unbridged.
10. Verify the USB fly-lead pinout before connecting it to the CP2102 module.
11. Load/configure OnStep using the supplied CubeMate pin map and settings.
12. Test both axes at low speed before carrying out normal slews.

## Replaceability and serviceability

One of the main design goals of the CubeMate PCB is that the controller should remain easy to repair and upgrade.

Where practical, the reference build therefore uses:

- socketed TMC2209 drivers;
- a socketed ESP32-S3 Zero;
- removable JST-XH motor connectors;
- a detachable power connection;
- a detachable USB fly lead;
- a modular USB-to-TTL interface.

This allows most likely points of failure - or future upgraded controller modules - to be replaced without rebuilding the complete electronics assembly.

---

## References still to add

- [LINK REQUIRED] Waveshare ESP32-S3 Zero reference / supplier
- [LINK REQUIRED] TMC2209 V3.0 module reference / supplier
- [LINK REQUIRED] Mini 360 buck converter reference
- [LINK REQUIRED] ZY-CP2102 module reference / supplier
- [LINK REQUIRED] TMC2209 Vref/current calculator or setup reference
- [LINK REQUIRED] CubeMate OnStep pin map / configuration
