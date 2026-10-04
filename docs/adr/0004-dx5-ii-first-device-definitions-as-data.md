# DX5 II first; devices described as data, no plugin interface yet

The ambition is an open control stack for the Topping family, but v1 supports exactly one
device: the DX5 II. Its capabilities (band count, ranges, inputs) are described as data in
a device definition, so a second model is mostly a new table. We deliberately build no
device plugin interface until a second real device exists: with only one device behind
it, the shape of that interface would be a guess. Other models are detected and shown but
stay read-only.
