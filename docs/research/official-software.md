# Topping's official software for the DX5 II

Research snapshot, 2026-10-04. This is the baseline cherrytop has to match or beat.
Each claim is marked by how it was established:

- **[checked]** read on Topping's own page during this session;
- **[reported]** found by a research pass with the source linked, not re-read here;
- **[third-party]** stated by someone other than Topping.

## What exists

- **TOPPING Home WEB**, at https://home.toppingaudio.com. It supports the DX1 II, the
  DX5 II (firmware V2.39 or above) and the E50 II (firmware V1.56 or above). [checked] [1]
  - It runs in the browser over WebHID, and its troubleshooting panel recommends desktop
    Chrome or Edge. [reported, seen on web v1.14.0]
  - Login is optional (device binding, cloud backup, social features). No login step
    appears before device control. [reported]
  - It is a hosted site, so it needs internet to load. [reported] The author of
    toppingctl says it cannot be reached from some US ISPs. [third-party] [9]
- **TOPPING Home APP** (iOS and Android) supports the E50 II and the DX5 II (firmware
  V2.27 or above) and connects over Bluetooth. [reported] [3] [10]
- **Topping Tune** V1.16 (Windows 10/11, macOS 12+, no Linux). Its current compatibility
  list does not include the DX5 II, although the DX5 II manual still refers to it.
  [reported] [5] [6]
- Topping publishes no protocol documentation or SDK. [reported] [19]

## What Home WEB does

All [reported] from Topping's own descriptions [12] [13] unless marked.

- Volume, input switching and output switching.
- PEQ with up to 10 bands.
- AutoEQ with a built-in headphone database. It imports measurements as txt, csv or REW
  files (marked beta).
- Community presets exchanged by share code.
- C1/C2 scenes.
- Crossfeed, in a convolution mode and a BS2B-based mode.
- A firmware page that reads the model and version.
- [third-party] Preset names are kept in browser storage, not on the DAC, and the web app
  also offers pass and notch filters. [14]

## PEQ details from the Topping Tune V1.3 guide

[reported] [11]. That guide is older than the DX5 II, so these are likely rather than
confirmed for it.

- 10 bands. Filter types: Peaking, Low-pass, High-pass, Low-shelf, High-shelf.
- Gain ±12 dB, Q 0.1–15, preamp ±12 dB. The frequency range is not documented.
- One EQ for both channels, or separate left and right EQs.
- Up to 5 custom profiles are stored on the device; others live on the computer.
- Imports target curves, source frequency responses and filter parameters.

## DX5 II behaviour a control app must respect

[reported] from the DX5 II manual V1.6 [6] unless marked.

- PEQ can be switched on and off and has 5 built-in presets plus 5 custom ones.
- PEQ, volume and crossfeed each have a memory mode: Follow Output (the default), Follow
  Input, or Disabled. That is how the device keeps a separate EQ per output or per input.
- PEQ works on USB up to 192 kHz/32-bit, S/PDIF up to 192/24 and Bluetooth up to 96/24.
  Crossfeed works only at 44.1–48 kHz.
- Volume step is 0.5 dB (the default) or 1 dB. Channel balance goes up to 9.5 dB on either
  side. Headphone gain is Low or High.
- **Line-out mode is Preamp (variable, the default) or DAC (fixed at 0 dB, available only
  when line output alone is selected).** Switching to DAC mode sends full level to
  whatever is connected, so it is a hazardous write under ADR 0002.
- Other settings: PCM filters F-1 to F-8; UAC1/UAC2; Bluetooth and aptX toggles; S/PDIF
  mode; polarity; home screen Normal, VU or FFT; themes; brightness with Auto; VU 0 dB
  reference; 12V trigger in/out; assignable knob and remote A/B keys; DC-detect
  sensitivity; language; factory reset.
- Hardware: dual ES9039Q2M DACs, XMOS XU316 USB, QCC5125 Bluetooth 5.1. Inputs are USB,
  optical, coax and Bluetooth; outputs are 6.35 mm, 4.4 mm and 4-pin XLR headphone jacks
  plus XLR and RCA line out. [15]

## Firmware

- The latest version is V2.53, released 2026/9/17. [checked] [16] Earlier public versions
  include 1.39, 1.69, 2.07 and 2.31. [reported]
- Updates do not go through the control protocol: you hold the knob while switching the
  power on, the device appears as a USB drive, and you copy a `.Topping` file onto it.
  [checked] [16]
- The V2.53 changelog includes a fix for audio briefly becoming louder when previewing
  AutoEQ, and a fix for data occasionally failing to synchronise between the Home app,
  Home WEB and the device. [checked] [16] cherrytop has to design against both failure
  modes (ADR 0002 rule 3, and the state model).

## What users complain about

All [reported], paraphrased.

- PEQ profiles saved with Topping Tune were lost after a power cycle. [17]
- Topping Tune exports CSV but imports only .txt. [17]
- A firmware update wipes all user settings. [17] [7]
- No Linux software: a Linux reviewer had to move the DAC to a Windows PC to change the
  EQ (May 2026, before Home WEB supported the DX5 II). [8]
- The drivers would not install on Windows for ARM. [17]
- The Home APP is rated poorly, with reports of unstable connections. [10]
- No complaints specific to Home WEB were found; it gained DX5 II support only in
  September 2026.

## Not verified

- Whether Home WEB works fully without logging in, and whether it has any offline mode.
- Whether Home WEB works on Linux. Not tested.
- Home WEB's filter types and ranges, and the PEQ frequency range in any tool.
- The DX5 II's volume range. The manual does not give it.
- Whether the built-in presets can be edited. The DX5 II manual and the Tune guide
  disagree.
- Whether Topping Tune V1.16 still controls a DX5 II.

## Sources

- [1] https://www.toppingaudio.com/download/topping-home-web-web-control-center
- [3] https://www.toppingaudio.com/download/topping-home-app
- [5] https://www.toppingaudio.com/download/topping-tune-peq-tuning-software
- [6] https://dl.topping.audio/um/dx5_ii.pdf
- [7] https://www.pragmaticaudio.com/reviews/2025/09/topping-dx5-ii/
- [8] https://delightlylinux.wordpress.com/2026/05/26/the-topping-dx5-ii-dac-headphone-amp-and-linux/
- [9] https://github.com/gjcourt/toppingctl
- [10] https://apps.apple.com/us/app/topping-home/id6751623327
- [11] https://dl.topping.audio/um/TOPPING_Tune_V1.3.pdf
- [12] https://www.topping.store/blogs/news/topping-home-web-a-new-way-to-experience-topping
- [13] https://www.topping.store/blogs/news/major-update-topping-dx5-ii-gets-a-new-web-based-control-experience
- [14] https://github.com/gjcourt/lab/blob/main/01-audio-midi/_reference/topping-dx5ii-hid-protocol.md
- [15] https://www.topping.store/products/topping-dx5-ii-hi-res-dac-headphone-amp-combo
- [16] https://www.toppingaudio.com/download/dx5-ii-version-v2-53-firmware-update
- [17] https://www.audiosciencereview.com/forum/index.php?threads/topping-dx5-ii.60996/page-52
- [19] https://www.toppingaudio.com/supports?tab=drivers-software
