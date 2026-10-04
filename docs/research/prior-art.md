# Prior art: open-source control of the DX5 II

Research snapshot, 2026-10-04. Each claim is marked by how it was established:

- **[checked]** read in the linked source by us during this session;
- **[calculated]** recomputed by us;
- **[reported]** found by a research pass with the source linked, not re-read here.

## Projects

| Project | What it is | Language, licence | State |
|---|---|---|---|
| [gjcourt/toppingctl](https://github.com/gjcourt/toppingctl) | CLI for the DX5 II; D90 III Discrete and DX1 II are marked unverified | Python, Apache-2.0 | Created 2026-08-22, last commit 2026-09-30, no releases. Run on macOS and Linux only. DX5 II confirmed on firmware 2.39 and 2.46. [reported; device table checked] |
| [gjcourt/lab protocol spec](https://github.com/gjcourt/lab/blob/main/01-audio-midi/_reference/topping-dx5ii-hid-protocol.md) | The most complete public description of the protocol | Markdown | Began as observation of Topping's web app. On 2026-08-24 it was corrected against the constant tables in Topping's web bundle (v1.10.0), and it says itself that it is no longer clean-room. [checked] |
| [tcreswick/topping-dx5ii-linux](https://github.com/tcreswick/topping-dx5ii-linux) | Linux kernel patches that expose volume, mute, input, output, filter, balance and gain as ALSA mixer controls | C, GPL-2.0-only | Its protocol notes are black-box only: observed notifications, confirmed by replay. [checked] With its patches installed the kernel owns interface 2, so no other program can open it. [reported] |
| [WiiChef/topping-dx5-ii-power-toggle](https://github.com/WiiChef/topping-dx5-ii-power-toggle) | Power on/off only | C#, MIT | The only third-party client known to work on Windows. [reported] |
| [jeromeof/devicePEQ](https://github.com/jeromeof/devicePEQ) | WebHID PEQ tool for many devices | JavaScript, 0BSD | Topping support (DX1 II, E50 II) is disabled because another brand shares the vendor ID. [reported] |
| [ModerRAS/MiruPlay PR #83](https://github.com/ModerRAS/MiruPlay/pull/83) | Android port of toppingctl | Kotlin, GPL-3.0 | Merged with hardware testing still pending. [reported] |

## What the sources agree on

- Control runs over HID interface 2: 16-byte input and output reports, no report IDs, no
  feature reports. Both specs print the same report descriptor. [checked]
- Frame layout: `22 33`, a type byte, a count byte and an index byte, a 16-bit command
  (group and property), a signed big-endian 32-bit value, a CRC, `66 77`, `00`. [checked]
- The CRC is CRC-16/MODBUS over bytes 2–10, stored high byte first. We recomputed it for
  four published frames from the two independent sources and all four matched.
  [calculated]
- Vendor ID `0x152A` belongs to Thesycon (the XMOS USB audio stack) and is shared across
  brands. Product ID `0x8750` is shared by several Topping models; the DX1 II, E50 II and
  D90 III Discrete are named. [checked]
- bcdDevice tracks the firmware version: the spec author's unit read `0x0239` on firmware
  2.39. [checked] Ours reads `0x0253`.
- Volume is sent as attenuation. tcreswick gives the range as 0 to −99 dB. [checked]

## Where the sources disagree or stop

- **Change notifications.** tcreswick says the device broadcasts a frame on every state
  change from any origin (front panel, remote, Bluetooth, HID), and its driver depends on
  that. gjcourt's spec says passive listening shows no volume or PEQ state. Unresolved. A
  passive listen on our unit would settle it without sending anything. [checked both]
- **Reading PEQ back.** A bulk dump exists (write `12 06`; the records come back tagged
  `11 06`) that carries preamp and, per band, frequency, Q, type and the enabled flag. The
  spec could not find band gain in it. Topping's app evidently reads full state by some
  path. Open. [checked]
- **Byte 4 of single frames.** Vendor writes use `01` with a zero CRC; device
  notifications and tcreswick's commands use `00` with a real CRC. The meaning is not
  established. [checked both]
- **Persistence and flash wear.** Whether writes survive a power cycle is untested, and
  flash wear is not discussed anywhere. [checked for persistence; reported for wear]
- **Firmware 2.53 and Windows.** Nothing has been verified on firmware 2.53, and on
  Windows only the power toggle. [reported]

## Hazards the prior art documents

1. **Shared product ID, colliding command numbers.** The same command number means
   different things on models that share the product ID: `0x7113` is the home-page setting
   on a DX5 II and Bluetooth mode on an E50 II. [checked] The research pass also reports,
   from Topping's web bundle, that `0x712E` is FirmwareUpdate on a DX5 II and SaveC1 on an
   E50 II. [reported] Identification must use the USB product string; the product ID alone
   is unsafe.
2. **A read can act as a write.** On a DX1 II, a read request for a register outside the
   vendor's own query list is treated as a write of zero. A "read-only" sweep reset a
   user's gain setting. [checked]
3. **Destructive commands sit in the same space.** Factory reset (`0x710B`), firmware
   update (`0x712E`) and scene save (`0x7135`, `0x7136`) are ordinary commands. toppingctl
   hard-blocks all four. [checked]
4. **Silent acceptance.** A well-formed write produces no error even when it has no
   effect: toppingctl's volume writes to a D90 III were accepted and did nothing. Accepted
   writes are echoed back, and toppingctl does not check the echoes. [checked]
5. **Volume units depend on a setting.** The raw unit is 0.5 dB or 1 dB according to the
   device's volume-step setting, as measured on a DX5 II (raw 60 is −30.0 dB on the
   half-dB setting; raw 25 is −25.0 dB on the 1 dB setting). [checked] A client that
   assumes 1 dB while the device is on 0.5 dB lands at half the attenuation it asked for,
   which is louder: a request for −30 dB becomes −15 dB. [calculated]
6. **Metering flood.** The device streams VU and FFT frames at roughly 500 per second
   while a host sends heartbeats. [reported] gjcourt's spec describes a stream of about
   500 Hz whenever a host holds the interface open. [checked] Either way a client must
   filter it.
7. **No transaction IDs, shared access on Windows.** Frames carry no transaction ID, so
   replies match only by command number. [reported] hidapi opens HID devices in shared
   mode on Windows and exclusively by default on macOS. [checked in hidapi source] So on
   Windows two programs can talk at once and see each other's replies.
8. **The serial number is readable** (`0x7133`). Captures must be treated as sensitive.
   [checked]

## Capture method

Topping's web app speaks WebHID, so its traffic can be logged from the browser's developer
tools by wrapping `HIDDevice.prototype.sendReport`, with no USB capture driver. This is
how the gjcourt spec was started. [checked]

## Not verified

- Behaviour of any of this on firmware 2.53.
- Whether Topping's web app and a second program can hold the device at once on Windows.
- Whether change notifications need the heartbeat.
- Whether band gain, and so a complete PEQ profile, can be read back.
- PEQ filter type values 2 and 3 (probably the pass filters).
- Flash wear from repeated writes.
