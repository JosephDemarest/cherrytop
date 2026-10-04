# Processes coordinate through an operation lock, not an owner process

The CLI and the GUI must each work with nothing else running, and must work at the same
time. The protocol has no transaction IDs, so two programs talking at once cannot tell
their replies apart. We considered a single owner process that every other surface routes
through. We chose instead that each process opens the device itself and takes a
cross-process operation lock (a named mutex on Windows, a file lock elsewhere) for the
length of one operation.

A lock is needed in any case, for two CLI calls that run with no owner present. Once it
exists, routing through an owner adds an IPC layer and a process lifecycle without adding
correctness. Another process's write arrives as an echo or notification on every open
handle, which is what keeps an open GUI in step with the CLI.

## Consequences

- No IPC and no background process in 1.0.
- An interrupted operation leaves a marker, so the next process warns and re-reads state.
- Programs that are not cherrytop (Topping's web app in a browser) cannot be locked out.
  cherrytop detects foreign traffic and warns.
- This rests on every open handle receiving every input report. That is documented for
  Linux and expected on Windows; a two-handle hardware test confirms it. If the test
  fails, or if opening the device per call breaks the 100 ms CLI budget, an owner process
  is added behind the same transport seam.
