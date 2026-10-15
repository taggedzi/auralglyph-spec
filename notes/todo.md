# AuralGlyph v0.1 TODO

The [current specification](../spec/auralglyph-spec.md) is consolidated for review. The [review register](spec-review.md) contains rationale, scope, and completion criteria for the remaining work.

## Completed in the consolidation draft

- [x] Establish one authoritative specification entry point with linked normative sections.
- [x] Reconcile the CAS core with the June conformance levels and band vocabulary.
- [x] Clarify metadata defaults, parameter conflicts, and reconstruction claim requirements.
- [x] Add a cross-section conformance checklist.
- [x] Retain §7.1's decision to defer physical serialization.
- [x] Update README, terminology, historical-draft notices, and example status.

## Before a frozen interoperable specification

- [ ] R1: Define logical coefficient payload shapes, numeric types, missing values, and inline/reference semantics.
- [ ] R2: Select a metadata binding and complete the single-CAS and manifest examples.
- [ ] R3: Finish baseline STFT numerical, boundary, and original-length conventions; add reconstructable stereo test vectors.
- [ ] R4: Define or explicitly defer portable pose-track and clock-drift mappings.
- [ ] R2/R5: Validate the composed sensor-network example and all complete fixtures.
- [ ] R5: Review the new clarifications with the steward, verify rendering, and assign a frozen release revision.

## Implementation work

- [ ] Validation schema and automated fixture validation.
- [ ] Reference encoder/decoder and numerical round-trip tests.

## Deferred profiles

- Physical serialization profiles, including binary packing and byte order.
- Streaming / networking transport profiles.
- Rendering profiles.
- Scene / interpretation framework.
