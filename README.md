# BMW F1x 6WA Instrument Cluster Controller

Control a physical BMW F1x instrument cluster using an **Arduino Uno**, a **CAN Bus Shield**, and live telemetry from **BeamNG.drive**.

The project receives vehicle data from BeamNG.drive through OutGauge, forwards it to an Arduino over USB, and converts the data into BMW F Series CAN messages understood by a physical BMW instrument cluster.

The project was primarily developed and tested with a **BMW F10/F11 6WA instrument cluster**.

---

## Features

### BeamNG.drive integration

Live telemetry from BeamNG.drive can be displayed on the physical cluster, including:

- Vehicle speed
- Engine RPM
- Fuel level
- Oil temperature
- Coolant temperature
- Throttle position
- Gear state
- Turn indicators
- High beam
- Handbrake
- ABS
- Traction control
- Battery warning
- Selected warning indicators
  
---

# Hardware requirements

## Required Parts

You will need:

- Arduino Uno
- Seeed Studio CAN BUS Shield V2
- BMW 6WA instrument cluster
- 12 V DC power supply
- USB cable for the Arduino
- Jumper wires
  
Recommended:

- Laboratory power supply with current limit
- Proper BMW instrument cluster pigtail

---

# BMW 6WA instrument cluster wiring

The following wiring has been used with the F10/F11 6WA cluster in this project.

> **Important:** Verify the pinout of your specific cluster before applying power. BMW cluster revisions and connectors may differ.

| Cluster Pin | Function | Connection |
|---|---|---|
| Pin 1 | +12 V | Power supply +12 V |
| Pin 2 | +12 V | Power supply +12 V |
| Pin 3 | -- | -- |
| Pin 4 | Temperature sensor signal | Outside temperature sensor |
| Pin 5 | Temperature sensor signal | Outside temperature sensor |
| Pin 6 | CAN High | CAN High on CAN Bus Shield |
| Pin 7 | Ground | Power supply Ground |
| Pin 8 | Ground | Power supply Ground |
| Pin 9 | -- | -- |
| Pin 10 | -- | -- |
| Pin 11 | Wake / Terminal / 15WUP | Power supply +12 V |
| Pin 12 | CAN Low | CAN Low on CAN Bus Shield |

Basic connection:

```text
BMW 6WA             CAN Bus Shield / Power supply

Pin 1  -------------------- +12 V
Pin 2  -------------------- +12 V
Pin 7  -------------------- GND
Pin 8  -------------------- GND
Pin 11 -------------------- +12 V

Pin 6  -------------------- CAN H
Pin 12 -------------------- CAN L
```

# CAN Bus

The BMW F Series instrument cluster communicates at:

```text
500 kbit/s
```

The current project uses:

```text
Arduino Uno
+
Seeed Studio CAN Bus Shield V2
```

The MCP2515 chip (present on the Seeed Studio CAN Bus Shield V2) pin used by the firmware is:

```cpp
CS = D10
```

If your CAN shield can select between D9 and D10 for chip select, configure it for **D10**.

---

# Arduino connection

Connect the CAN shield to the Arduino Uno.

Then connect:

```text
CAN H -> instrument cluster pin 6
CAN L -> instrument cluster pin 12
GND   -> shared ground
```

Connect the Arduino to the computer through USB.

The Python application communicates with the Arduino at:

```text
115200 baud
```

---

# Software requirements

## Python

Python 3 is required.

Install the Python dependency:

```bash
py -m pip install pyserial
```

On Linux/macOS:

```bash
python3 -m pip install pyserial
```

---

# Arduino firmware

Open the Arduino firmware in the Arduino IDE.

Select:

```text
Board:
Arduino Uno
```

Select the correct serial port and upload the firmware.

The current firmware identifies itself as:

```text
F_Series_6WA_Universal_Firmware
READY FW=BMW6WA_EXTENDED_V7 PROTO=7
```

The firmware handles:

- CAN initialization
- BMW CAN frame generation
- Rolling counters
- CRC calculations
- Speed transmission
- RPM transmission
- Fuel level
- Temperature data
- Gear state
- Warning indicators
- Lighting
- Drive mode
- Check-Control
- Cruise / ACC frames
- Raw CAN messages
- CAN diagnostics

---

# Running the Python software

Start the software:

```bash
py F_Series_6WA_Controller.py
```

---

# Connecting the Arduino

1. Connect the Arduino through USB.
2. Start the Python application.
3. Select the Arduino COM port.
4. Click:

```text
Arduino verbinden
```

The software requests the firmware version automatically.

A compatible firmware should respond with something similar to:

```text
FW BMW6WA_EXTENDED_V7 PROTO=7
```

---

# BeamNG.drive setup

The project receives telemetry using **OutGauge**.

Default configuration:

```text
IP:
127.0.0.1

UDP Port:
4444
```

The Python software listens on:

```text
127.0.0.1:4444
```

Configure BeamNG.drive to send OutGauge packets to that address.

Then click:

```text
Start listening
```

When packets are received, the GUI displays live values such as:

```text
Speed
RPM
Gear
Coolant temperature
Oil temperature
Fuel
Throttle
Lights
```
---

# Safety

## Do not connect 12 V to Arduino pins!

The 6WA instrument cluster uses 12 V power.

Arduino GPIO pins operate at 5 V.

Never connect:

```text
12 V -> Arduino I/O
12 V -> CAN-H
12 V -> CAN-L
```

Doing so can permanently damage the Arduino, CAN Bus shield, or instrument cluster.

---

# Disclaimer

This project is intended for educational, simulation, development, and benchtesting purposes.

BMW is a trademark of BMW AG.
This project is not affiliated with, endorsed by, or sponsored by BMW AG.

Use this software and hardware setup at your own risk.
Incorrect wiring, incorrect CAN messages, or improper power connections can damage electronic equipment.
Do not use this project to alter vehicle identification data, stored odometer values, or other legally relevant vehicle information.
