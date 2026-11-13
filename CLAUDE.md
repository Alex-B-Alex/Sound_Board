# Handheld Soundboard

A battery-powered, handheld soundboard, inspired by the remote from the TV show *iCarly*. It is a gift for a friend, and also a portfolio project for the owner's resume and personal website. The owner is a Northeastern University student in the combined CS + Computer Engineering program.

**Status (2026-11-13):** Planning. Nothing has been bought, wired, modeled or coded yet.

## What the device does

1. **Battery powered.** A battery pack in the back holds replaceable batteries (AA by default).
2. **Power switch** on the side turns it on and off.
3. **See-through, roughly square buttons.** Each button has its own small display under it showing an icon, emoji or symbol for its sound. This is the main way it differs from the original prop.
4. **Red LED next to each button** lights up while its sound plays.
5. **Microcontroller board** reads the images and sounds from storage, drives the displays and plays the audio.
6. **USB port** (prefer USB-C over micro-USB) for loading new sounds and icons into the button slots.
7. **A decently sized speaker** that is loud enough to be fun.
8. **Stretch goal: microphone.** Record a short clip, apply a funny voice effect (chipmunk, robot, deep voice, echo), and play it back. Then either delete it, or keep it and evict old clips when storage fills up.

## Constraints

- **Budget:** keep it cheap, with up to about $200 total for a good result. Don't suggest parts that blow past this without a reason.
- **Fabrication:** the enclosure is 3D printed at Northeastern's EXP makerspace. Assembly is hand soldered there too.
- **Quality bar:** it is a gift and a portfolio piece. It should look finished, with clean wiring, no loose breadboard, and parts screwed down or properly mounted. The repo and docs should also be clean enough to show a recruiter.
- **Design for the mic now.** Even if the mic comes later, leave room in the circuit, pins, enclosure and firmware so it can be added without a redesign.

## Deliverables

1. Get trained on 3D printing and soldering at the EXP makerspace (done by the owner, not an agent task).
2. Choose and gather parts, plus CAD/STEP models of those parts for the enclosure design.
3. **Circuit:** a schematic for the whole system, with the mic included or ready to drop in.
4. **CAD:** a model of the full assembly. The most important part is the printed enclosure, with screw bosses for the board and battery pack, and mounts for the buttons, displays, LEDs, speaker, switch and USB port. Parts should be mounted directly in the enclosure or on a perfboard/custom PCB, not on a breadboard.
5. **Firmware:** loads images and audio from storage, shows each slot's icon on its button display, plays the sound and lights the LED on press.
6. **Bonus, desktop app:** a portable Windows `.exe` that connects to the board over USB and shows every slot with a drag-and-drop square for its image and one for its sound. It should convert the files (resize/convert images, resample/convert audio) before uploading so the user never has to.

## Proposed technical direction (not final; confirm with the owner before buying parts)

These are starting recommendations. Record decisions in the Decisions log below once they are made.

| Subsystem | Leading option | Why / alternatives |
|---|---|---|
| MCU | **ESP32-S3** dev board with PSRAM (e.g. N16R8) | Native USB (can act as a USB drive or serial device), 2× I2S (speaker amp + mic at once), enough RAM for images, decodes MP3 easily. Alternative: Raspberry Pi Pico 2 (RP2350), which is cheaper but tighter for MP3 decoding plus several displays. |
| Button displays | Small SPI color TFTs (0.85"–1.3", ST7735/GC9A01/ST7789), one per button | They share one SPI bus with a chip-select pin per display. Monochrome I2C OLEDs (SSD1306) are cheaper but all share one address, so they need a TCA9548A mux. |
| Buttons | Clear cap (resin-printed or clear acrylic) over the display, pressing a tactile switch | **This is the hardest mechanical problem.** The display has to show through the cap while the press still reaches a switch, e.g. switches at the cap's edges or a floating cap on springs. Prototype this early. |
| LEDs | Red 3mm/5mm LEDs with resistors, one per button | If GPIO runs short, use a shift register (74HC595) or I/O expander (MCP23017), which can also read the buttons. |
| Audio out | **MAX98357A** I2S class-D amp + 3W 4Ω/8Ω speaker (40–50mm) | Cheap and loud, and the sound is clean because the audio stays digital until the amp. |
| Mic (stretch) | **INMP441** I2S MEMS mic | Goes on the ESP32-S3's second I2S peripheral. Effects are done in software: pitch shift, ring modulation (robot), delay/echo. |
| Storage | microSD card module, or the board's onboard flash | SD makes "lots of sounds" easy. Flash means fewer parts. |
| Power | 3× or 4× AA holder → slide switch → buck-boost regulator (3.3V/5V) | Add an ADC voltage divider so the firmware can read battery level. A rechargeable Li-ion cell with a charging board is an alternative if the owner changes their mind on AA. |
| Loading content | USB Mass Storage (device appears as a drive) **or** a custom serial protocol used by the desktop app | Mass storage is the simplest. A custom protocol is the better fit for the desktop app and makes a better resume story. |
| Desktop app | Python + PySide6 (Qt), packaged with PyInstaller into one portable `.exe` | Handles drag-and-drop slots, image conversion with Pillow, and audio conversion. Alternatives: Tauri or Electron. |
| Schematic / PCB | **KiCad** | A custom PCB (about $20 from a board house) is a strong resume item and tidier than perfboard. It is optional. |
| Enclosure CAD | Fusion 360 (free for students) or Onshape | Export STEP + STL into the repo. |

Open questions for the owner:
- How many buttons? Suggested default is 6–8.
- Overall size and shape of the device, and the button layout.
- Which microcontroller (see the MCU row above).
- Whether a custom PCB is in scope, or perfboard is fine.

## Planned repo layout

Create these folders as the work starts. They don't exist yet.

```
firmware/      # MCU code (Arduino/PlatformIO or ESP-IDF; pick one and note it here)
desktop-app/   # Slot editor / uploader app + build script for the portable .exe
hardware/      # KiCad schematic (+ PCB), BOM with prices and links, wiring notes
cad/           # Enclosure source files, STEP/STL exports, models of purchased parts
docs/          # Build log, photos, write-up for the website/resume
assets/        # Sample icons and sounds for testing (only royalty-free or self-made)
```

## Working notes for agents

- **Keep a running BOM** (`hardware/bom.md` or `.csv`) with part, quantity, price, link and running total against the ~$200 budget.
- **Record decisions** in the log below with the date and the reason. The reasoning behind choices matters for the portfolio write-up.
- **Firmware and app share a slot format.** Define it once (slot count, image size and pixel format such as RGB565 at the display's resolution, audio format and sample rate) and keep firmware, app and docs in sync when it changes.
- **Don't commit copyrighted audio** (e.g. clips from the show) to the repo, since it may be public. Use placeholder or royalty-free sounds in `assets/`. The friend's real sounds get loaded onto the device, not into git.
- **Document as you go** for the website: photos, problems hit and how they were solved, and lessons learned.
- Electrical safety: watch battery polarity, put a fuse or polyfuse on battery input if practical, and make sure USB power and battery power can't back-feed into each other.

## Decisions log

| Date | Decision | Reason |
|---|---|---|
| 2026-11-13 | Project started. Plan captured in this file. | — |
