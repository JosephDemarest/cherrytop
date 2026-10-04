# Platform support is defined by hardware evidence

A platform is Tier 1 only if a maintainer verified that release against a real device on
it. Otherwise it is Tier 2: built in CI, tested against the simulator, and labelled
experimental. Windows and Linux are Tier 1. macOS is Tier 2, because no maintainer has a
Mac to test on; we would rather ship "experimental" honestly than "supported" on faith.
