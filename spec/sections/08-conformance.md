# 8. Conformance Checklist

Part of the [current v0.1 working specification](../auralglyph-spec.md).

## Document summary

- **Purpose:** Bring the core, transform, band, Session, and container requirements into one review checklist.
- **Status:** Normative except §8.7, which describes draft readiness.
- **Applies to:** Producers, consumers, logical objects, and declared capability claims.
- **Does not define:** A physical serialization, executable schema, or numerical test suite.

## 8.1 Scope of a conformance claim

A conformance claim MUST identify the specification line and the profiles or extensions required to interpret the data. CAS claims MUST identify `cas_conformance_level` and `reconstruction_expectation`. An implementation MUST distinguish logical metadata conformance from support for a particular physical serialization or inverse transform.

Conformance to this draft does not by itself establish measurement accuracy, scientific validity, synchronization, complete provenance, or successful waveform reconstruction. Each additional claim MUST have the metadata and data required by the applicable sections.

## 8.2 Core CAS checklist

A CAS MUST satisfy all applicable requirements in §3, including:

- It represents one scalar signal stream; channel and component relationships are external to the CAS.
- `cas_id` is unique within its container, and `cas_version` identifies the targeted specification line.
- `cas_conformance_level`, `transform_type`, `coefficient_encoding`, `num_frames`, `bands`, and `reconstruction_expectation` are present.
- Frame and bin counts are positive integers and agree with the available data.
- All bands use the same frame axis. Band order and local indices are unambiguous.
- Effective encoding and scaling are explicit wherever required; duplicated parameter declarations agree.
- Physical units, timing, and sensor-pose claims have the metadata they require. Unknown information is not replaced with invented values.
- Processing results and lossy views are distinguishable from the source coefficients.

## 8.3 Transform and band checklist

The following summarizes §4 and §5; it does not replace their conditional requirements.

| Declared CAS level | Time meaning | Band meaning | Reconstruction claim |
| --- | --- | --- | --- |
| `descriptive` | Ordered frames; physical time may be unknown | Identity, role, count; physical frequency may be unknown | `not-reconstructable`; bands are non-reconstructable |
| `interpretable` | Explicit sample-based or seconds-based frame spacing | Declared physical, transform-derived, symbolic, or explicitly unknown mapping | No guaranteed inverse; any approximate or per-band inverse claim requires a defined method and sufficient parameters |
| `reconstructable` | All timing required by the inverse | Explicit mappings, shapes, encodings, and invertible scaling for reconstructable bands | At least one reconstructable band and all data, parameters, and dependencies required for the declared inverse |

For each CAS and its bands, verify:

- The transform profile is identified when required. Its fixed requirements and configurable values are respected.
- Every band has a unique `band_id`, a `band_label`, a `band_role`, a `bin_count`, and a `reconstructable` flag.
- Required frequency mappings and coefficient shapes are present and consistent with the actual coefficients.
- Derived bands and overlapping data have explicit relationships; reconstruction does not accidentally double-count overlapping coefficients.
- Retained phase, scaling, boundary behavior, padding, normalization, and dependencies support each reconstruction claim.
- `lossy-bounded` specifies an error metric, reference signal, and tolerance. Exact and partial-band claims describe their scope without implying recovery of unavailable source data.
- Unknown or unsupported custom behavior is identified. An identifier alone is not proof that an inverse is available.

## 8.4 Session checklist

A Session MUST satisfy §6, including:

- It has `session_id`, `session_type`, and a nonempty `members` list.
- Its identifier is unique within the container, and each member resolves to an existing CAS or an explicitly declared external CAS dependency.
- A stereo Session identifies two members with distinct left and right roles.
- Synchronization and offset claims are explicit and supported by the time references and uncertainty information available to the producer.
- CAS origins, frame-zero offsets, and Session offsets are composed consistently without applying corrections twice.
- Sensor latency, integration windows, and propagation delay retain their distinct meanings.
- Spatial and moving-pose claims resolve to declared reference frames and sufficient pose metadata or profiles.
- Derived interpretations remain distinguishable from observations.

## 8.5 Container checklist

An AGF container MUST satisfy §7, including:

- `agf_version` is present and at least one CAS is contained or explicitly referenced.
- CAS and Session identifiers are unique in their respective container scopes.
- Claimed multi-stream relationships have Sessions. Unrelated packaged streams are not implicitly synchronized or co-located.
- Object references, metadata scopes, and inheritance rules are unambiguous.
- Required external dependencies are declared. Missing dependencies are reported as completeness or conformance problems.
- Derived products, unknown metadata, redactions, and extensions are clearly identified.

The manifest, provenance, and integrity recommendations in §7 also apply. A complete reconstruction claim requires its dependencies to be available, even if an external reference is syntactically valid.

## 8.6 Consumer behavior

A consumer MAY inspect incomplete, non-conformant, or unsupported data. It MUST distinguish these conditions from verified support and MUST NOT silently promote a CAS to a stronger conformance level.

Consumers MUST NOT infer missing phase, unknown units, physical frequency from labels, shared clocks from membership, source time from observation time, or physical sensor identity from channel roles.

When validation is incomplete, the consumer MUST limit its claims to what it has established. Metadata inspection alone does not demonstrate a successful inverse or verify the fidelity of reconstructed samples.

## 8.7 Draft readiness (informative)

This checklist makes the current logical model reviewable. It is not a declaration that the specification is ready for interoperable implementations.

The [review register](../../notes/spec-review.md) tracks the remaining logical payload contract, STFT conventions, pose and clock mapping details, complete fixtures, and validation work. Physical serialization remains deliberately deferred. A future freeze should identify its exact document revision and the supported capabilities rather than treating every illustrative profile identifier as an implemented standard.
