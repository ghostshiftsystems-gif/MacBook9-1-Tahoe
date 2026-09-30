# macOS Tahoe 26.7 on MacBook9,1

Experimental macOS Tahoe 26.7 support for the **Early 2016 12-inch MacBook (MacBook9,1)** with full Intel HD Graphics 515 acceleration.

This project documents the configuration I used to take a MacBook9,1 from the normal unaccelerated Tahoe state:

```text
Intel HD Graphics 515
15 MB
VESA / no acceleration
```

to:

```text
Intel HD Graphics 515
1536 MB
Full graphics acceleration
Liquid Glass
Wi-Fi
Bluetooth / Continuity stack
Audio
Safari hardware acceleration
Normal desktop use
```

## Tested Hardware

This exact configuration has been tested on:

```text
Model:       MacBook9,1
Machine:     12-inch MacBook, Early 2016
CPU:         Intel Core m3-6Y30
iGPU:        Intel HD Graphics 515
Real PCI ID: 191E
macOS:       Tahoe 26.7
```

Other 2016 MacBook9,1 configurations, including Core m5 and Core m7 models, are interesting test candidates, but should currently be considered **experimental until independently confirmed**.

---

# How It Works

Tahoe no longer provides the normal Skylake graphics path required by the Intel HD 515.

Instead of restoring the old Skylake system graphics stack, this project keeps Tahoe's **stock Kaby Lake graphics stack** and presents the Skylake HD 515 through its corresponding Kaby Lake path.

The tested mapping is:

```text
Intel HD 515 / Skylake
191E
  ↓
spoof
  ↓
Intel HD 615 / Kaby Lake
591E
```

with:

```text
AAPL,ig-platform-id = 591E0001
```

The important graphics configuration uses:

```text
Lilu.kext
WhateverGreen 1.7.1 DEBUG
KBL191EInjector.kext
```

and the following WhateverGreen arguments:

```text
wegtree=1
-igfxsklaskbl
igfxframe=0x591E0001
igfxfw=0
igfxpavp=0
-igfxnotelemetryload
```

The system graphics files themselves remain **stock Tahoe**.

No old Skylake graphics kexts are installed into `/System/Library/Extensions`.

---

# What Works

On the tested MacBook9,1:

- ✅ Intel HD Graphics 515 acceleration
- ✅ 1536 MB graphics memory reported
- ✅ Metal / WindowServer acceleration
- ✅ Liquid Glass
- ✅ Safari GPU acceleration
- ✅ Wi-Fi
- ✅ Bluetooth / wireless stack
- ✅ Audio
- ✅ Normal multitasking
- ✅ Sleep/wake in normal use
- ✅ Tahoe desktop and animations

Wireless/Continuity and audio restoration are handled through **Skyfall**, while the custom EFI configuration handles the Skylake → Kaby Lake graphics path.

---

# Included Files

```text
MacBook9-1-Tahoe/
│
├── README.md
├── INSTALL.md
├── RECOVERY.md
├── CREDITS.md
├── LICENSE
│
└── EFI-Files/
    ├── Lilu.kext
    ├── WhateverGreen.kext
    ├── KBL191EInjector.kext
    └── TAHOE-ACCELERATION-SKL-MacBook9,1-SANITIZED.plist
```

The supplied plist is a **sanitized public configuration** and does not intentionally contain the original machine's private SMBIOS identity information.

---

# Installation

**Do not simply copy the files and reboot without reading the guide.**

This installation involves:

```text
Working macOS
      ↓
Skyfall OpenCore EFI
      ↓
Install Tahoe
      ↓
Complete slow unaccelerated Setup Assistant
      ↓
Skyfall root patching
      ↓
Confirm Wi-Fi/audio baseline
      ↓
Install WEG 1.7.1 Debug
      ↓
Install KBL191EInjector.kext
      ↓
Install acceleration config.plist
      ↓
Reboot
      ↓
Full HD 515 acceleration
```

Read the full guide:

### [INSTALL.md](INSTALL.md)

---

# Recovery

**Back up your current EFI before changing anything.**

A second working macOS installation and/or USB recovery EFI is strongly recommended.

If the acceleration config fails, black-screens, kernel panics, or returns you to an unusable state, follow:

### [RECOVERY.md](RECOVERY.md)

Do not immediately reinstall macOS because of a graphics boot failure. In most cases, restoring the previous known-good EFI is enough.

---

# Important: WhateverGreen Version

The known-good setup documented here uses:

```text
WhateverGreen 1.7.1 DEBUG
```

For your first attempt, use the tested version supplied/referenced by this project.

Do not immediately substitute another WhateverGreen build and assume the behavior will be identical.

Get the known-good configuration working first.

---

# Important: The Injector

`KBL191EInjector.kext` must be placed in:

```text
EFI/EFI/OC/Kexts/
```

alongside:

```text
Lilu.kext
WhateverGreen.kext
```

Do **not** install these project kexts into:

```text
/System/Library/Extensions
/Library/Extensions
```

The graphics components used by this project belong in the OpenCore EFI.

---

# What NOT To Do

The final working configuration does **not** require:

```text
AppleIntelSKLGraphics.kext copied into Tahoe
AppleIntelSKLGraphicsFramebuffer.kext copied into Tahoe
old Sonoma graphics frameworks
manual kmutil rebuilding for graphics
DSC patching
40 MB framebuffer memory patches
replacing Tahoe system graphics files
```

Several of those approaches were tested during development.

They are **not part of the final solution**.

The successful architecture is:

```text
Stock Tahoe KBL graphics stack
            +
OpenCore EFI compatibility layer
            +
SKL 191E → KBL 591E spoof
```

---

# About the Final WindowServer Fix

During development, the Kaby Lake graphics path eventually reached real acceleration but WindowServer still crashed in the Intel Metal/render pipeline.

The final known-good configuration introduced:

```text
igfxpavp=0
-igfxnotelemetryload
```

together.

After this, the machine reached a stable accelerated Tahoe desktop.

At this time, I have **not isolated which of these two settings is individually required**.

For that reason, the known-good configuration keeps both.

This project tries to distinguish between:

```text
proven working configuration
```

and:

```text
suspected individual cause
```

rather than pretending every variable has already been independently tested.

---

# Other Skylake Macs

Do **not** assume every Skylake GPU should use:

```text
591E
591E0001
```

Those values are specifically used for the HD 515 configuration documented here.

The broader idea is:

```text
Your Skylake GPU
        ↓
appropriate corresponding Kaby Lake GPU
```

For example, a different Skylake GPU may require a different:

```text
device-id
AAPL,ig-platform-id
injector configuration
```

Do not blindly use the MacBook9,1 plist on another machine.

If you successfully reproduce this method on another Skylake Mac, please report:

```text
Mac model
CPU
real iGPU device ID
spoofed KBL device ID
platform-id
Tahoe version
working / broken features
```

Additional confirmed hardware results would be very useful.

---

# Skyfall

Skyfall is used for the working Tahoe OpenCore baseline and restoration of hardware such as wireless/Continuity and Modern Audio.

This repository does **not** redistribute `Skyfall.pkg`.

Download Skyfall from its official project/source.

This project is not an official Skyfall release.

---

# Credits

This project builds on a large amount of existing work.

Major credits include:

- **Acidanthera** — OpenCore, Lilu and WhateverGreen
- **Dortania / OpenCore Legacy Patcher contributors**
- **xtomasnemec** — Skyfall
- **YBronst** — Tahoe/OpenCore compatibility research
- **Raz / NoClipRoutine** — MacBook9,1 Tahoe testing, SKL → KBL integration, dual-ID testing, WindowServer debugging and documentation
- the wider unsupported-Mac, OpenCore, InsanelyMac, MacRumors and Hackintosh communities

See:

### [CREDITS.md](CREDITS.md)

for the complete attribution notes.

---

# License

Original material in this repository is licensed under the **BSD 3-Clause License**, unless otherwise stated.

Third-party projects, binaries and components retain their own original licenses and copyrights.

The repository license does **not** relicense third-party software such as:

```text
OpenCore
Lilu
WhateverGreen
OpenCore Legacy Patcher
Skyfall
macOS / Apple system components
```

Preserve the appropriate upstream copyright and license notices when redistributing third-party components.

---

# Disclaimer

This is an **experimental community project**.

It is:

```text
not supported by Apple
not an official OCLP release
not endorsed by Dortania
not endorsed by Acidanthera
not an official Skyfall release
```

You are modifying the boot configuration of unsupported hardware.

Back up your data and keep a known-good EFI before testing.

---

# Current Status

```text
macOS Tahoe 26.7              ✅
MacBook9,1                    ✅
Intel HD Graphics 515         ✅ 1536 MB
Full graphics acceleration    ✅
Liquid Glass                  ✅
Wi-Fi                         ✅
Audio                         ✅
Daily desktop use             ✅
```

From:

```text
"Can Tahoe even work properly on this Mac?"
```

to:

```text
"It actually works."
```

Enjoy the 12-inch. :)
