# Design decisions

Running record of the design interview (2026-10-04). One entry per question: what was
decided and what follows from it. Hard-to-reverse calls also get an ADR in `adr/`;
vocabulary lives in `CONTEXT.md`; the facts behind the answers are in `research/`.

How each answer was reached:

- **Round 1 (Q1–Q9):** answered by the maintainer.
- **Rounds 2–3 (Q10–Q25):** the maintainer took the interviewer's recommendation.
- **Self-grill (Q26–Q125):** asked and answered by the interviewer, on the maintainer's
  instruction.

Tags:

- **[check]** rests on a fact that still has to be confirmed on hardware or in docs.
- **[yours]** commits the maintainer's money, time or taste, so it deserves his eyes.
- **[changed]** differs from an earlier recommendation.

## Round 1: roots

### Q1. Audience and ambition
**Decided:** The open control stack for the Topping family: community-grade, with
contributor-facing device definitions and the protocol spec as a reference.
**Follows:** Packaging, docs and safety defaults are built for strangers' hardware, not
only the maintainer's. See ADR 0004 for how this squares with "DX5 II first".

### Q2. What "way better" means
**Decided:** Five win conditions: speed (instant launch, no browser); scripting and
automation; PEQ workflow (curve editing, AutoEQ/REW import, A/B, a profile library);
Linux and macOS; openness (documented protocol, offline, no account). No specific
reliability bugs in Topping's tool were named.

### Q3. Surfaces
**Decided:** All of them are wanted: core library, CLI, desktop GUI, tray, TUI, browser
UI. Web technology inside a native window is acceptable only if it proves good. The order
and the cuts were settled in Q10 and Q12.

### Q4. Programmability
**Decided:** The CLI contract, resident features and embedded scripting are all wanted.
"Lightweight" describes the UI, not the absence of a background process.

### Q5. Core language
**Decided:** Rust (ADR 0001). The maintainer does not review the code; correctness rests
on tests, the simulator and the implementing agent.

### Q6. Budgets
**Decided:** Gates, to be measured on the maintainer's machine: a CLI read completes in
<= 100 ms end to end; the GUI is interactive <= 500 ms after a cold start, uses <= 150 MB
of memory, downloads at <= 30 MB and uses 0% CPU when idle. Stretch: 200 ms, 50 MB, 10 MB.
The gates may be stretched a little if a clearly better result needs it. Nothing has been
measured yet; Q111 says how they are enforced.

### Q7. Device scope
**Decided:** 1.0 supports the DX5 II only; other devices follow. Capabilities are data; no
plugin interface until a second real device exists (ADR 0004). Amended by Q16(d): a model
without a verified definition gets no protocol traffic at all.

### Q8. Write-safety contract
**Decided:** Accepted as proposed (ADR 0002), including: guards may block a script until
it passes an explicit flag; firmware update is out for good; unknown firmware is read-only
with an explicit per-firmware opt-in. The numbers were set in Q58–Q68.

### Q9. Platform verification
**Decided:** Windows and Linux can be verified on real hardware and are Tier 1. There is
no Mac, so macOS is Tier 2: built in CI, tested against the simulator, labelled
experimental until someone confirms it on hardware (ADR 0003).

## Round 2: shape

### Q10. The v1 cut
**Decided:** 1.0 is the core, the simulator, the `cherryctl` CLI and the desktop GUI with
the PEQ editor. CLI-only releases come first as 0.x. After 1.0, in order: resident mode
(tray, hotkeys, rules, local API), scripting, browser UI. The TUI is dropped unless a real
need to drive a headless box over SSH appears.

### Q11. Who owns the device
**Decided:** Everything works standalone and coordinates automatically: the CLI and the
GUI each work with nothing else running, and they work at the same time.
**[changed]** The mechanism was refined in Q51: each process opens the device itself and
takes a cross-process operation lock, instead of routing through an owner process
(ADR 0006).

### Q12. What the browser UI is for
**Decided:** A hosted page that talks to the DAC straight from the browser, as Topping's
Home WEB does. It comes last, and only if the bake-off picks a web UI, because only then
does it reuse the desktop UI. Serving the UI to other machines on the network is not
planned; if a concrete need appears it will be off by default and authenticated. The
README line becomes "no account, no server, works offline".

### Q13. What "good" means for the GUI shell
**Decided:** The toolkit is chosen by a measured bake-off: the same thin slice (window, EQ
curve with draggable bands, live volume readout against the simulator) built in two or
three stacks. A stack passes if cold start and memory are inside the Q6 gates on Windows
and Linux, the curve drag holds display refresh rate, text and window chrome do not look
foreign, and tray works. Tie-breaker: which one looks best to the maintainer. Candidates
and extra criteria: Q102, Q103.

### Q14. Aesthetic direction
**Decided:** A precision-instrument look: dark neutral surface, one accent colour, the EQ
curve as the hero of the main window, large numeric readouts with units, keyboard-first,
no skeuomorphic knobs. Mock-up directions come before any UI code (Q110). **[yours]**

### Q15. Scripting
**Decided:** An embedded Lua-family engine in resident mode, sandboxed, with every device
write passing the write-safety contract. Python stays supported from outside through
`cherryctl` and the local API. If the first automations are all "when X, apply profile
Y", rules cover them and the engine waits. Engine choice: Q118.

### Q16. Verification and reverse-engineering rules
**Decided (ADR 0005):**

1. No command is sent to a real device unless that exact command shape was seen in a
   capture of Topping's own software. No opcode scanning, no guessing.
2. Captures live in the repo with serials scrubbed, as ground truth. The core must decode
   every captured exchange and re-encode it byte for byte.
3. The simulator is built from the same spec and captures, and every surface is tested
   against it in CI on all three OSes.
4. Hardware runs happen only on the maintainer's go-ahead, reads first. A write command's
   first live run happens at low volume with nothing valuable connected, and restores the
   prior state.
5. Each release is verified on real hardware on Windows and Linux before it is called
   supported.

(a) Rule 1 is absolute. (b) The maintainer runs the capture sessions (method: Q70).
(c) The maintainer is present for the first live run of each new write command.
(d) A model without a verified definition gets no protocol traffic at all, not even reads.
**[yours]** (b) and (c) cost the maintainer's time.

## Round 3: scope and policy

### Q17. Feature scope for 1.0
**Decided:** The baseline is what Topping's Home WEB does for the DX5 II.

- **In 1.0:** volume, mute, input, output, headphone gain, balance, PCM filter, crossfeed,
  PEQ on/off; every device setting we can capture and verify; the 10-band PEQ with
  preamp; scene recall and save (Q47); a snapshot and restore of all readable state;
  lookup of published AutoEQ results by headphone name (an explicit network action,
  cached locally); import and export of EQ text files.
- **After 1.0:** fitting an EQ to a custom target curve; live meters on by default (Q113).
- **Never:** firmware update, factory reset, accounts, cloud sync, Topping's share codes
  (profiles are plain files instead), control over Bluetooth.

**[check]** The licence terms of the AutoEQ data before shipping the lookup.

### Q18. PEQ workflow for 1.0
**Decided:** A draggable curve on a log-frequency plot showing each band and the sum;
numeric entry with units; live preview on the device while dragging; automatic preamp
(Q65); undo and redo; level-matched A/B and bypass (Q100); a profile library of plain
files with tags and search; import of EqualizerAPO/AutoEQ and REW filter text; export to
the same; overlay of an imported measurement and target curve. No automatic curve fitting
in 1.0.

### Q19. State model
**Decided:** The device is the source of truth for everything it can report. cherrytop
never assumes a write took effect (Q36), and connecting never writes. For PEQ, the profile
files hold names and intent; cherrytop remembers a fingerprint of the last profile it
wrote and compares it with whatever can be read back. A profile it cannot confirm is shown
as "last written by cherrytop", not as fact. No automatic two-way sync.
**[check]** Whether a complete PEQ profile, including band gain, can be read back.

### Q20. License and provenance
**Decided (ADR 0007):** Code is dual-licensed MIT OR Apache-2.0. The protocol spec and
docs are CC BY 4.0. cherrytop's spec is written from our own captures of traffic on a real
device; prior-art documents are used only as leads for what to capture; nothing is copied
from Topping's web code or from GPL sources, and Topping's code is not read. Prior-art
authors are credited. This is not legal advice. **[yours]**

### Q21. Local API security
**Decided:** The local API (after 1.0) listens on an OS-local channel by default: a named
pipe on Windows, a Unix socket elsewhere, user-only. A network listener is a separate
opt-in: loopback only, bearer token, Host and Origin checks. Binding to the LAN is a
second opt-in. Every API write passes the write-safety contract, and nothing sent over
any API can raise the volume ceiling or the guard threshold.

### Q22. File formats and storage
**Decided:** TOML for config, profiles and snapshots, one profile per file, each with a
schema version and units in the key names (`freq_hz`, `gain_db`). Files live in the
platform's standard config and data directories. No database. The CLI's `--json` output
uses the same structures.

### Q23. Device contributions
**Decided:** A new model is supported only with three things: a device definition, a
capture set from that model with Topping's own tool, and a named verifier who ran the
hardware suite on it. Definitions record who verified them and on which firmware. The
definition format is frozen and documented when the second device arrives.

### Q24. Naming and command shape
**Decided:** Two binaries: `cherryctl` (CLI, no GUI code) and `cherrytop` (GUI). The CLI
uses plain verbs: `status`, `get`, `set`, `volume`, `mute`, `input`, `output`, `peq`,
`snapshot`, `apply`, `watch`, `devices`, `doctor`. Details: Q80–Q91.

### Q25. Distribution, signing and updates
**Decided:** GitHub Releases are the source: per-OS archives with checksums, built in CI.
Windows: winget and a portable zip; unsigned until 1.0, with the signing decision made
before the GUI ships. Linux: tarball, `.deb`, and a udev rule that packages install and
`doctor` explains. macOS: unsigned CI builds, labelled experimental; no Apple developer
account until there is a maintainer with a Mac. No auto-update and no update check unless
the user asks for one. **[yours]** Signing costs money.

## Self-grill: protocol core

### Q26. Is the protocol logic free of I/O?
**Decided:** Yes. A pure frame codec and a pure session state machine (bytes and time in,
bytes and events out). Transports are thin adapters. This is what makes the simulator,
capture replay and a later browser build cheap.

### Q27. Blocking or async?
**Decided:** A blocking API built on threads and channels. No async runtime in the core or
the CLI.

### Q28. How are units represented?
**Decided:** As integers in device units (0.1 dB, Hz, Q x 10^4) wrapped in distinct types.
Decimal input is parsed exactly. Floating point is used only for curve maths.

### Q29. What if a value is finer than the device can store?
**Decided:** An error by default. Rounding happens only when asked for (`--round`, or an
import report that lists every rounded value).

### Q30. How is preamp encoded?
**Decided:** As linear gain in 25-bit fixed point, per the prior art. The frame codec
carries the raw integer, so byte-for-byte replay never depends on a dB conversion.
**[check]** The vendor's value for -3.0 dB differs from the textbook formula by 27 counts,
and neither double nor single precision reproduces it (our calculation). Captures at
several preamp values will show the vendor's arithmetic.

### Q31. How is a device identified?
**Decided:** By vendor ID, product ID, exact USB product string, and the shape of
interface 2 (usage page, usage, 16-byte reports), all read from OS descriptors. Any
mismatch means "unsupported" and no traffic.

### Q32. How is the firmware version known before any traffic?
**Decided:** From the USB device revision, then cross-checked against the version in the
settings read. **[check]** That the mapping holds on firmware 2.53.

### Q33. Can the encoder produce a factory reset or firmware update frame?
**Decided:** No. Those commands do not exist in the encoder, and a test asserts their
command numbers can never be emitted.

### Q34. Are command tables loaded at run time?
**Decided:** No. They are declarative tables compiled into the binary. A loadable table
would be raw-opcode passthrough under another name. A developer-only flag exists for
people bringing up a new device on their own hardware.

### Q35. How are replies matched without transaction IDs?
**Decided:** One request in flight per device. A reply is matched by command number and
exact value; multi-record replies are assembled by their count and index bytes; stale
input is drained before each request.

### Q36. What confirms a write?
**Decided:** The device's echo must match what was sent. Hazardous writes are also read
back. Anything else is reported as "sent, not confirmed". **[check]** That the DX5 II
echoes writes on firmware 2.53.

### Q37. Own frame style or the vendor's?
**Decided:** Byte for byte what Topping's app sends for the same operation on the same
firmware, including its checksum habits and heartbeats.

### Q38. How are incoming frames validated?
**Decided:** Start bytes, end bytes and CRC are checked. Invalid frames are dropped and
counted, all-zero idle reports are ignored, and nothing acts on unvalidated data.

### Q39. How is device state modelled?
**Decided:** As typed fields where "unknown" is a real value. No invented defaults. Fields
are refreshed by reads and by notifications.

## Self-grill: device behaviour

### Q40. Push or poll for changes?
**Decided:** Push if the device broadcasts changes; otherwise poll at most once a second,
and only while something is watching. One-shot commands never poll.
**[check]** The two main sources disagree on whether the device broadcasts. A listen-only
test settles it.

### Q41. When is the heartbeat sent?
**Decided:** Only where captures show Topping's app needs it for that operation, and
continuously only while meters are displayed.

### Q42. How is the serial number handled?
**Decided:** It is read only to tell two units apart. It is never logged or exported,
stored only as a salted hash, and scrubbed from captures.

### Q43. Two devices at once?
**Decided:** Addressed with `--device`. Exactly one supported device is the default; more
than one is an error that lists them. Never a guess.

### Q44. What happens on unplug and replug?
**Decided:** The session ends on an I/O error. Reconnecting re-identifies the device and
re-reads its state. Nothing is written automatically on reconnect.

### Q45. What about standby?
**Decided:** Standby is reported as a state. There is no implicit wake; a command that
needs an awake device fails with a clear message. **[check]** How the device answers while
in standby.

### Q46. Settings that make the device re-enumerate (UAC mode)?
**Decided:** They are "disruptive writes": confirmation required, and the session expects
the disconnect.

### Q47. Scenes (C1/C2)?
**Decided:** Recall and save are both supported and both need confirmation. Save
overwrites a slot, and the level effect of a recall cannot be known in advance.

## Self-grill: transport and coordination

### Q48. Which HID library?
**Decided:** Start with `hidapi` (mature, opens shared on Windows). Evaluate `async-hid`
(pure Rust, hot-plug, no libudev) in a spike behind the same transport seam, and choose by
test on Windows and Linux. The choice is reversible because the seam hides it.

### Q49. Who handles the Windows report-ID byte?
**Decided:** The transport adapter adds and strips it. The codec only ever sees 16-byte
frames. A hardware test on each OS covers it, because the simulator sits above this layer.

### Q50. Which transport adapters exist?
**Decided:** Real HID, simulator and capture replay now; WebHID if the browser UI happens.
Three adapters make the seam real.

### Q51. How do the CLI and GUI work at the same time?
**Decided (ADR 0006):** Each process opens the device itself and takes a cross-process
operation lock: a named mutex on Windows, a file lock elsewhere. No owner process and no
IPC in 1.0. Another process's write shows up as an echo or notification, which keeps an
open GUI in step with the CLI. **[changed]** from "route through an owner".
**[check]** That every open handle receives every input report on Windows.

### Q52. Lock scope and crashes?
**Decided:** One lock per device, held for a whole operation (for example a muted PEQ
apply). An interrupted operation leaves a marker, so the next process warns and re-reads
state.

### Q53. When would an owner process be added?
**Decided:** Only if the two-handle test in Q51 fails, or if measurement shows the cost of
opening the device per call breaks the 100 ms budget.

### Q54. Other programs on the device (Topping's web app)?
**Decided:** They cannot be locked out. cherrytop detects foreign traffic (replies nobody
asked for) and warns; the docs say to close the tab.

### Q55. How is the device opened on macOS?
**Decided:** Shared, not seized, so cherrytop does not block other tools. **[check]**
Whether macOS shows an Input Monitoring prompt.

### Q56. How is the meter flood handled?
**Decided:** A reader thread decodes everything. Meter frames go to a latest-value mailbox
that drops old values; state and echo frames go to a queue that never drops. The OS input
buffer is raised to its maximum on Windows.

### Q57. Timeouts and retries?
**Decided:** About 250 ms per request, tuned from captures. Writes are never retried
automatically. A read may retry once.

## Self-grill: safety numbers and mechanics

### Q58. What counts as a large jump?
**Decided:** One rule for every write: an estimated level increase of more than 10 dB
needs confirmation. The number can be set from 3 to 99 dB, in the local config file only.
**[yours]** The number.

### Q59. What does the estimate cover?
**Decided:** Volume change, headphone gain switch, preamp change, PEQ bypass, line-out
mode switch (to DAC mode the increase equals the current attenuation) and output switch.
A write whose effect cannot be estimated is treated as hazardous.

### Q60. Are large changes ramped?
**Decided:** A confirmed large upward change ramps at 20 dB per second, which leaves time
to hit mute. Downward changes are immediate.

### Q61. Is there a volume ceiling?
**Decided:** Off by default and offered on first run. It is set only in the local config
file; nothing sent over an API or from a script can raise it.

### Q62. How is relative volume expressed?
**Decided:** With the words `up` and `down`. A bare signed number is always an absolute
level, so "-2" can never be read two ways.

### Q63. How is the volume unit resolved?
**Decided:** The device's volume-step setting is read once per operation, before any
absolute volume write, and never cached across operations. If it cannot be read, the
write is refused.

### Q64. How is PEQ headroom computed?
**Decided:** The summed response is evaluated on a dense grid at each supported
sample-rate family, and the worst peak is used. Peak plus preamp above 0 dB means "may
clip" and needs an override.

### Q65. Automatic preamp?
**Decided:** On by default: preamp is minus the peak, rounded toward more attenuation.
A manual value is allowed.

### Q66. How is a whole profile applied?
**Decided:** Mute, write, verify, unmute. If it fails midway the device stays muted and
the user is told. There is no gap-free mode in 1.0. **[check]** That mute over HID is
immediate and confirmable.

### Q67. How are live drag edits applied?
**Decided:** Streamed without muting. When a change raises the peak, preamp is lowered
first; when it lowers the peak, the band changes first.

### Q68. How does a user opt in to unverified firmware?
**Decided:** Per exact model and version, stored locally, after a compatibility read shows
that every setting parses within known ranges. Reads stay limited to the commands
Topping's app sends on connect.

### Q69. Is there a record of writes?
**Decided:** Yes. Every write is logged locally (time, surface, old value, new value)
with no serials, and can be shown with `cherryctl log`.

## Self-grill: verification

### Q70. How is traffic captured?
**Decided:** With a logging snippet of our own in Chrome's developer tools on Topping's
page, recording both directions with timestamps. No capture driver is installed; the
usual Windows capture driver has had no release since 2020.

### Q71. Capture format and storage?
**Decided:** JSON lines (time, direction, hex) with a header (firmware, web app version,
scenario), kept in the repo by model and firmware. Serial replies are replaced by a
placeholder and the header says so.

### Q72. What is captured?
**Decided:** One control at a time: every control and every value of every setting, plus
connect, idle, changes made with the knob and remote, and disconnect. The checklist lives
in the repo.

### Q73. Golden tests?
**Decided:** Every captured frame must decode and re-encode to identical bytes, and each
replayed scenario must end in the expected state.

### Q74. How faithful is the simulator?
**Decided:** Fed a capture's host frames, it must reproduce the device's frames, ignoring
meters and timing. A difference fails the build.

### Q75. Property tests?
**Decided:** The encoder emits only allowlisted shapes; decode and encode are inverses;
for random pairs of profiles the apply sequence never unmutes early; unit conversions are
exact.

### Q76. Is the decoder fuzzed?
**Decided:** Yes. Arbitrary 16-byte input must never cause a panic.

### Q77. Mutation testing?
**Decided:** On the safety and codec modules before each release: deleting a guard must
make a test fail. This stands in for the human review that does not happen.

### Q78. How does a hardware run work?
**Decided:** A separate runner. It refuses to start unless the volume is at or below
-60 dB and the operator types a confirmation. Reads come first; every write restores the
prior state; the run writes a report (firmware, OS, commit) into the repo.

### Q79. Is the EQ maths checked against the real device?
**Decided:** Yes, by a loopback sweep comparing the measured response with our curve, once
per filter type. **[yours]** It uses the maintainer's measurement rig and time.

## Self-grill: CLI

### Q80. Output modes?
**Decided:** Readable text on a terminal; `--json` with a versioned schema and units in
the key names.

### Q81. Exit codes?
**Decided:** A fixed table: 0 ok, 2 usage, 3 no device, 4 busy, 5 unsupported or
read-only, 6 guard refused, 7 device error, 8 timeout.

### Q82. What happens when a guard refuses?
**Decided:** On a terminal, a prompt. Otherwise exit 6 unless `--yes` was passed. No
environment variable can pre-approve.

### Q83. Declarative apply?
**Decided:** `apply -f` computes a plan (the differences, the write sequence, the
hazards). `--dry-run` prints it. Settings not named in the file are left alone.

### Q84. Snapshot?
**Decided:** `snapshot` writes every readable setting with the model and firmware.
`apply` refuses a snapshot from another model and warns on another firmware.

### Q85. Watch?
**Decided:** State changes as JSON lines. Meters only with `--meters`, at a capped rate.

### Q86. How are settings discovered?
**Decided:** `cherryctl keys` lists every setting with its type, unit, range and hazard
tag, generated from the device definition. The same data drives shell completions and
the docs.

### Q87. Doctor?
**Decided:** It diagnoses permissions, a missing udev rule, other programs or kernel
drivers holding the device, and firmware status. It prints the fix and changes nothing.

### Q88. Do profile tools need a device?
**Decided:** No. `peq check` (headroom report) and `peq convert` (import and export) work
offline.

### Q89. Frame tracing?
**Decided:** `--trace-frames` prints every frame in hex with the serial redacted, for bug
reports.

### Q90. Config precedence?
**Decided:** Flags, then environment, then the config file. Safety thresholds come only
from the config file.

### Q91. Stability promise?
**Decided:** The JSON schema and exit codes follow semantic versioning from 1.0. Before
that, anything may change.

## Self-grill: PEQ

### Q92. Filter maths?
**Decided:** Standard audio-EQ-cookbook biquads evaluated at the stream's sample rate.
This is an assumption until the measurement in Q79 confirms it. **[check]**

### Q93. Filter types in 1.0?
**Decided:** Peaking, low shelf and high shelf. Pass and notch filters are added once
their values are captured.

### Q94. Profile file?
**Decided:** One TOML file per profile: filters (type, `freq_hz`, `gain_db`, `q`),
`preamp_db` or "auto", channel mode, tags, source, schema version. Profiles are
device-independent.

### Q95. Profile against device limits?
**Decided:** Checked at apply time against the device definition (10 bands, gain, Q and
frequency ranges). Every violation is listed. Nothing is clamped or dropped silently.
**[check]** The exact ranges on the DX5 II.

### Q96. Left and right channels?
**Decided:** Linked by default. Separate per-channel editing is supported because the
device has separate registers.

### Q97. On-device preset slots?
**Decided:** 1.0 writes the active configuration only. Selecting and saving slots is added
when captured. **[check]**

### Q98. Memory modes?
**Decided:** They are shown, and apply says plainly when the device keeps a separate EQ
per output or input ("this changes the EQ for the headphone output only").

### Q99. Sample-rate limits?
**Decided:** Status shows when PEQ or crossfeed is inactive at the current sample rate.

### Q100. A/B comparison?
**Decided:** Two profiles and bypass, level-matched through preamp using a mid-band
loudness estimate. Switching uses the muted apply, so there is a short gap in 1.0.

### Q101. Flash wear?
**Decided:** Unknown, so: nothing writes periodically, live edits go no faster than
Topping's app sends them, and a hardware run tests whether settings survive a power
cycle. **[check]**

## Self-grill: GUI

### Q102. Toolkit candidates?
**Decided:** Slint, egui and Tauri 2 (system webview with a web UI), compared in the Q13
bake-off. The published figures (`research/gui-toolkits.md`) suggest Tauri will miss the
memory gate by about a factor of two, and that Slint and egui on an OpenGL renderer sit
near the stretch targets. Expected front-runner: Slint, which has a stable API, a built-in
tray and working accessibility. Its royalty-free license needs an attribution in the About
dialog. iced is the alternate. **[yours]** The Slint attribution, and whether Qt/QML
should be in the bake-off: it was left out without being researched in depth.

### Q103. Extra bake-off criteria?
**Decided:** Every build pins and records its renderer, because the renderer drives memory
more than the toolkit does. Memory is measured for the whole process tree, at idle.
Accessible names and roles are exposed for a button and a slider. The build is
reproducible in CI on all three OSes. UI logic can be tested headless against the
simulator.

### Q104. Main window?
**Decided:** The curve as the hero, with the band table beside it; a top bar with volume,
mute, output and gain; a profile library; a device panel. The volume control moves by drag
and by step only, with no click-to-jump. Mute is always visible.

### Q105. Settings view?
**Decided:** It mirrors the grouping of the device's own menu. Hazardous settings are
tagged. A banner shows when the device is read-only.

### Q106. Keyboard?
**Decided:** Every control is reachable. M mutes, Space switches A/B, Ctrl+Z undoes,
arrows nudge and Shift makes the nudge fine.

### Q107. How truthful are the readouts?
**Decided:** They show confirmed device state. A write that is pending or unconfirmed is
shown as pending, never as fact. Values are not animated.

### Q108. Demo mode?
**Decided:** The GUI can run against the simulator with no device, for exploring,
screenshots and UI tests.

### Q109. GUI process model?
**Decided:** The GUI opens the device itself under the operation lock. It is single
instance: a second launch focuses the first window.

### Q110. Mock-ups?
**Decided:** Two or three visual directions as static mock-ups, for the maintainer to
pick before UI code. Dark by default, light follows the OS, one accent colour, band
colours safe for colour-blind users. **[yours]**

### Q111. How are the Q6 budgets enforced?
**Decided:** By scripts that measure CLI latency against the real device during hardware
runs, and GUI cold start, memory and download size before each release. A miss blocks the
release or is recorded as an accepted stretch. **[check]** A full settings read is about
sixty frames, so `status` may not fit in 100 ms; a single write should.

### Q112. Undo?
**Decided:** Every applied state is kept in a session history, and "restore the state at
connect" is always available.

### Q113. Live meters?
**Decided:** A spectrum behind the curve and a level meter, off by default and remembered
once enabled. While off, no heartbeat is sent. **[check]** Whether the device streams
anyway while the interface is open; if it does, the idle-CPU gate becomes "under 1%".

## Self-grill: resident mode, rules and scripting (after 1.0)

### Q114. What form does resident mode take?
**Decided:** The GUI's "keep running in tray" option: a per-user process, not a system
service. Autostart is opt-in.

### Q115. Global hotkeys?
**Decided:** On Windows, macOS and X11 through the `global-hotkey` crate; on Wayland
through the desktop portal where the desktop supports it. The gaps are documented.

### Q116. Rules?
**Decided:** A TOML file of "when, do" entries. First triggers: device connected, output
or input changed, sample rate changed, hotkey. Triggers from the OS and from apps come
later.

### Q117. Can a rule or script confirm a hazardous write?
**Decided:** Only if that rule is marked pre-approved in the local file. Otherwise the
write is denied and the user is notified.

### Q118. Which scripting engine?
**Decided:** Luau through `mlua`, with its sandbox on and memory and time caps. Rhai (pure
Rust, sandboxed by design) is the fallback if the C build causes trouble. To be confirmed
when scripting is actually built.

### Q119. Script API?
**Decided:** Small: get, set, on(event), apply a profile, timers, log. No file or network
access.

## Self-grill: repo, build and release

### Q120. Workspace layout?
**Decided:** Crates for the core (pure), the HID transport and lock, the simulator, the
CLI and the app, plus `captures/`, `tools/` and `docs/`.

### Q121. Dependencies?
**Decided:** Few. Licences and advisories are checked in CI, the lockfile is committed,
and nothing GPL enters the core or CLI.

### Q122. Build targets?
**Decided:** Windows x64 and Linux x64 as Tier 1. Windows ARM64, Linux ARM64 and macOS as
Tier 2.

### Q123. Release gate?
**Decided:** A tag builds a draft release. It is published only when a hardware-run report
for that commit exists for Windows and Linux, and a fresh-context review agent has audited
the write path against ADR 0002.

### Q124. Network and telemetry?
**Decided:** No network calls unless the user asks for one. No telemetry and no crash
upload. `doctor --report` makes a redacted bundle the user can attach by hand.

### Q125. Rules for the agents that write the code?
**Decided:** An `AGENTS.md` in the repo carries the hardware rules: no frame goes to a
real device outside the gated runner, and never without the maintainer's go-ahead.
