# BMW F1x 6WA Instrument Cluster Controller for BeamNG.drive

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

# Hardware

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

# BMW 6WA Wiring

The following wiring has been used with the F10/F11 6WA cluster in this project.

> **Important:** Verify the pinout of your specific cluster before applying power. BMW cluster revisions and connectors may differ.

| Cluster Pin | Function | Connection |
|---|---|---|
| Pin 1 | +12 V | Power supply +12 V |
| Pin 2 | +12 V | Power supply +12 V |
| Pin 3 | -- | -- |
| Pin 6 | CAN High | CAN High on CAN Bus Shield |
| Pin 7 | Ground | Power supply Ground |
| Pin 8 | Ground | Power supply Ground |
| Pin 11 | Wake / Terminal / 15WUP | Power supply +12 V |
| Pin 12 | CAN Low | CAN Low on CAN Bus Shield |

Basic connection:

```text
BMW 6WA                     CAN Shield / Power Supply

Pin 1  -------------------- +12 V
Pin 2  -------------------- +12 V
Pin 7  -------------------- GND
Pin 8  -------------------- GND
Pin 11 -------------------- +12 V

Pin 6  -------------------- CAN H
Pin 12 -------------------- CAN L
```

# CAN Bus

The BMW F-Series instrument cluster communicates at:

```text
500 kbit/s
```

The current project uses:

```text
Arduino Uno
+
Seeed Studio CAN Bus Shield V2
```

The MCP2515 chip select pin used by the firmware is:

```cpp
CS = D10
```

If your CAN shield can select between D9 and D10 for chip select, configure it for **D10**.

---

# Arduino Connection

Connect the CAN shield to the Arduino Uno.

Then connect:

```text
CAN H -> BMW cluster Pin 6
CAN L -> BMW cluster Pin 12
GND   -> shared ground
```

Connect the Arduino to the computer through USB.

The Python application communicates with the Arduino at:

```text
115200 baud
```

---

# Software Requirements

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

# Arduino Firmware

Open the Arduino firmware in the Arduino IDE.

Select:

```text
Board:
Arduino Uno
```

Select the correct serial port and upload the firmware.

The current firmware identifies itself as:

```text
BMW Cluster CAN Controller EXTENDED V7
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

# Running the Python GUI

Start the GUI:

```bash
py BMW_6WA_BeamNG_GUI_EXTENDED_V7_MODERN.py
```

The GUI is organized into several tabs:

```text
Dashboard
Controls
CAN Lab
Presets
Diagnose
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

The GUI requests the firmware version automatically.

A compatible firmware should respond with something similar to:

```text
FW BMW6WA_EXTENDED_V7 PROTO=7
```

---

# BeamNG.drive Setup

The project receives telemetry using **OutGauge**.

Default configuration:

```text
IP:
127.0.0.1

UDP Port:
4444
```

The Python application listens on:

```text
127.0.0.1:4444
```

Configure BeamNG.drive to send OutGauge packets to that address.

Then click:

```text
BeamNG Listener starten
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

# Operating Modes

The GUI supports three operating modes.

## Manual

```text
Manuell
```

All cluster values are controlled directly from the GUI.

Useful for:

- Bench testing
- Gauge testing
- Warning-light testing
- CAN experiments

---

## BeamNG Live

```text
BeamNG Live
```

BeamNG controls:

- Speed
- RPM
- Temperatures
- Fuel
- Throttle
- Gear
- Supported warning lamps
- Indicators

---

## Hybrid

```text
Hybrid
```

BeamNG supplies the main vehicle telemetry while some cluster states can remain manually controlled.

This is useful when experimenting with unsupported BeamNG outputs.

---

# Supported BMW CAN Frames

The project currently uses several BMW F-Series CAN messages.

Examples include:

| CAN ID | Function |
|---|---|
| `0x0F3` | Engine RPM |
| `0x12F` | Ignition / terminal state |
| `0x1A1` | Vehicle speed |
| `0x1EE` | Steering wheel / menu events |
| `0x1F6` | Turn indicators |
| `0x202` | Cluster backlight |
| `0x21A` | Exterior lighting |
| `0x289` | Cruise / longitudinal dynamics |
| `0x291` | Language / units |
| `0x2A7` | Steering / EPS alive frame |
| `0x2BB` | Distance / consumption context |
| `0x2C4` | Consumption / range context |
| `0x30B` | Auto Start/Stop indication |
| `0x33B` | ACC / longitudinal dynamics |
| `0x349` | Fuel level |
| `0x36A` | Automatic high beam |
| `0x3A7` | Drive mode |
| `0x3FD` | Transmission / gear |
| `0x581` | Seatbelt related frame |
| `0x5C0` | Check-Control messages |

Some CAN mappings are still experimental and may behave differently depending on:

- Cluster software version
- Vehicle coding
- Cluster hardware revision
- Vehicle equipment

---

# Fuel Gauge

The F10 fuel gauge does not use a direct `0–100` CAN percentage.

The project currently maps fuel percentage to the values used by the cluster.

Known reference points:

```text
0%   -> 0x25
50%  -> 0x12
100% -> 0x04
```

Intermediate values are calculated by the Arduino firmware.

---

# Check-Control Messages

The project can generate several BMW Check-Control messages on the instrument cluster.

Examples include:

```text
Door open
Low engine oil
Coolant warning
Brake system warning
Cruise Control error
DSC warning
Transmission warning
Washer fluid low
Battery discharge
Parking brake warning
Tire pressure warning
```

Check-Control support is experimental because message behavior can depend on cluster coding and software revision.

---

# Cruise Control / ACC

The project currently sends BMW F-Series Cruise Control and ACC frames:

```text
0x289
0x33B
```

Both use:

- Rolling counters
- BMW CAN CRC calculations
- Periodic transmission

The GUI also contains a **Cruise Reverse Engineering** section.

This allows individual payload bytes to be modified while the Arduino continues generating the correct CRC and rolling counter.

Example:

```text
Frame:
0x289

Payload Byte:
1

Value:
00..FF
```

The sweep function can automatically test a range of values.

Example:

```text
Start: 00
End:   FF
Step:  10
Delay: 700 ms
```

The exact encoding of Cruise Control SET speed is still under investigation.

---

# CAN Lab

The CAN Lab allows manual CAN experimentation.

Example:

```text
21A 06 00 F7
```

means:

```text
CAN ID:
0x21A

Payload:
06 00 F7
```

Available features:

- CAN ID dropdown
- Known CAN packet presets
- Manual payload entry
- Repeat transmission
- Adjustable interval
- RAW CAN sender
- CAN diagnostics

---

# Vehicle Identity Frame / 0x380

CAN ID:

```text
0x380
```

is associated with BMW vehicle identity / VIN information.

Because of this, the GUI and Arduino firmware intentionally block transmission of this frame by default.

It must be manually unlocked before transmission.

Do not transmit random `0x380` payloads.

Only use data captured from a system whose origin you understand.

---

# CAN Diagnostics

The firmware can report MCP2515 CAN diagnostics.

Example:

```text
CAN_DIAG EFLG=0x40 [RX0_OVERFLOW] TEC=0 REC=0 TX_OK=762 TX_FAIL=0
```

Useful values include:

```text
TEC
Transmit Error Counter

REC
Receive Error Counter

TX_OK
Successful transmissions

TX_FAIL
Failed transmissions
```

A continuously increasing transmit error counter usually indicates problems with:

- CAN wiring
- Bitrate
- Missing CAN ACK
- Termination
- Cluster power/wake state

---

# Presets

Several test sequences are included.

Examples:

```text
Needle Sweep
Complete Tacho Test
Acceleration
Highway Demo
Gear Cycle
Drive Mode Cycle
Light Test
Indicator Test
Warning Light Test
Temperature Test
Idle / Normal
```

These make it easier to verify cluster functionality without BeamNG.

---

# Safety

## Do not connect 12 V to Arduino pins

The BMW cluster uses automotive 12 V power.

Arduino GPIO pins operate at 5 V.

Never connect:

```text
12 V -> Arduino I/O
12 V -> CAN-H
12 V -> CAN-L
```

Doing so can permanently damage the Arduino, CAN shield, or cluster.

---

## Use a fused/current-limited supply

For bench testing, use:

- A current-limited power supply
- A fuse on the 12 V cluster supply
- Proper wiring
- Secure connections

Avoid powering the cluster from the Arduino.

---

## Bench use recommended

This project is designed primarily for:

```text
Bench testing
Simulation
Reverse engineering
BeamNG hardware integration
Educational experimentation
```

Do not transmit experimental CAN frames on a real vehicle CAN network unless you fully understand their behavior.

Unexpected CAN messages can interfere with vehicle systems.

---

# Known Limitations

The following areas are still experimental or incomplete:

- Exact Cruise Control SET-speed encoding
- Some Check-Control messages
- Exact language mapping for every cluster firmware
- Exact unit configuration for every cluster
- Automatic high-beam status
- Some seatbelt states
- Some drive-mode behaviors
- Some vehicle-specific warning indicators
- Full ACC behavior

Different BMW cluster software versions may react differently to the same CAN frames.

---

# Project Structure

Example repository layout:

```text
BMW-F1x-6WA-BeamNG/
│
├── Arduino/
│   └── BMW_Cluster_CAN_Controller_EXTENDED_V7_MODERN.ino
│
├── Python/
│   └── BMW_6WA_BeamNG_GUI_EXTENDED_V7_MODERN.py
│
├── docs/
│   ├── wiring.md
│   └── images/
│
├── README.md
│
└── LICENSE
```

---

# Future Plans

Possible future improvements:

- Decode Cruise Control SET speed
- Better ACC support
- More F10/F11 warning indicators
- Additional CAN frame documentation
- F30/F20 compatibility testing
- Configurable CAN packet database
- CAN logging
- CAN recording/playback
- Automatic BeamNG vehicle profiles
- Better gear display mapping
- More realistic fuel-consumption simulation
- Additional cluster variants
- Improved trip/distance simulation

---

# Disclaimer

This project is intended for educational, simulation, development, and bench-testing purposes.

BMW is a trademark of BMW AG.

This project is not affiliated with, endorsed by, or sponsored by BMW AG.

Use this software and hardware setup at your own risk.

Incorrect wiring, incorrect CAN messages, or improper power connections can damage electronic equipment.

Do not use this project to alter vehicle identification data, stored odometer values, or other legally relevant vehicle information.

---

# Credits

This project builds upon information learned from the BMW reverse-engineering community and other open-source BMW instrument-cluster projects.

Special thanks to developers and researchers documenting BMW F-Series CAN communication.

---

# License

Choose a license appropriate for your project.

For example:

```text
MIT License
```

or:

```text
GNU General Public License v3.0
```

See the `LICENSE` file for details.
