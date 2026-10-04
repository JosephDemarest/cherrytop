# Protocol notes

Status: nothing is decoded by us yet. This file separates what we have verified from what
prior art reports and what is only suspected. The full prior-art survey, with sources, is
in [../research/prior-art.md](../research/prior-art.md).

## Verified by us

From Windows device metadata on one DX5 II, 2026-10-04. Nothing was sent to the device.

- USB vendor ID `0x152A`, product ID `0x8750`, device revision (bcdDevice) `0x0253`.
- Composite device with three functions:
  - an audio function starting at interface 0;
  - a HID interface at interface 2 whose top-level collection is usage page `0x0001`,
    usage `0x0000`;
  - a HID consumer-control interface at interface 3 (usage page `0x000C`, usage `0x0001`).

By calculation, 2026-10-04:

- The frame checksum is CRC-16/MODBUS over bytes 2–10, stored high byte first. Recomputed
  for four frames published by two independent sources; all four matched.
- Preamp is reported as linear gain in 25-bit fixed point, `round(10^(dB/20) * 2^25)`.
  The published value for −6.0 dB (`0x01009B9D`) matches that formula. The vendor's own
  value for −3.0 dB (`0x016A77C4`) does not: the formula gives `0x016A77DF` in double
  precision and `0x016A77DE` in single precision. The vendor's exact arithmetic is
  unknown; captures at several preamp values will show it.

## Reported by prior art (not yet verified on our unit)

- The control protocol runs over HID interface 2 with 16-byte input and output reports and
  no report IDs. Two independent sources print the same report descriptor.
- Frame layout: `22 33`, type, count, index, 16-bit command, signed big-endian 32-bit
  value, CRC, `66 77`, `00`.
- bcdDevice tracks the firmware version (`0x0239` on firmware 2.39).
- The most complete public spec was corrected against the code of Topping's own web app
  (web v1.10.0) and is no longer clean-room. That matters for how cherrytop may use it;
  see the license decision.

## Hypotheses (unproven)

- Our unit runs firmware 2.53. Supporting evidence: bcdDevice `0x0253`, the reported
  bcdDevice-to-firmware mapping, and Topping's latest DX5 II firmware being V2.53
  (released 2026/9/17). Not yet confirmed by reading the version from the device.
- The device broadcasts a frame on every state change. The two main sources disagree.

## Safety notes for whoever implements this

These come from the hazards documented in the prior-art survey.

- Identify a device by its USB product string as well as its IDs. The product ID is shared
  by models whose command numbers mean different things.
- Never send a read for a command that is not on the verified list for that exact model.
  On at least one sibling model an unlisted read acts as a write.
- Resolve the volume unit (0.5 dB or 1 dB per raw step) from fresh device state before
  every absolute volume write. A stale assumption can land louder than requested.
- Factory reset, firmware update and scene save are ordinary command numbers. The encoder
  must not be able to produce them.
- A write that returns no error has not necessarily taken effect. Check the echo, and read
  back where the protocol allows it.
