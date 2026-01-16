# OakBridge MkI PCB Design Rules & Guidelines

## Board Analysis Summary

### Key Components:
- **ESP32-S3-WROOM-1**: Main microcontroller (RF module)
- **USB-C Connector**: GSB1C41110SSHR (USB 2.0)
- **3.3V LDO**: TLV1117LV33 (1A)
- **FRAM**: MB85RS4MT (4Mbit SPI)
- **Display**: 3.5" Waveshare via 11-pin connector
- **Inputs**: 5x Cherry MX switches, rotary encoder, 3x tactile switches
- **LEDs**: 5x Cree JE2835 (with series resistors)
- **MicroSD**: Card connector

---

## LAYER STACK RECOMMENDATION: **2 LAYERS IS SUFFICIENT**

### Why 2 layers works:
✅ Single digital voltage domain (3.3V)
✅ Low-speed USB 2.0 (not high-speed USB)
✅ Low-frequency SPI communication (<50 MHz)
✅ No high-power analog circuits
✅ WiFi/BT RF contained within shielded module
✅ Good grounding achievable with careful ground plane design

### Cost-benefit:
- 2-layer: ~$5-20 for prototype (JLCPCB/PCBWay)
- 4-layer: ~$40-80 for same board
- No technical advantage justifies 4x cost increase for this design

---

## VIA SPECIFICATIONS

### Signal Vias (default):
- **Drill size**: 0.3mm (12 mil)
- **Annular ring**: 0.2mm (8 mil) minimum
- **Finished hole**: 0.3mm drill = ~0.28mm finished
- **Pad diameter**: 0.7mm (28 mil)
- **Use case**: All general signal routing

### Power Vias (USB +5V, +3.3V):
- **Drill size**: 0.4mm (16 mil)
- **Pad diameter**: 0.8mm (32 mil)
- **Quantity**: Multiple vias in parallel for power distribution
- **Spacing**: 2-3mm apart in power delivery paths
- **Use**: USB input, LDO output, ESP32 power pins

### Ground Stitching Vias:
- **Drill size**: 0.3mm (12 mil)
- **Pad diameter**: 0.7mm (28 mil)
- **Spacing**: Every 5-10mm along board perimeter
- **Quantity**: ~20-30 vias around board edge
- **Purpose**: Connect top/bottom ground planes, reduce EMI

### USB Shield Vias:
- **Drill size**: 0.5mm (20 mil)
- **Pad diameter**: 1.0mm (40 mil)
- **Placement**: 4x vias surrounding USB connector
- **Purpose**: Low impedance ground connection for shield

---

## TRACE WIDTH SPECIFICATIONS

### Power Traces:
```
+5V USB Input (up to 1A):
- Width: 0.6mm (24 mil) minimum
- Preferred: 0.8mm (32 mil)
- Temperature rise: <10°C @ 1A

+3.3V Rail (up to 800mA):
- Width: 0.5mm (20 mil) minimum
- Preferred: 0.6mm (24 mil)

LED Current (20-40mA per LED):
- Width: 0.2mm (8 mil) - adequate
- From resistors to LEDs
```

### Signal Traces:
```
Standard Digital I/O:
- Width: 0.15mm (6 mil) typical
- Spacing: 0.15mm (6 mil)
- SPI, I2C, GPIO: all 0.15mm is fine

High-frequency signals (if any):
- Width: 0.2mm (8 mil)
- Keep traces short
```

### Ground Planes:
```
- Pour on both layers
- Top layer: Split only where necessary
- Bottom layer: Solid continuous plane (ideal)
- Minimum clearance: 0.2mm (8 mil)
- Thermal relief: 0.5mm spoke width, 4 spokes
```

---

## USB 2.0 DIFFERENTIAL PAIR ROUTING

### USB D+/D- Specifications:
The USB-C connector provides USB 2.0 data:
- **Target impedance**: 90Ω differential (±10%)
- **Single-ended**: ~45Ω each

### Trace Dimensions (2-layer FR-4, 1.6mm thick):
```
Trace width: 0.35mm (14 mil)
Trace spacing (edge-to-edge): 0.15mm (6 mil)
Trace gap (center-to-center): 0.50mm (20 mil)
```

This achieves **~90Ω differential impedance** on standard FR-4:
- Er = 4.5
- H (dielectric height) = 1.6mm to bottom ground plane
- Calculate: Use Saturn PCB toolkit or KiCad calculator

### Critical USB Routing Rules:
1. **Keep D+ and D- paired**: Always route together
2. **Length matching**: ±2mm (±80 mil) maximum difference
3. **No vias if possible**: Route on single layer (top preferred)
4. **If vias needed**: Use both D+ and D- vias together, keep symmetrical
5. **Avoid 90° angles**: Use 45° or curved traces
6. **Keep away from**: 
   - Switching power traces (LDO input)
   - High-frequency digital signals (SPI CLK)
   - LED switching signals
7. **Ground reference**: Continuous ground plane underneath
8. **Trace length**: Keep <75mm (3 inches) total

### USB Connector Ground Strategy:
```
1. Four shield pins → direct via to ground plane (0.5mm vias)
2. GND pins → connect to ground pour
3. Add ground stitching vias around connector (every 2-3mm)
4. Ground guard traces on both sides of D+/D- pair
```

### Series Resistors (Already in schematic):
- **R6, R7**: 5.1kΩ on CC1, CC2 (correct for USB-C PD detection)
- Place **immediately adjacent** to USB connector
- No termination resistors needed on D+/D- for device-side

---

## COMPONENT-SPECIFIC LAYOUT GUIDELINES

### ESP32-S3-WROOM-1 Module:
```
Critical requirements:
1. Ground plane: Solid copper under entire module (both layers)
2. Keep-out zone: 15mm from antenna side (check datasheet)
3. Decoupling caps: Place C2, C4 as close as possible to 3V3 pin
4. SPI routing: Keep FRAM signals (CS, SCK, MOSI, MISO) <50mm
5. Boot/enable pullups: R1, R10 near module pins
```

### TLV1117LV33 LDO Regulator:
```
Input (VIN):
- C1 (1µF): Place <3mm from input pin
- Trace from USB: 0.6mm width minimum

Output (VOUT):
- C3 (1µF): Place <3mm from output pin
- Use multiple 0.4mm vias to distribute 3.3V

Ground:
- Large thermal pad on bottom layer
- Multiple vias (5-10x) connecting to bottom ground plane
- Thermal relief NOT recommended (need heat dissipation)
```

### USBLC6-2SC6 TVS Diode:
```
Placement:
- Must be between USB connector and ESP32
- Distance from USB connector: <10mm
- Route D+/D- through TVS before going to ESP32
- Ground pin: Direct via to ground plane (0.4mm via)

Routing:
  USB D+ → TVS I/O1 → ESP32 GPIO19
  USB D- → TVS I/O2 → ESP32 GPIO20
  TVS GND → Ground plane (low impedance)
```

### Cherry MX Switches (SW1-SW5):
```
Mechanical:
- Standard MX spacing: 19.05mm (0.75")
- PCB cutout: 14mm x 14mm
- Ensure adequate clearance for keycap travel

Electrical:
- Pull-up resistors: Already included (10kΩ)
- Debouncing: Software (GPIO inputs have hardware filters)
- Series resistors: 33Ω (R17, R19, R22, R23, R26) - correct!
```

### LED Routing:
```
Current-limiting resistors: 68Ω (R27, R30, R34) for 3 LEDs
                           33Ω for 2 LEDs (depends on color/current)

Placement:
- Resistor adjacent to LED (cathode side preferred)
- Trace width: 0.2mm adequate for <40mA
- Ground return: Use local ground plane
```

### MicroSD Card Slot:
```
SPI routing to ESP32:
- Keep all SPI traces <50mm
- Route on same layer (avoid vias)
- No need for matched lengths (low speed)
- Decoupling: C13 (1µF) near SD card power pin

Card detect switch:
- Pull-up already included in design
- Debounce in software
```

### Rotary Encoder (ACZ11BR1E):
```
Mounting:
- Through-hole component
- Ensure mechanical alignment with panel cutout

Routing:
- A, B, Common pins to ESP32 GPIOs
- Pull-ups: 10kΩ (R11, R12, R13)
- Shield connection: Ground via
- Debounce: Handle in firmware
```

---

## PCB DESIGN RULES (KiCad Settings)

### Design Rules → Constraints:
```
Minimum clearance: 0.15mm (6 mil)
Minimum track width: 0.15mm (6 mil)
Minimum via diameter: 0.7mm (28 mil)
Minimum via drill: 0.3mm (12 mil)
Minimum hole-to-hole: 0.5mm (20 mil)

Differential pairs (USB):
- Width: 0.35mm
- Gap: 0.15mm
- Max uncoupled: 0.5mm (design for 0.2mm)
```

### Design Rules → Pre-defined Sizes:
```
Via size #1 (Signal):
- Diameter: 0.7mm / Drill: 0.3mm

Via size #2 (Power):
- Diameter: 0.8mm / Drill: 0.4mm

Via size #3 (Ground stitching):
- Diameter: 0.7mm / Drill: 0.3mm

Track width #1 (Signal): 0.15mm
Track width #2 (Power 3.3V): 0.5mm
Track width #3 (Power 5V): 0.6mm
Track width #4 (USB diff): 0.35mm
```

### Board Setup → Physical Stackup:
```
2-Layer board:
- Total thickness: 1.6mm (standard)
- Copper weight: 1 oz (35µm) on both layers
- Core: FR-4 (Er ~4.5)
- Solder mask: Both sides (green/black/blue)
- Silkscreen: Both sides
- Surface finish: ENIG (gold) or HASL (lead-free)
```

### Solder Mask Expansion:
```
Solder mask expansion: 0.05mm (2 mil)
Solder paste shrinkage: 0.0mm (default)
Minimum web width: 0.1mm

For USB connector pads:
- Consider "solder mask defined" pads
- Prevents solder bridging on fine-pitch
```

---

## GROUNDING STRATEGY

### Two-Layer Ground Approach:
```
TOP LAYER (Component side):
- Ground pour with splits around:
  * USB differential pair
  * High-current power traces
  * RF antenna keep-out zone
- Connect pour to bottom ground with stitching vias

BOTTOM LAYER (Solder side):
- Solid continuous ground plane (IDEAL)
- No splits except for specific isolation
- Main return path for all signals
- ESP32 module sits over solid ground
```

### Ground Stitching Pattern:
```
Via placement:
1. Around board perimeter: Every 5-10mm
2. Under ESP32 module: 10-15 vias
3. Around USB connector: 6-8 vias
4. Between power sections: 3-5 vias
5. Near high-speed signals: Every 10mm along trace

Total ground vias: ~40-60 for typical board size
```

### Ground Connection Priority:
```
1. USB shield → Ground (0.5mm vias, 4x)
2. LDO thermal pad → Ground (0.4mm vias, 8x)
3. ESP32 GND pins → Ground (0.3mm vias, 4x)
4. Bypass cap grounds → Ground (0.3mm vias, 1-2x each)
5. Connector shields → Ground
```

---

## MANUFACTURING & ASSEMBLY NOTES

### Fabrication Specifications (JLCPCB/PCBWay):
```
Board size: Measure actual (likely <100x100mm)
Layers: 2
Thickness: 1.6mm
Min track/space: 6/6 mil (0.15/0.15mm)
Min hole size: 0.3mm
Copper weight: 1 oz
Surface finish: ENIG preferred (better for RF/USB)
Solder mask color: Your choice
Silkscreen: White on dark, black on light
```

### Panelization:
```
If ordering multiple boards:
- Add mouse bites or V-grooves
- Keep 5mm border around board edge
- Tooling holes: 4x at corners
```

### Assembly Recommendations:
```
1. Reflow profile for 0603 components: Standard SAC305
2. Stencil thickness: 0.12mm (standard)
3. Paste release: 1:1 ratio (stencil holes = pads)
4. Order: 
   a. Reflow SMD components
   b. Hand-solder through-hole (switches, encoder, connectors)
```

### Special Considerations:
```
- ESP32 module: Pre-assembled, reflow once only
- Cherry MX switches: Hand solder after SMD reflow
- USB-C connector: Verify alignment before soldering
- MicroSD slot: Correct orientation critical
- Display connector: Pin 1 alignment
```

---

## DESIGN RULE CHECK (DRC) SETTINGS

### Critical Checks:
```
✅ Clearance violations
✅ Track width violations
✅ Via size violations
✅ Hole-to-hole spacing
✅ Copper-to-edge clearance (0.5mm min)
✅ Silkscreen-to-pad clearance (0.15mm)
✅ Courtyard overlap (component collision)
✅ Unconnected pads
✅ Missing traces
```

### Electrical Rules Check (ERC):
```
✅ Power pins connected
✅ Pull-ups/downs present
✅ Unused inputs tied to valid logic levels
✅ Decoupling caps on all power pins
✅ Crystal loading caps (N/A - using internal oscillator)
✅ Reset/boot circuitry correct
```

---

## IMPEDANCE CALCULATIONS (Reference)

### USB Differential Impedance:
For 2-layer board (1.6mm FR-4, 1oz copper):
```
W = 0.35mm (trace width)
S = 0.15mm (edge-to-edge spacing)
H = 1.6mm (height to ground plane)
Er = 4.5 (FR-4)

Calculated Zdiff ≈ 88-92Ω ✅ (target: 90Ω ±10%)
```

Verify using:
- KiCad PCB Calculator (Tools → Calculator)
- Saturn PCB Toolkit
- Online calculator: https://www.eeweb.com/tools/differential-stripline-impedance/

---

## TESTING & VALIDATION CHECKLIST

### Pre-Assembly Tests:
```
□ Visual inspection: No short circuits, clean pads
□ Continuity test: Power rails isolated from ground
□ USB connector: No shorts between D+, D-, VBUS, GND
```

### Post-Assembly Tests:
```
□ Power-on test: Measure 5V at USB input
□ LDO output: Verify 3.3V ±3%
□ ESP32 boot: Check via serial (EN button + GPIO0)
□ USB enumeration: Device appears on PC
□ LED test: All LEDs functional
□ Switch test: All switches register
□ MicroSD: Card detect and SPI communication
□ Display: SPI communication and display init
□ Encoder: A/B phase signals
```

---

## RECOMMENDED LAYOUT WORKFLOW

1. **Placement** (Most critical step):
   - USB connector on board edge
   - ESP32 in center region
   - LDO near USB input
   - TVS diode between USB and ESP32
   - MicroSD slot near ESP32 SPI pins
   - Switches arranged ergonomically
   - LEDs visible from front panel

2. **Power routing** (Do this first):
   - 5V from USB to LDO input
   - 3.3V from LDO to all components
   - Multiple vias for power distribution
   - Ground plane on both layers

3. **Critical signal routing**:
   - USB D+/D- differential pair (0.35mm, 0.15mm gap)
   - SPI signals (FRAM, MicroSD)
   - Display connector signals

4. **General signals**:
   - Switch inputs
   - LED outputs
   - Encoder signals

5. **Ground plane pour**:
   - Top layer: Pour with keepouts
   - Bottom layer: Solid pour
   - Add stitching vias (~50 total)

6. **Silkscreen**:
   - Component references
   - Pin 1 indicators
   - Polarity markers
   - Version number
   - Your logo/name

7. **Final checks**:
   - DRC: Zero errors
   - Visual inspection
   - 3D view verification
   - Gerber preview

---

## COST ESTIMATES (2025 Prices)

### PCB Fabrication:
```
2-layer, 100x100mm, 1.6mm, ENIG:
- JLCPCB: $5 for 5 boards + shipping (~$20-30 total)
- PCBWay: Similar pricing
- OSH Park: ~$5/sq.in × area (~$25-35 total)

Turnaround:
- Standard: 7-10 days
- Express: 3-5 days (+50% cost)
```

### Component Assembly (if using JLCPCB):
```
Basic SMD assembly: ~$15-30
+ Extended parts library: +$3/part type
+ Setup fee: $8
Total BOM cost: ~$50-80 (depends on parts)

Hand assembly recommended for this design:
- Through-hole switches
- Display connector
- USB connector (easier by hand)
```

---

## FILES TO GENERATE FOR MANUFACTURING

### Required Gerber Files:
```
- F.Cu (Top copper)
- B.Cu (Bottom copper)
- F.Mask (Top solder mask)
- B.Mask (Bottom solder mask)
- F.Silkscreen (Top silkscreen)
- B.Silkscreen (Bottom silkscreen)
- Edge.Cuts (Board outline)
- F.Paste (Top solder paste)
- B.Paste (Bottom solder paste)

Drill files:
- .drl (Excellon drill file)
- PTH and NPTH separate or combined
```

### Additional Files:
```
- BOM (CSV or Excel)
- CPL/PnP (Pick-and-place coordinates)
- Assembly drawing (PDF)
- Schematic (PDF)
- 3D model (STEP file - for enclosure design)
```

---

## FINAL RECOMMENDATIONS

### For your OakBridge MkI design:

✅ **Use 2 layers** - No technical need for 4 layers
✅ **Standard vias** (0.7mm/0.3mm drill) work fine
✅ **USB impedance**: 0.35mm traces, 0.15mm gap → 90Ω
✅ **Power traces**: 0.5-0.6mm adequate for currents involved
✅ **Ground plane**: Bottom layer solid, top layer poured
✅ **Total cost**: ~$30-50 for 5 prototype boards

### Next Steps:
1. Apply these design rules in KiCad
2. Complete component placement
3. Route critical signals (USB, power, SPI)
4. Route remaining signals
5. Pour ground planes
6. Add ground stitching vias
7. Run DRC (fix all errors)
8. Generate Gerbers
9. Order from JLCPCB/PCBWay

Good luck with your layout! 🚀

