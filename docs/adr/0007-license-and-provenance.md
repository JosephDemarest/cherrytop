# Permissive license; the spec comes from our own captures

Code is dual-licensed MIT OR Apache-2.0, the Rust ecosystem's norm, because the goal is an
open control stack that other projects can build on. The protocol spec and the docs are
CC BY 4.0, so others can reuse the spec with credit.

The most complete public protocol description says itself that it was corrected against
Topping's own web code and is no longer clean-room. The other main source is GPL-2.0-only.
To keep cherrytop's provenance simple, its spec is written from our own captures of
traffic on a real device. Prior-art documents are used only as leads for what to capture.
Nothing is copied from Topping's web code or from GPL sources, and Topping's code is not
read. Prior-art authors are credited.

This records an engineering policy, not legal advice.

## Consequences

- Every statement in the spec traces to a capture file in the repo.
- The capture work is needed anyway: nothing public has been verified on firmware 2.53.
- GPL-licensed dependencies cannot enter the core or the CLI.
