# DX5 II first; devices described as data, no plugin interface yet

The ambition is an open control stack for the Topping family, but v1 supports exactly one
device: the DX5 II. Its capabilities (band count, ranges, inputs) are described as data in
a device definition, so a second model is mostly a new table. We deliberately build no
device plugin interface until a second real device exists: with only one device behind
it, the shape of that interface would be a guess. Other models are detected and shown but
stay read-only.

## Amendment (accepted 2026-10-04)

"Read-only" is not safe enough for other models. Prior art documents a sibling model (the
DX1 II) on which a read request for an unlisted register acts as a write and reset a
user's settings, and several models share the DX5 II's product ID with different command
meanings (see `../research/prior-art.md`). So a model without a verified device
definition gets no protocol traffic at all. It is identified from its USB descriptors
only, and shown as unsupported. This replaces "stay read-only" above.

A new model becomes supported with three things: a device definition, a capture set from
that model with Topping's own tool, and a named verifier who ran the hardware suite on
it. Definitions are compiled into the binary; a loadable definition would be raw-opcode
passthrough under another name.
