# Protocol notes

Status: nothing is decoded yet. This file separates what has been verified from what is
only suspected.

## Verified

From Windows device metadata on one DX5 II, 2026-10-04. Nothing was sent to the device.

- USB vendor ID `0x152A`, product ID `0x8750`, device revision (bcdDevice) `0x0253`.
- Composite device with three functions:
  - an audio function starting at interface 0;
  - a HID interface at interface 2 whose top-level collection is usage page `0x0001`,
    usage `0x0000`;
  - a HID consumer-control interface at interface 3 (usage page `0x000C`, usage `0x0001`).

## Hypotheses (unproven)

- Interface 2 carries the control protocol.
- The device revision `0x0253` corresponds to the firmware version.
