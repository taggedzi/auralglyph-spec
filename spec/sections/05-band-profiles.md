#### 5. Band Profiles

---

Summary: This document defines how bands organize coefficient data within Canonical Audio Surfaces (CAS) instances. Each band has an identity, frequency mapping, and conformance requirements based on the CAS's declared level of reconstruction support. It covers various roles for bands such as physical-frequency ranges, transform-derived groups, and perceptual groupings. The document also provides examples of standard band profiles and how to mark derived or summary bands. Finally, it outlines rules for band conformance and the distinction between metadata-defined meanings and labels that help human interpretation.

---

This section is **normative** unless otherwise noted.

A band profile defines how a subset of CAS frequency indices is named, described, and interpreted.

Bands allow a CAS instance to organize coefficient data into meaningful regions without changing the underlying CAS model. A band may correspond to a physical frequency range, a perceptual range, an instrument-specific range, a symbolic category, or a transform-specific group of bins.

AuralGlyph does not require all CAS instances to use the same band layout. However, every CAS instance MUST define enough band metadata for its declared conformance level.

---

##### 5.1 Band Identity

Each CAS band MUST declare:

```yaml
band_id: <string>
band_label: <string>
band_role: <string>
```

Where:

* `band_id` is a stable identifier unique within the CAS instance.
* `band_label` is a human-readable label.
* `band_role` describes the intended meaning of the band.

Examples:

```yaml
band_id: "audible"
band_label: "Audible Range"
band_role: "physical-frequency-range"
```

```yaml
band_id: "low-frequency-vibration"
band_label: "Low Frequency Vibration"
band_role: "domain-specific"
```

```yaml
band_id: "unknown-band-0"
band_label: "Unknown Band 0"
band_role: "unknown"
```

A producer SHOULD use stable, descriptive identifiers when band meaning is known.

A consumer MUST NOT infer scientific, perceptual, or physical meaning from a band label alone.

---

##### 5.2 Band Frequency Mapping

Each band SHOULD define how its internal frequency indices map to physical frequencies or symbolic frequency positions.

For bands with known physical frequency meaning, the band SHOULD declare:

```yaml
frequency_unit: "Hz"
frequency_min_hz: <number>
frequency_max_hz: <number>
frequency_axis: <object>
```

The `frequency_axis` object SHOULD describe how band indices map to frequency values.

Examples of frequency-axis metadata include:

```yaml
frequency_axis:
  mapping: "linear"
  bin_count: <integer>
  first_bin_hz: <number>
  last_bin_hz: <number>
```

```yaml
frequency_axis:
  mapping: "explicit"
  bin_frequencies_hz: [<number>, <number>, ...]
```

```yaml
frequency_axis:
  mapping: "transform-derived"
  source_transform: "AGF-STFT-BASE-V0"
  fft_bin_start: <integer>
  fft_bin_count: <integer>
```

If the band does not have known physical frequency meaning, it MUST NOT claim a physical frequency range.

Such bands MAY use symbolic or transform-specific metadata:

```yaml
frequency_axis:
  mapping: "symbolic"
  labels: ["low", "mid", "high"]
```

or:

```yaml
frequency_axis:
  mapping: "unknown"
```

---

##### 5.3 Band Conformance Requirements

Band metadata requirements depend on the CAS conformance level.

###### 5.3.1 Descriptive CAS Bands

A band in a descriptive CAS MUST declare at least:

```yaml
band_id: <string>
band_label: <string>
band_role: <string>
bin_count: <integer>
reconstructable: false
```

A descriptive band MAY omit exact frequency mapping.

If approximate frequency information is known, it SHOULD be included.

Example:

```yaml
band_id: "approx-audible"
band_label: "Approximate Audible Range"
band_role: "approximate-frequency-range"
bin_count: 512
frequency_unit: "Hz"
frequency_min_hz: 20
frequency_max_hz: 20000
reconstructable: false
```

###### 5.3.2 Interpretable CAS Bands

A band in an interpretable CAS MUST declare enough metadata for consumers to understand what the band represents.

An interpretable band MUST declare at least:

```yaml
band_id: <string>
band_label: <string>
band_role: <string>
bin_count: <integer>
frequency_axis: <object>
reconstructable: <boolean>
```

If `reconstructable` is `true`, the band MUST meet the reconstructable band requirements in Section 5.3.3.

If physical frequencies are known, the band SHOULD declare:

```yaml
frequency_unit: "Hz"
frequency_min_hz: <number>
frequency_max_hz: <number>
```

If physical frequencies are not known, the band MUST declare a symbolic, transform-derived, or unknown mapping.

###### 5.3.3 Reconstructable CAS Bands

A band marked:

```yaml
reconstructable: true
```

MUST provide enough information to map its coefficients back into the transform representation required for inverse reconstruction.

A reconstructable band MUST declare:

```yaml
band_id: <string>
band_label: <string>
band_role: <string>
bin_count: <integer>
frequency_axis: <object>
coefficient_shape: <object>
coefficient_encoding: <string>
reconstructable: true
```

For transform-derived bands, the frequency axis MUST specify how band-local indices map back to transform indices.

Example:

```yaml
frequency_axis:
  mapping: "transform-derived"
  source_transform: "AGF-STFT-BASE-V0"
  fft_bin_start: 0
  fft_bin_count: 1025
```

A reconstructable band MUST NOT discard coefficient data required by the declared inverse transform unless the transform profile explicitly permits that loss and declares the reconstruction expectation accordingly.

A reconstructable band MUST NOT use approximate-only, visual-only, or symbolic-only frequency mapping unless the transform profile defines a valid inverse using that mapping.

---

##### 5.4 Standard Band Roles

AuralGlyph v0.1 defines the following standard band roles:

```yaml
band_role: "physical-frequency-range"
```

The band corresponds to a known physical frequency range.

```yaml
band_role: "transform-derived"
```

The band corresponds to a range or set of bins from a declared transform.

```yaml
band_role: "perceptual"
```

The band represents a perceptual grouping, such as a musical or hearing-related range.

```yaml
band_role: "domain-specific"
```

The band is meaningful within a particular domain, instrument, or analysis workflow.

```yaml
band_role: "symbolic"
```

The band uses labels or categories rather than physically meaningful frequency values.

```yaml
band_role: "visualization"
```

The band is intended primarily for display.

```yaml
band_role: "unknown"
```

The producer does not know the band meaning.

Consumers MUST treat unknown or symbolic bands cautiously and MUST NOT infer physical frequency meaning unless the metadata explicitly provides it.

---

##### 5.5 Standard Frequency Range Labels

AuralGlyph v0.1 provides optional standard labels for common frequency ranges.

These labels are **not mandatory** and do not replace explicit frequency metadata.

For audio-like signals, producers MAY use:

```yaml
band_id: "infrasound"
band_label: "Infrasound"
frequency_max_hz: 20
```

```yaml
band_id: "audible"
band_label: "Audible"
frequency_min_hz: 20
frequency_max_hz: 20000
```

```yaml
band_id: "ultrasound"
band_label: "Ultrasound"
frequency_min_hz: 20000
```

For vibration, seismic, sonar, radio, biological, or other domains, producers SHOULD prefer explicit frequency metadata over audio-centric labels.

The labels `infrasound`, `audible`, and `ultrasound` are convenience labels only. A consumer MUST rely on declared frequency values, not labels alone.

---

##### 5.6 Band Ordering

A CAS instance MUST define the order in which bands appear in the coefficient structure.

Band order MAY be declared explicitly:

```yaml
band_order:
  - "low"
  - "mid"
  - "high"
```

Or it MAY be implied by the order of band declarations in the CAS metadata.

If band order matters for decoding, rendering, or reconstruction, it MUST be unambiguous.

A consumer MUST NOT assume that bands are ordered from low to high frequency unless the metadata declares that ordering or provides frequency mappings that establish it.

---

##### 5.7 Band Indexing

Within each band, frequency indices MUST be zero-based unless a transform profile explicitly defines otherwise.

A band-local frequency index is written:

```text
k_band
```

A CAS-wide frequency index, if used, is written:

```text
k_cas
```

For transform-derived bands, the metadata MUST define the relationship between band-local indices and transform indices.

Example:

```yaml
band_id: "stft-positive-frequencies"
frequency_axis:
  mapping: "transform-derived"
  fft_bin_start: 0
  fft_bin_count: 1025
```

This means:

```text
fft_bin_index = fft_bin_start + k_band
```

for `0 <= k_band < fft_bin_count`.

Profiles MAY define more complex mappings when needed, but the mapping MUST be explicit at the declared conformance level.

---

##### 5.8 Overlapping Bands

Bands MAY overlap.

Overlapping bands are useful when the same coefficient data is exposed through different views, ranges, or interpretive groupings.

If bands overlap, the CAS metadata MUST state whether the overlapping bands:

* reference the same underlying coefficient data,
* duplicate coefficient data,
* derive new coefficient data,
* or provide interpretive views over another band.

Example:

```yaml
band_id: "speech-range"
frequency_min_hz: 300
frequency_max_hz: 3400
relationship:
  type: "view-of"
  source_band_id: "audible"
```

A band that is only a view or derived summary MUST NOT be required for reconstruction unless the transform profile explicitly defines it as part of reconstruction.

---

##### 5.9 Derived and Summary Bands

A band MAY contain derived or summary information.

Examples include:

* magnitude-only bands,
* averaged power bands,
* perceptual bands,
* spectral centroid tracks,
* feature bands,
* visualization bands,
* ML embedding bands.

Such bands MUST declare:

```yaml
derived: true
reconstructable: false
```

unless the transform profile explicitly defines a valid reconstruction method using that derived data.

Derived bands SHOULD declare their source when known:

```yaml
derived: true
source_band_id: "audible"
derivation: "magnitude-only"
```

A consumer MUST NOT treat derived or summary bands as equivalent to original transform coefficients unless the metadata explicitly states that they are equivalent.

---

##### 5.10 Band Units and Scaling

Each band SHOULD declare the units and scaling of its coefficient values.

Examples:

```yaml
coefficient_unit: "linear-amplitude"
amplitude_scale: "linear"
```

```yaml
coefficient_unit: "relative-power"
amplitude_scale: "log10"
reference_value: <number>
```

```yaml
coefficient_unit: "arbitrary"
amplitude_scale: "unknown"
```

A reconstructable band MUST declare enough unit and scaling information to invert the coefficient representation required by the transform profile.

A descriptive or interpretable band MAY use arbitrary or unknown units, but consumers MUST treat such data as non-reconstructable unless a valid inverse is declared.

---

##### 5.11 Band Profiles

A band profile is a reusable definition for a common band layout or band type.

A band MAY declare:

```yaml
band_profile_id: <string>
```

Standard AuralGlyph band profiles SHOULD use the form:

```text
AGF-BAND-<NAME>-V<MAJOR>
```

Examples:

```text
AGF-BAND-AUDIO-AUDIBLE-V1
AGF-BAND-STFT-POSITIVE-FREQUENCIES-V1
AGF-BAND-SEISMIC-LOW-FREQUENCY-V1
```

Custom band profiles SHOULD use identifiers beginning with:

```text
custom:
```

Example:

```text
custom:taggedzi.experimental-bioacoustic-band-v0
```

A band profile MUST NOT override explicit metadata declared on the band unless the profile defines how conflicts are resolved.

If a consumer does not understand a band profile, it MAY still interpret the band using explicit metadata present on the band.

---

##### 5.12 Example Band Definitions

###### 5.12.1 Descriptive Audio-Like Band

```yaml
band_id: "display-band-0"
band_label: "Display Band 0"
band_role: "visualization"
bin_count: 512
frequency_axis:
  mapping: "unknown"
coefficient_encoding: "magnitude-only"
amplitude_scale: "log"
derived: true
reconstructable: false
```

###### 5.12.2 Interpretable Audible Band

```yaml
band_id: "audible"
band_label: "Audible"
band_role: "physical-frequency-range"
bin_count: 1024
frequency_unit: "Hz"
frequency_min_hz: 20
frequency_max_hz: 20000
frequency_axis:
  mapping: "linear"
  first_bin_hz: 20
  last_bin_hz: 20000
coefficient_encoding: "complex-mag-phase"
amplitude_scale: "linear"
derived: false
reconstructable: false
```

###### 5.12.3 Reconstructable STFT Band

```yaml
band_id: "stft-positive-frequencies"
band_label: "STFT Positive Frequencies"
band_role: "transform-derived"
band_profile_id: "AGF-BAND-STFT-POSITIVE-FREQUENCIES-V1"
bin_count: 1025
frequency_unit: "Hz"
frequency_axis:
  mapping: "transform-derived"
  source_transform: "AGF-STFT-BASE-V0"
  fft_bin_start: 0
  fft_bin_count: 1025
coefficient_shape:
  axes: ["time_frame", "frequency_bin", "complex_component"]
  complex_components: ["real", "imaginary"]
coefficient_encoding: "complex-real-imag"
amplitude_scale: "linear"
derived: false
reconstructable: true
```

###### 5.12.4 Domain-Specific Seismic Band

```yaml
band_id: "seismic-low-frequency"
band_label: "Seismic Low Frequency"
band_role: "domain-specific"
bin_count: 256
frequency_unit: "Hz"
frequency_min_hz: 0.01
frequency_max_hz: 20
frequency_axis:
  mapping: "logarithmic"
  first_bin_hz: 0.01
  last_bin_hz: 20
coefficient_encoding: "complex-mag-phase"
amplitude_scale: "linear"
derived: false
reconstructable: false
```

---

##### 5.13 Band Conformance

A CAS band conforms to its declared band metadata only if:

1. all fields required for the CAS conformance level are present,
2. `band_id` is unique within the CAS instance,
3. `bin_count` matches the coefficient data,
4. frequency mappings are explicit at the declared conformance level,
5. coefficient encoding is declared,
6. derived bands are clearly marked,
7. reconstructable bands provide enough mapping information for inverse transformation,
8. and unknown or approximate meanings are clearly identified.

An implementation MUST reject, warn on, or mark as non-conformant any band that claims reconstructability without sufficient metadata.

A consumer MAY display or inspect a non-conformant band, but MUST NOT silently treat it as reconstructable or scientifically precise.

---

##### 5.14 Non-Normative Notes

Bands are organizational and interpretive structures layered over CAS coefficient data.

They are not required to correspond to human hearing, musical pitch, or audio-specific frequency ranges.

AuralGlyph should support audio, vibration, sonar, seismic, biological, radio, and other scalar sampled signals without forcing all domains into audio-centric assumptions.

Band metadata describes how coefficient data is organized along frequency or symbolic axes. It does not define when an observed signal originated at its source. Timing, sensor latency, synchronization, and propagation-delay metadata belong to CAS timing metadata and Session metadata.

The safest general rule is:

```text
labels help humans;
metadata defines meaning.
```
