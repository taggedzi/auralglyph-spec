# v0.1 Specification Review Register

**Updated:** 2026-10-15  
**Current specification:** [AuralGlyph v0.1 working draft](../spec/auralglyph-spec.md)

This register is informative. It records the rationale for the consolidation and the remaining work needed before freezing an interoperable specification. New clarifications in this working draft are proposed text for steward review, not a claim of prior approval.

## Consolidation decisions

| Topic | Treatment in the working draft | Basis |
| --- | --- | --- |
| Authoritative document | One current entry point, with linked normative sections; versioned drafts are historical | June TODO requested integration of modular sections |
| One scalar stream per CAS | Preserved, including logical sensors and decoded channels | November single-sensor decision and June audio examples |
| Conformance levels | Descriptive, interpretable, and reconstructable requirements govern the core as well as the transform and bands | June sections 4 and 5 |
| Coefficient meaning | Complex reconstruction model retained; magnitude-only and other declared derived encodings allowed at appropriate levels | June §4.6 and §5.9 |
| Physical and symbolic axes | Physical coordinates required only for the claims that use them; unknown values remain unknown | June conformance and band decisions |
| Band vocabulary | `band_id`, `band_label`, `bin_count`, and `frequency_axis` replace older illustrative names in current text | June section 5; migration table added to §3.5 |
| Timing | Observation time, sensor latency, integration windows, and propagation delay remain distinct | June section 6 and sensor-network example |
| Interpretation and views | Read-only views, explicitly derived processing, downstream interpretation | Original core and June container section |
| Physical serialization | Deferred; AGF v0.1 specifies a logical container | Latest §7.1; resolves the stale choice in the old TODO |

## Clarifications introduced for review

These tighten internal consistency without attempting to design a complete binary or numerical profile:

- CAS metadata has a common identity minimum plus conditional conformance requirements. The current `cas_version` and `agf_version` value is the specification line `"0.1"`; historical document revision labels remain separate.
- CAS-level coefficient encoding and scaling are explicit defaults for bands. Reconstructable bands declare their encoding directly. Other container/Session inheritance remains explicit.
- Repeated logical parameters must agree, including parameters shown at CAS scope and in `transform_parameters` or `time_axis`. There is no silent conflict precedence.
- Core indices are zero-based. A custom transform maps its native indices into that model.
- Reconstruction requires at least one reconstructable band. A bounded-loss claim must name its metric, reference signal, and bound.
- Containers contain or reference at least one CAS. Sessions have at least one member; stereo Sessions identify exactly two members and explicit channel roles.
- Static pose may be resolved through an explicit Session reference; moving pose requires a declared track format and time mapping. A physical microphone location is not required for an ordinary stereo-channel relationship.
- Partial examples are labeled as fragments. They do not establish a serialization or prove reconstruction.
- Custom invertible encodings remain possible through an explicit profile, while the baseline STFT retains its stricter complex-coefficient and phase requirements. Illustrative profile names do not supply undefined behavior.

## R1: Logical coefficient payload contract

**Status:** Open; needed before complete machine-validatable CAS objects can be specified.

Define the logical data object used by every band: axes and sizes, component ordering, permitted numeric types, finite/non-finite values, missing-data representation, and inline versus referenced data. Specify the relationship between `coefficient_shape`, `num_frames`, `bin_count`, encoding, and the payload.

The core already fixes the abstract band/frame/bin model. This item should define the remaining logical contract without prematurely choosing ZIP, directory, or binary storage. Byte order and byte-level packing belong to whichever physical binding is eventually selected.

**Completion evidence:** A complete single-CAS fixture can be interpreted without undocumented array or missing-value assumptions.

## R2: Metadata binding and complete examples

**Status:** Open; needed for executable validation and fixtures.

The current YAML is explanatory logical notation. Decide canonical nesting for transform parameters, time fields, sensors, references, and inline objects before publishing a machine-readable schema. Document any migration from the shorthand accepted in this draft.

Create a complete minimal CAS example and a complete logical container manifest once R1 is resolved. Add a reconstructable stereo fixture after R3. Validate the sensor-network example as a composed object; its current snippets are independent illustrations.

**Completion evidence:** Examples contain all required metadata, resolve all references, and pass a schema or equivalent validator for the chosen binding.

## R3: Baseline STFT conventions

**Status:** Open; needed for independent encoder/decoder interoperability.

`AGF-STFT-BASE-V0` currently lists parameters and reconstruction conditions. It does not yet assign complete numerical meaning to every parameter value. Specify:

- Analysis and inverse transform conventions, normalization, and coefficient ordering.
- Window definitions and parameters, including periodic versus symmetric windows where applicable.
- Frame reference point, boundary extension, padding amounts, frame count, and original sample count or equivalent trimming metadata.
- One-sided versus two-sided storage, retained bins, and treatment of DC and Nyquist where applicable.
- Stable overlap-add requirements and behavior for invalid window/hop combinations.
- Reconstruction coverage, numeric precision, and measurable error expectations.

The current draft requires an explicit supporting profile or extension for unresolved behavior before reconstruction is claimed. It does not pretend that the baseline identifier supplies those details.

**Completion evidence:** Two independently configured implementations agree on coefficients and reconstructed samples for published test vectors, including short signals and boundary cases.

## R4: Pose tracks and clock mappings

**Status:** Open for portable moving-sensor and drift-aware use.

The abstract sensor-pose and timing requirements are retained. Define or explicitly defer a common pose-track object, orientation representation, interpolation/extrapolation policy, time-standard conversion, drift mapping, and validity intervals. Until then, profiles or extensions must define any such behavior used by a consumer.

Clarify time uncertainty composition and whether a metadata binding will support irregular sampling directly or require preprocessing into a declared uniformly spaced signal.

**Completion evidence:** A moving sensor and a drifting clock can be aligned by a consumer without inferring coordinate or timing conventions.

## R5: Review and freeze

**Status:** Open.

Review the proposed clarifications above with the steward. Check normative wording, field vocabularies, example consistency, links, and GitHub math rendering. Decide which capabilities are required for the first frozen release and which require explicit custom profiles. Assign an unambiguous release revision when freezing.

**Completion evidence:** The [conformance checklist](../spec/sections/08-conformance.md) can be applied to complete fixtures, open limitations are reflected in release scope, and the steward approves the release.

## Deferred beyond the logical v0.1 scope

- Required physical serialization and compression formats.
- Streaming and network transport profiles.
- Standard renderers, projections, and Scene formats.
- Domain-specific interpretation or classification algorithms.

A reference encoder/decoder and automated schema validation are implementation work that can help resolve R1–R3. They are not present in this repository yet.
