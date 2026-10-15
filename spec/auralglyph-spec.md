# AuralGlyph Specification — v0.1 Working Draft

**Status:** Experimental; consolidated for review, not a frozen interchange standard  
**Specification line:** v0.1  
**Draft revision:** 2026-10-15  
**Author and steward:** taggedzi (Matthew Craig)

This document and the linked normative sections below form the current AuralGlyph specification. The versioned files `auralglyph-spec-v0.1.md`, `auralglyph-spec-v0.1.1.md`, and `auralglyph-spec-v0.1.2.md` are historical drafts; they do not add requirements to this working draft.

The consolidation follows the June 2026 decisions on conformance levels, bands, Sessions, and logical containers. Unresolved implementation details are recorded in the [review register](../notes/spec-review.md). The draft does not yet define a portable coefficient file format or a fully specified baseline STFT interchange convention.

## Reading order and authority

| Section | Contents | Status |
| --- | --- | --- |
| 1–2, below | Scope, terminology, and requirement language | Normative |
| 3, below | CAS core model and metadata conventions | Normative except §3.7 |
| [4. Transform profiles](sections/04-transform-profiles.md) | Conformance levels, transform metadata, reconstruction | Normative except identified notes |
| [5. Band profiles](sections/05-band-profiles.md) | Band identity, mappings, coefficient meaning | Normative except identified notes |
| [6. Sessions and timing](sections/06-sessions-and-timing.md) | Membership, timing, spatial relationships | Normative except identified notes |
| [7. AGF container](sections/07-agf-container.md) | Packaging, references, provenance, extensions | Normative except identified notes |
| [8. Conformance checklist](sections/08-conformance.md) | Cross-section review and claim validation | Normative except readiness notes |
| Appendix A, below | Scenes and interpretive frameworks | Informative |

Examples illustrate the model and are non-normative. YAML fragments describe logical metadata; they do not establish a required YAML serialization or imply that omitted payloads exist. A fragment is not a complete conforming object unless explicitly identified as such.

Field templates containing `<type>` placeholders or `|` alternatives use text fences and are pseudocode. YAML fences contain literal metadata examples or fragments.

## 1. Scope

AuralGlyph represents scalar sampled signals as time–frequency data that can be visualized, analyzed, and, when sufficient information is preserved, reconstructed into time-domain waveforms. It supports audio, bioacoustics, sonar, vibration, seismic, biological, and other scalar sampled measurements.

The architecture separates three responsibilities:

- **CAS** represents one scalar signal stream and its coefficient data.
- **Sessions** describe relationships among independent CAS instances.
- **AGF containers** package CAS instances, Sessions, metadata, references, and related objects.

Views are derived from CAS without altering its coefficients. Processing that changes coefficients produces a new CAS instance or another explicitly derived product. Interpretation and rendering are downstream activities.

The v0.1 scope includes the logical model, metadata meaning, conformance levels, and requirements for reconstruction claims. It does not prescribe a binary format, compression, encryption, transport, renderer, classification system, or Scene format. Physical serialization profiles remain future work, as specified in §7.1.

### 1.1 Requirement language

**MUST** and **MUST NOT** express requirements. **SHOULD** and **SHOULD NOT** express recommendations whose omission requires a justified choice. **MAY** expresses permission. Lowercase uses of these words are ordinary prose.

Conformance is evaluated against the declared level and any applicable profile. Optional metadata MUST NOT be treated as known when absent. A requirement conditional on reconstruction, spatial analysis, or another capability applies only when that capability is claimed.

## 2. Terminology

| Term | Meaning |
| --- | --- |
| Signal stream | One real-valued scalar sample sequence from a physical or logical sensor. A multicomponent instrument produces multiple scalar streams. |
| Logical sensor | An identified source of a scalar stream, including a decoded audio channel, authored source, or derived virtual channel. Physical sensor details need not be known. |
| CAS | Canonical Audio Surface: coefficient data indexed by band, time frame, and band-local bin, with metadata appropriate to its conformance level. |
| Coefficient | A stored value associated with a frame and bin. Reconstructable transform coefficients retain the information required for inversion; descriptive or derived data may contain magnitude, power, or other declared values. |
| Band | An identified group of bins with declared meaning. A band may use physical frequencies, transform indices, symbolic positions, or unknown mappings. |
| Transform profile | A versioned definition of how coefficients relate to a scalar signal, including inverse behavior when reconstruction is claimed. |
| Conformance level | `descriptive`, `interpretable`, or `reconstructable`, as defined in §4.2. This describes available information, not scientific validity or synchronization quality. |
| Reconstruction expectation | The declared fidelity of inverse transformation: exact within numerical precision, bounded loss, approximate, or not reconstructable. |
| Session | Metadata describing membership and relationships among CAS instances without merging their signal data. |
| Observation time | The producer-declared receiving-sensor timeline. Its timestamp basis may describe acquisition time, reported time, or explicitly corrected time. |
| Sensor latency | Delay within the sensor or acquisition chain; distinct from travel time between source and sensor. |
| Propagation delay | Signal travel time from a source to a receiving sensor. Source-emission time is not implied by an observation timestamp. |
| Pose track | A referenced time sequence of sensor positions and, optionally, orientations. Its representation and interpolation require an explicit profile or extension until standardized. |
| View / projection | A mapping from CAS into a visual representation that does not mutate the source coefficients. |
| Derived product | Data produced by processing or analysis, with its relationship to source observations explicitly identified. |
| AGF container | The logical package of CAS instances and associated objects. Its physical storage is defined by a separate binding or future serialization profile. |
| Scene | An informative concept for a reproducible way to examine aligned data; outside the normative core. |

## 3. Canonical Audio Surface

A CAS represents exactly one scalar signal stream from one physical or logical sensor. Multi-channel, multi-sensor, and multi-component data MUST use multiple CAS instances; their relationships belong in Sessions.

Different representations or processing results of a stream MAY be separate CAS instances, with distinct `cas_id` values and explicit provenance. A consumer MUST NOT infer physical source identity from a channel role or an unknown sensor.

A CAS consists of:

1. Coefficient data organized into one or more bands.
2. One ordered time-frame axis shared by those bands.
3. Metadata describing each band's bins and values at the declared conformance level.
4. Transform metadata and a reconstruction expectation appropriate to that level.

The original complex-valued time–frequency model remains the model for reconstructable coefficients. Descriptive and interpretable CAS data may preserve less information, as explicitly permitted by §4 and §5. Missing phase is not equivalent to zero phase, and an unknown frequency axis is not an inferred physical frequency axis.

### 3.1 Indices and coefficient structure

Let $`B`$ be the positive integer number of bands, $`T`$ the positive integer number of frames, and $`F_b`$ the positive integer number of bins in band $`b`$.

```math
0 ≤ b < B,  0 ≤ t < T,  0 ≤ f < F_b
```

Here all indices are integers. The logical structure is a family of per-band matrices, or equivalently a ragged tensor:

```math
C[b,t,f] = C_b[t,f]
```

All bands MUST share `num_frames = T` and the same frame coordinates. Each band's `bin_count` MUST equal its $`F_b`$. Band ordering and identification are defined in §5.6–§5.7. Core indices are zero-based; a custom transform's native indices MUST be mapped explicitly to the core indices.

For complex coefficient encodings:

```math
C_b[t,f] ∈ ℂ
```

The encodings `complex-real-imag` and `complex-mag-phase` retain real/imaginary or magnitude/phase components, respectively. Other encodings are permitted under §4.6 with explicit meaning and reconstruction restrictions. A magnitude-only value MUST NOT be silently interpreted as a complete complex transform coefficient.

Each band MUST have an unambiguous effective coefficient encoding: an explicit band value, or an explicitly declared CAS default as described in §3.5. Physical packing, numeric types, byte order, and missing-value encodings are not fixed by this abstract tensor definition; a binding MUST specify them before interoperable payload decoding is claimed. See the [review register](../notes/spec-review.md).

### 3.2 Time axis

The frame index is contiguous and zero-based. All bands in a CAS refer to the same observation timeline; different CAS instances MAY have different frame rates or sample rates.

A descriptive CAS MUST declare `num_frames`, but MAY lack a physical time mapping. Consumers MUST NOT infer seconds from frame indices alone.

An interpretable CAS MUST declare `time_axis` using either sample-based spacing (`sample_rate_hz` and `hop_length_samples`) or direct spacing (`frame_spacing_sec`). Known sample rates and spacings MUST be positive. For either representation, the frame time relative to the declared origin is:

```math
τ(t) = τ_0 + t Δτ
```

Here $`τ_0`$ is `frame_zero_time_offset_sec`. For sample-based timing:

```math
Δτ = \frac{hop\_length\_samples}{sample\_rate\_hz}
```

For direct spacing, $`Δτ`$ is `frame_spacing_sec`. If both forms are supplied, they MUST agree. A known relative spacing does not establish an absolute origin. If the origin or offset is unknown, consumers may use time differences but MUST NOT assume cross-CAS alignment.

A reconstructable CAS MUST additionally supply the timing and analysis parameters required by its transform profile. For frame-based transforms these include window length, hop length, frame reference point, boundary behavior, and padding behavior. Frame reference points default to the nominal window center only where the declared profile supports that convention; consumers MUST NOT guess missing reconstruction parameters.

Sensor integration windows and transform analysis windows describe different operations. Sensor timing, latency correction, synchronization, and Session offsets follow §6. A CAS timestamp does not automatically specify source-event time.

### 3.3 Bands and frequency mappings

Each band MUST declare `band_id`, `band_label`, `band_role`, `bin_count`, and `reconstructable`. Additional requirements depend on conformance level as specified in §5.

When physical frequency is known, the `frequency_axis` metadata defines a mapping from band-local bin index to frequency in the declared unit. For nonnegative physical frequencies in Hz:

```math
φ_b(f) ∈ ℝ,  φ_b(f) ≥ 0
```

Transform-derived bands MUST declare how local bins map to the source transform whenever that mapping is needed for interpretation or inversion. Symbolic or unknown bands MUST NOT claim physical meaning unsupported by their metadata.

Bands MAY overlap. Their relationship MUST identify shared, duplicated, derived, or view data as specified in §5.8. When reconstruction is claimed, the transform profile MUST define how the reconstructable bands are combined without unintended double-counting.

Band labels such as `infrasound`, `audible`, or `ultrasound` are optional conveniences. Frequency metadata defines the actual coverage; no universal audio band split is required.

### 3.4 Transforms and reconstruction

A declared analysis operation maps a scalar time-domain sequence $`x[n]`$ into coefficients:

```math
C_b[t,f] = A_b(x)[t,f]
```

Where inversion is supported, the declared inverse produces $`\hat{x}[n]`$ from the required reconstructable bands. Its fidelity is determined by `reconstruction_expectation`, the transform profile, retained data, and numeric precision.

Every CAS MUST declare `cas_conformance_level`, `transform_type`, and `reconstruction_expectation`. `transform_type: "unknown"` is permitted only when reconstruction is not claimed. The parameters required at each level are defined in §4.2 and §4.5.

Reconstruction claims MUST be supported by the transform, coefficient encoding, scaling, band mapping, and available dependencies. They MUST identify any limited frequency coverage or approximate behavior; reconstructing a retained band does not imply recovery of omitted source content.

Decoded lossy source media can only be preserved relative to the decoded sample stream. A reconstruction claim MUST NOT imply recovery of information discarded before CAS creation.

### 3.5 Metadata conventions

The names below are the canonical logical vocabulary for this working draft. A future serialization binding MAY use different names or nesting only if it defines an explicit mapping that preserves these semantics. The YAML examples use this logical vocabulary.

**Identity and minimum content**

Each CAS in an AGF container MUST have a unique `cas_id`. Each CAS MUST declare `cas_version` (the targeted specification line, currently `"0.1"`), `cas_conformance_level`, `transform_type`, `coefficient_encoding`, `num_frames`, `bands`, and `reconstruction_expectation`. Conformance-level-specific requirements in §4 and §5 are additional to this minimum.

`signal_domain`, `sensor_id`, and `sensor_type` SHOULD be recorded when known. A sensor identifier denotes the physical or logical sensor within its declared scope; it is distinct from `cas_id`. A producer MUST NOT invent a physical sensor identity for decoded or authored channels. Unknown information MAY be omitted or explicitly marked unknown where the field permits it.

**Transform and time fields**

`transform_profile_id` identifies a known profile. `transform_parameters` holds its parameters. Existing examples also show parameters at CAS scope; these are shorthand for the same logical parameters, not independent overrides. If a parameter is repeated, values MUST agree. Conflicting declarations are non-conformant.

The same consistency rule applies to timing fields at CAS scope and within `time_axis`. Top-level `sample_rate_hz`, `analysis_window_length_samples`, and `hop_length_samples` remain the required declarations for the reconstructable frame-based profiles in §4.

**Coefficient and band fields**

CAS-level `coefficient_encoding` is an explicit default for bands that omit a band-level encoding. An explicit band encoding takes precedence for that band. CAS-level `amplitude_scale`, when present, is likewise an explicit default. Reconstructable bands MUST state their encoding explicitly and provide all scaling information needed for inversion. This is a core-defined default rule, not general metadata inheritance.

Canonical per-band names are `band_id`, `band_label`, `band_role`, `bin_count`, `frequency_axis`, `frequency_unit`, `frequency_min_hz`, `frequency_max_hz`, and `reconstructable`. Requirements and allowed omissions are defined in §5.

Historical names are mapped as follows for document migration; readers are not required to accept historical objects automatically:

| Historical draft term | Current logical vocabulary |
| --- | --- |
| `band_index` | Zero-based position determined by band order; identity is `band_id` |
| `band_name` | `band_label`, with a separate stable `band_id` |
| `num_bins` | `bin_count` |
| `f_min_hz`, `f_max_hz` | `frequency_min_hz`, `frequency_max_hz` |
| `bin_layout`, `bin_mapping_params` | `frequency_axis` with an explicit mapping |

**Sensor pose**

A CAS that claims spatial relationships MUST identify its coordinate frame and pose model, directly or through an explicit Session relationship. `sensor_pose_model` is `"static"`, `"time-varying-external"`, or `"unknown"`.

- Static pose requires `sensor_position` in the declared `reference_frame`, or an explicit Session reference that resolves it. Cartesian positions use `[x, y, z]`; geodetic positions require declared coordinates and units as illustrated in §6.11.
- Moving pose requires `sensor_pose_track_id` resolving to a track with a declared coordinate frame, time basis, and mapping to the CAS timeline. A profile or extension MUST define the track representation and interpolation before it is used for spatial processing.
- `sensor_orientation` is optional, but its representation, units, and coordinate convention MUST be declared when used.
- Unknown pose MUST NOT be interpreted as a known spatial relationship. Ordinary stereo channel roles do not require microphone positions.

**Time reference**

`time_reference_type` is `"absolute-utc"`, `"relative-session"`, or `"relative-arbitrary"`. `time_reference_value` supplies the origin where known; a claim of absolute UTC time requires that origin. `frame_zero_time_offset_sec` supplies the frame-zero offset where known. `time_sync_quality` records known synchronization quality without replacing the explicit Session relationships and uncertainties in §6.

Sessions may declare other time standards such as GPS or TAI. A mapping between differing time bases MUST be explicit before aligned time is claimed. Shared membership alone does not establish such a mapping.

**Units and unknown values**

Physical quantities MUST use SI units unless otherwise declared. Frequencies default to Hz, times to seconds, and Cartesian positions to meters. Explicitly named alternatives, such as `latitude_deg`, declare their units. Scaled encodings MUST define conversion to physical values where physical interpretation is claimed.

Absence or `null` MUST NOT be interpreted as zero. Missing metadata and missing coefficient samples are different conditions; coefficient missing-data behavior remains a binding requirement. Redaction and metadata scope follow §7.

### 3.6 Views and derived data

A projection maps coefficient data to images or geometry without changing the source CAS. A projection MAY be lossy as a display; a rendered image does not thereby retain enough information to reconstruct the signal.

Views MUST treat source coefficients as read-only. Filtering, masking, enhancement, or any other coefficient-changing operation MUST produce a new CAS instance or an explicitly derived data product. Its provenance SHOULD identify the source data and processing operation.

### 3.7 Multi-sensor use (informative)

Stereo, sensor arrays, moving sensors, and distributed stations use independent CAS instances joined by Sessions. Streams may have different sample rates, transforms, bandwidths, calibration, and clock behavior. Their individual metadata remains applicable when used outside a Session.

Spatial reconstruction, beamforming, source localization, and rendered audio are derived operations. A Session can preserve the relationships needed by these operations without asserting their conclusions. Source-preserving spatial audio can retain captured or authored streams for multiple renderers; it does not imply that a microphone recording isolates one acoustic source.

## Appendix A. Scenes and interpretive frameworks (informative)

A Scene is a reproducible description of how one or more Sessions are examined or experienced. It may select sensors, time ranges, frequency regions, spatial frames, annotations, or derived overlays. Given the same inputs and declared operations, it should support reproducing how the data was examined.

Scenes are outside the normative v0.1 specification. A Scene's existence does not establish causality, classification, explanation, or ground truth. Interpretations may be stored as derived products, but must remain distinguishable from observations. AuralGlyph preserves data and relationships so that different disciplines can apply their own transparent analysis and rendering methods.

## Authorship and AI assistance

The core concept and architectural direction originate with the project creator. AI tools have assisted with drafting, review, and organization. Proposed specification edits remain subject to steward review under [GOVERNANCE.md](../GOVERNANCE.md). Contributions follow [CONTRIBUTING.md](../CONTRIBUTING.md).
