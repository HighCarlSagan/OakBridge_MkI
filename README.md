# OakBridge MkI

**A customizable PC monitoring dashboard with tactile control and real-time system visualization**

Built around the ESP32-S3, OakBridge MkI combines a vibrant 3.5" IPS display with Cherry MX mechanical switches and a precision rotary encoder to create an interactive desktop companion. Monitor system performance, control media playback, display photos, access quick references, and interact with your PC through WiFi or USB-C—all housed in a handcrafted wooden enclosure.

---

## Overview

OakBridge MkI is an open-source hardware project that bridges the gap between your PC and your desktop workspace. Featuring a 320×480 IPS touchscreen, five programmable Cherry MX switches, RGB status LEDs, and wireless connectivity, it transforms system monitoring and control into a tactile, visual experience.

The project integrates custom PCB electronics, ESP32-S3 firmware (ESP-IDF), a Python companion application for PC communication, and parametric enclosure designs. Whether you're monitoring CPU temps during a compile, controlling Spotify without leaving your IDE, or keeping a family photo slideshow on your desk, OakBridge adapts to your workflow.

Designed with maker-friendly assembly in mind, the hardware uses common SMD components (0603 resistors/capacitors), through-hole switches, and a 2-layer PCB optimized for low-cost fabrication. Complete hardware documentation, firmware source, and companion software are included.

### Design Capabilities

- **Display**: 3.5" IPS touchscreen (320×480, Waveshare)
- **Input**: 5× Cherry MX mechanical switches with customizable keycaps
- **Control**: Rotary encoder with integrated push button
- **Indicators**: 5× RGB LEDs (Cree JE2835)
- **Connectivity**: WiFi 802.11b/g/n + Bluetooth 5.0 (ESP32-S3)
- **Interface**: USB-C (power + serial communication)
- **Storage**: MicroSD card slot + 4Mbit FRAM non-volatile memory
- **Power**: USB-C 5V input, onboard 3.3V regulation (1A LDO)
- **Enclosure**: Wooden case (walnut) with parametric CAD design

---

## Table of Contents

- [Hardware](#hardware)
  - [Electronics](#electronics)
  - [Enclosure](#enclosure)
- [Firmware](#firmware)
  - [ESP32 Firmware](#esp32-firmware)
  - [PC Companion](#pc-companion)
- [Assembly](#assembly)
- [Roadmap](#roadmap)
- [License](#license)
- [Contributing](#contributing)
- [Acknowledgments](#acknowledgments)

---

## Hardware

### Electronics

**Schematic**: [`hardware/docs/schematic.pdf`](hardware/docs/schematic.pdf)

**PCB Files**: `hardware/PC_Interface_Card/` (KiCad 9.0 project)

**Bill of Materials**: [`hardware/PC_Interface_Card/Exports/PC_Interface_Card.csv`](hardware/PC_Interface_Card/Exports/PC_Interface_Card.csv)

**Netlist**: [`hardware/PC_Interface_Card/Exports/PC_Interface_Card.net`](hardware/PC_Interface_Card/Exports/PC_Interface_Card.net)

#### Key Components

| Component | Part Number | Description |
|-----------|-------------|-------------|
| Microcontroller | ESP32-S3-WROOM-1-N16R8 | Dual-core Xtensa LX7, 16MB Flash, 8MB PSRAM |
| Display | Waveshare 3.5" RPi LCD (G) | 320×480 IPS resistive touch, SPI interface |
| Switches | Cherry MX2A-E1NA | PCB-mount mechanical switches (5×) |
| Encoder | ACZ11BR1E-15KQA1-12C | 12 PPR incremental rotary encoder |
| LEDs | Cree JE2835ARY-N | Blue LEDs (5×) with current limiting |
| Memory | MB85RS4MTYPN-GS | 4Mbit FRAM, SPI interface |
| USB | Amphenol GSB1C41110SSHR | USB Type-C receptacle (USB 2.0) |
| Regulator | TLV1117LV33DCYR | 3.3V LDO, 1A output current |
| Protection | USBLC6-2SC6 | USB ESD protection diode array |

#### PCB Specifications

- **Layers**: 2-layer FR-4 (1.6mm thickness)
- **Copper Weight**: 1 oz (35µm)
- **Surface Finish**: ENIG or HASL lead-free
- **Dimensions**: [To be added after layout completion]
- **Design Rules**: 
  - Minimum trace width: 0.15mm (6 mil)
  - Minimum clearance: 0.15mm (6 mil)
  - USB differential pairs: 0.35mm width, 0.15mm gap, 90Ω impedance
  - Via sizes: 0.7mm/0.3mm (signal), 0.8mm/0.4mm (power)
  - Ground stitching: ~50 vias for EMI reduction

**Design Documentation**: 
- [`hardware/docs/pcb_design_rules.md`](hardware/docs/pcb_design_rules.md) - Comprehensive PCB design guidelines
- [`hardware/docs/usb_routing_quick_ref.txt`](hardware/docs/usb_routing_quick_ref.txt) - USB 2.0 routing specifications
- [`hardware/docs/kicad_design_rules.json`](hardware/docs/kicad_design_rules.json) - KiCad project configuration

#### System Hardware Interfaces

**USB-C Interface**:
- **Power Delivery**: 5V input (up to 1A typical)
- **Data Communication**: USB 2.0 Full Speed (12 Mbps)
- **Serial Protocol**: CDC-ACM virtual COM port
- **Use Cases**: Firmware updates, serial debugging, PC companion communication

**MQTT Interface** *(Firmware implementation)*:
- **Broker**: Connects to local MQTT broker on PC/network
- **Topics**: 
  - `oakbridge/stats` - Receives CPU, GPU, RAM metrics from PC companion
  - `oakbridge/media` - Media player state (track, artist, playback status)
  - `oakbridge/commands` - Control commands from switches/encoder
- **QoS**: Level 1 (at least once delivery)
- **Transport**: WiFi (2.4GHz 802.11b/g/n)

**HTTP Interface** *(Firmware implementation)*:
- **Web Server**: Embedded HTTP server for configuration
- **Endpoints**:
  - `/config` - WiFi credentials, display settings
  - `/api/status` - Device status JSON
  - `/ota` - Over-the-air firmware updates
- **Access**: mDNS responder (`oakbridge.local`)

### Enclosure

A wooden enclosure design (walnut) is planned for OakBridge MkI. Parametric CAD files will be shared in the `enclosure/` directory, allowing customization for different materials (acrylic, 3D printed ABS/PLA, aluminum) or dimensions.

**Status**: Design files for the wooden enclosure will be added in a future update. The enclosure features panel cutouts for the display, USB-C port, switches, rotary encoder, and LEDs. Mounting holes align with PCB standoffs for secure assembly.

Additional details including material specifications, joinery methods, finishing options, and assembly instructions will be documented as the enclosure design progresses.

---

## Firmware

### ESP32 Firmware

**Framework**: ESP-IDF (Espressif IoT Development Framework)

**Location**: `firmware/`

The ESP32 firmware handles display rendering, touch input processing, switch/encoder reading, WiFi/BLE connectivity, and communication with the PC companion application. Built using ESP-IDF's component-based architecture for modularity and maintainability.

**Key Features**:
- **Display Driver**: Custom LVGL integration for the Waveshare 3.5" display
- **Input Handling**: Hardware debouncing for switches, quadrature decoding for encoder
- **Network Stack**: WiFi station mode, mDNS responder, MQTT client, HTTP server
- **Storage**: FRAM for non-volatile configuration, MicroSD for logs/assets
- **OTA Updates**: Secure firmware updates over WiFi

**Build Requirements**:
- ESP-IDF v5.x or later
- Python 3.8+
- CMake 3.16+

**Firmware Structure**:
```
firmware/
├── main/
│   ├── app/           # Application logic
│   ├── comms/         # Network communication (WiFi, MQTT, HTTP)
│   ├── drivers/       # Hardware drivers (display, switches, encoder)
│   ├── screens/       # UI screen definitions
│   ├── storage/       # FRAM, SD card, NVS management
│   ├── ui/            # LVGL UI components
│   └── utils/         # Helper functions
├── components/        # Reusable ESP-IDF components
├── resources/         # Fonts, images, web assets
└── tools/             # Build scripts, flash utilities
```

**Getting Started**: Detailed firmware build and flash instructions will be added to `docs/firmware_guide.md`.

### PC Companion

**Language**: Python 3.8+

**Platform Support**: Linux (primary), Windows and macOS support coming soon

**Location**: `companion/`

The PC companion application runs on your desktop, collecting system metrics (CPU usage, GPU stats, RAM, temperatures) and media player information (currently playing track, artist, playback state). Data is transmitted to OakBridge over MQTT or serial connection.

**Features**:
- System monitoring (CPU/GPU/RAM/disk usage)
- Hardware sensors (temperatures, fan speeds via `psutil`, `pynvml`)
- Media player integration (MPRIS on Linux)
- MQTT publisher for wireless communication
- Lightweight background service (<1% CPU usage)

**Dependencies**: Listed in `companion/requirements.txt`

**Usage**:
```bash
cd companion/
pip install -r requirements.txt
python display_server.py
```

**Configuration**: Connection settings, update intervals, and enabled metrics are configured via `config.yaml` (to be added).

---

### Flashing Pre-built Firmware

If firmware binaries are provided in releases:

```
# Install esptool
pip install esptool

# Flash firmware (adjust port as needed)
esptool.py --port /dev/ttyACM0 --baud 460800 write_flash 0x0 firmware.bin
```

## Assembly

**Status**: In Development (Hardware Revision: MkI)

OakBridge MkI uses surface-mount components (0603 resistors/capacitors) and through-hole switches/connectors. Assembly can be done with standard soldering equipment or automated pick-and-place for prototyping.

**Basic Assembly Steps**:

1. **PCB Fabrication**: Generate Gerber files from KiCad project (`hardware/PC_Interface_Card/`)
2. **Component Sourcing**: Order parts from BOM (`hardware/PC_Interface_Card/Exports/PC_Interface_Card.csv`)
3. **SMD Assembly**: Reflow or hand-solder 0603 passives, ICs, and QFN packages
4. **Through-Hole**: Solder Cherry MX switches, USB-C connector, display header, encoder
5. **Testing**: Verify power rails (3.3V, 5V), USB enumeration, display initialization
6. **Enclosure**: Assemble wooden case, mount PCB with standoffs
7. **Firmware**: Flash ESP32 firmware via USB-C

**Tools Required**:
- Soldering iron (temperature-controlled recommended)
- Solder paste + hot air station (for SMD) OR reflow oven
- Multimeter for continuity/voltage checks
- USB-C cable for programming

Detailed assembly guide with photos and troubleshooting tips will be added to `docs/assembly_guide.md`.

---

## Roadmap

**Current Status**: Hardware design complete, PCB layout in progress, firmware architecture defined

**Upcoming Milestones**:

- [ ] Complete PCB routing and design rule checks
- [ ] Order prototype PCBs and components
- [ ] Assemble and test first hardware revision
- [ ] Finalize enclosure CAD design (walnut variant)
- [ ] Implement core firmware features (display driver, network stack)
- [ ] Develop PC companion application (Linux version)
- [ ] Add Windows and macOS support to PC companion
- [ ] Create detailed assembly documentation with photos
- [ ] Publish gerber files and manufacturing guidelines
- [ ] Write user manual and API reference

**Future Enhancements**:
- Additional display screens (weather, calendar, notifications)
- Customizable switch actions via web interface
- Integration with home automation systems (Home Assistant, OpenHAB)
- Mobile app for configuration and remote control
- Alternative enclosure designs (acrylic, 3D printed)

---

Supported brokers: Mosquitto, HiveMQ, EMQX, or any standard MQTT 3.1.1 broker.

## Companion Application

The optional PC companion application (`companion/display_server.py`) provides:

- System statistics monitoring (CPU, RAM, GPU temperature)
- Automatic display updates
- CLI for quick display commands
- MQTT bridge for remote sync

### Installation

```
cd companion
pip install -r requirements.txt
cp config.example.yaml config.yaml
python display_server.py
```

## Troubleshooting

Common issues and solutions are documented in `docs/troubleshooting.md`. Quick reference:

| Issue | Solution |
|-------|----------|
| Display not working | Check FPC cable orientation and connection |
| USB not recognized | Verify USB data lines, try different cable |
| WiFi won't connect | Check antenna clearance, verify credentials |
| Device runs hot | Verify thermal vias, reduce brightness |

## Version History

See `CHANGELOG.md` for detailed version history.

## License

This project is licensed under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S-2.0)**.

See [`LICENSE`](LICENSE) for full license text.

**Summary**:
- ✅ You are free to use, modify, and distribute this hardware design
- ✅ Commercial use is permitted
- ⚠️ Derivative works must be released under the same license
- ⚠️ You must provide attribution and share design files

---

## Contributing

Contributions are welcome! Whether you're fixing bugs, improving documentation, adding features, or designing alternative enclosures, your help is appreciated.

**How to Contribute**:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Make your changes with clear commit messages
4. Test thoroughly (hardware changes require prototype verification)
5. Submit a pull request with detailed description

**Areas for Contribution**:
- Firmware features (new display screens, sensor integrations)
- PC companion enhancements (additional OS support, new metrics)
- Enclosure designs (alternative materials, form factors)
- Documentation improvements
- Hardware testing and validation

All contributions will be reviewed for compatibility and adherence to the project's design philosophy. For major changes, please open an issue first to discuss your proposal.

---

## Acknowledgments

OakBridge MkI was made possible by the excellent work of the open-source hardware and software community:

**Design Tools**:
- [KiCad EDA](https://www.kicad.org/) - PCB design and schematic capture
- [FreeCAD](https://www.freecad.org/) - Parametric 3D CAD for enclosure design
- [Blender](https://www.blender.org/) - 3D modeling and visualization

**Documentation & Resources**:
- [Espressif Systems](https://www.espressif.com/) - ESP32-S3 documentation and SDK
- [Waveshare Electronics](https://www.waveshare.com/) - Display module documentation and drivers
- Open-source CAD models and footprints from the hardware community

**Special Thanks**:
- The KiCad library team for comprehensive component symbols and footprints
- Espressif's excellent ESP-IDF framework and documentation
- The maker community for shared knowledge and design best practices

---

**Project Status**: ⚙️ In Development (Hardware MkI)

**Maintainer**: Mayank S (HighCarlSagan)

**Repository**: https://github.com/HighCarlSagan/PC_Interface_Box/
