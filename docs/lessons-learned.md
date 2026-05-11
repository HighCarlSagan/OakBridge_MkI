# Lessons Learned

A standing log of things this project taught me — kept here so future-me reads it before starting the next board.

---

## 1. The 0mm-via incident (v1.0, May 2026)

### The mistake

When I configured the KiCad project's via classes for OakBridge MkI, I left the **drill diameter at 0 mm** on one class. KiCad represents this as "use the rule from the class," and the class happened to inherit a 0 value that I never overrode.

This is exactly the kind of thing DRC exists to catch. KiCad's DRC would have flagged it in a single line.

### Why it shipped anyway

Three failures stacked:

1. **I skipped DRC before export.** The layout *looked* clean in the 3D renders and after a visual board-edge check. I told myself a final DRC pass was a formality and went straight to Gerber export.
2. **The fab did not flag it.** Most fabs run a DFM (Design for Manufacturing) review that catches structurally impossible features — a via with 0 mm drill is one of them. Either this fab didn't run that check, ran it and didn't surface a flag, or surfaced a flag I missed in the order confirmation email.
3. **I assembled before testing.** I soldered the whole board, including the ESP32 module, before checking continuity through any of the vias on the affected net class. By the time I noticed, the components were committed to a dud PCB.

### How I noticed

Continuity test between two pads that should have been on the same net read open. Probing a via pad on the same net read the via pad itself, but not the other side of the board. Held the bare side of the board to a light — pads, no holes.

### The cost

- One PCB order (5× boards from a prototype fab, ~$30 shipped)
- One BOM order (~$80 in unique components — ESP32 module, FRAM, display socket, Cherry MX × 5, encoder, USB-C)
- One Saturday of careful hand-soldering (~6 hours)
- The hit to my "yeah I know what I'm doing" ego

Total: roughly $110 and a day. As tuition for "always run DRC," it's cheap. At work, the same mistake on a flight board would cost an order of magnitude more and a schedule slip.

### What changed

Three concrete changes, all small, all permanent:

1. **DRC is now an export gate.** My personal KiCad project checklist now reads:
   - [ ] Run DRC. Zero violations, zero unresolved warnings.
   - [ ] Run electrical rules check (ERC).
   - [ ] Generate Gerbers.
   - [ ] Generate drill files.
   - [ ] Verify drill file in a separate viewer (not the same KiCad instance).
   - [ ] Open the Gerbers in `gerbview` and visually confirm drill layer alignment.
   - [ ] Then — and only then — submit to the fab.

2. **I read the DFM report.** Every fab order from now on, I open the DFM/DRC report the fab sends back *before* approving the production run, not after. If they don't send one, I ask.

3. **No assembly before continuity sanity check.** Before any SMD reflow on a new spin, a quick continuity sweep on three or four representative nets — power, ground, USB D+/D−, and one signal that should route through a via. Takes five minutes, catches structural failures.

### Why it's in the public README

Because the alternative is pretending the project went smoothly and that I built it on the first try. That's not what happened. Owning the mistake explicitly is the honest version and also — practically — the version that helps anyone else who's about to skip DRC for the same reason I did. If this note saves one person one Saturday, it has paid for itself.

---

## 2. Touch wired but unused

Decided late in the layout that wiring the capacitive touch lines from the Waveshare 3.5" was free (the FPC fans them out anyway) and shipping firmware support for them was *not* free. So the lines are on the board, terminated correctly, and the firmware ignores them.

The lesson is the principle: **route the optional input, defer the firmware**. The pin-out cost is sunk now; if I ever want touch in v2 or in a fork, the hardware already supports it.

This generalises: if a feature is cheap on the PCB and expensive in firmware, ship the hardware and skip the firmware. The reverse — committing the firmware before the hardware exists — is the path to vapourware.

---

## 3. Single-channel backlight is fine, actually

I considered RGB backlighting per key (WS2812B-2020 under each switch, daisy-chained). Rejected it for MkI because:

- One data line, one timing-sensitive protocol, one more thing to debug at bring-up.
- 5× independent blue LEDs with individual current-limit resistors gives me per-key on/off control via GPIO, which is 90% of what backlight actually needs to do.
- The aesthetic I'm going for ("warm wood + a few blue glows") doesn't want a rainbow.

If I'd been designing for community appeal rather than my own desk, the answer might be different. But the project is mine and the call was right for the scope.

---

*This file is a running log. Updated as new lessons turn up.*
