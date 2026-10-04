# Protocol notes

Status: nothing is decoded yet. This file separates what has been verified from what is
only reported or suspected.

## Verified

From Windows device metadata on one DX5 II, 2026-10-04. Nothing was sent to the device.

- USB vendor ID `0x152A`, product ID `0x8750`, device revision (bcdDevice) `0x0253`.
- Composite device with three functions:
  - an audio function starting at interface 0;
  - a HID interface at interface 2 whose top-level collection is usage page `0x0001`,
    usage `0x0000`;
  - a HID consumer-control interface at interface 3 (usage page `0x000C`, usage `0x0001`).

## Reported by third parties (not yet verified by us)

- The control protocol runs over HID on interface 2, with 16-byte reports and no report
  IDs. Source: gjcourt's protocol notes,
  https://github.com/gjcourt/lab/blob/main/01-audio-midi/_reference/topping-dx5ii-hid-protocol.md
- Those notes say parts of them were taken from the code of Topping's own web app
  (web v1.10.0). That matters for how cherrytop may use them; see the license decision.

## Hypotheses (unproven)

- The device revision `0x0253` is the firmware version. Supporting evidence: Topping's
  latest DX5 II firmware is V2.53 (released 2026/9/17), which matches `0x0253` read as
  binary-coded decimal. It has not been confirmed by reading the version from the device.
