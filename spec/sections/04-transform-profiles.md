#### 4. Transform Profiles

This section is **normative** unless otherwise noted.

A transform profile defines how a scalar time-domain signal is converted into a Canonical Audio Surface (CAS), and how reconstructable CAS bands may be converted back into a time-domain signal.

AuralGlyph does not require all CAS instances to use the same transform. However, every CAS instance MUST declare enough transform information for its stated conformance level.

A CAS intended only for visualization or approximate analysis may require less transform metadata than a CAS intended for full waveform reconstruction.

---

##### 4.1 Transform Profile Identity

Each CAS instance MUST declare a transform profile using the following fields:

- `transform_type` — the general transform family, such as `"stft"`, `"cqt"`, `"wavelet"`, `"custom"`, or `"unknown"`.
- `transform_profile_id` — a profile identifier naming a standard or custom transform profile, when known.
- `transform_parameters` — a structured object containing transform parameters known to the producer.

A transform profile identifier SHOULD be stable and versioned. Standard AuralGlyph profiles SHOULD use the form:

```text
AGF-<TRANSFORM>-<PROFILE>-V<MAJOR>
```

Examples:

```text
AGF-STFT-BASE-V0
AGF-STFT-AUDIO-48K-V1
AGF-STFT-ULTRA-192K-V1
AGF-CQT-MUSIC-V1
AGF-WAVELET-SEISMIC-V1
```

Custom profiles MAY use implementation-specific identifiers, but MUST include sufficient metadata to interpret the transform at the declared conformance level.

If the transform is not known, `transform_type` MAY be `"unknown"`, but the CAS MUST NOT claim reconstructability.

---

##### 4.2 CAS Transform Conformance Levels

Transform metadata requirements depend on the declared conformance level of the CAS instance.

A CAS instance MUST declare:

```yaml
cas_conformance_level: "descriptive" | "interpretable" | "reconstructable"
```

The declared level tells consumers what they may safely assume.

###### 4.2.1 Descriptive CAS

A **descriptive CAS** records a time-frequency surface with minimal metadata.

It is suitable for:

* visual display,
* experimental captures,
* rough inspection,
* archival notes,
* AI/ML input where exact physical reconstruction is not required.

A descriptive CAS MUST declare at least:

```yaml
cas_conformance_level: "descriptive"
transform_type: <string or "unknown">
coefficient_encoding: <string>
num_frames: <integer>
bands: <array>
reconstruction_expectation: "not-reconstructable"
```

Each band SHOULD include approximate frequency information when available.

A descriptive CAS MUST NOT claim that it can reconstruct the original time-domain signal.

###### 4.2.2 Interpretable CAS

An **interpretable CAS** provides enough metadata for consumers to understand the time and frequency meaning of the CAS coefficients.

It is suitable for:

* scientific inspection,
* comparison across captures,
* visualization with meaningful axes,
* ML datasets,
* partial or approximate analysis.

An interpretable CAS MUST declare at least:

```yaml
cas_conformance_level: "interpretable"
transform_type: <string>
coefficient_encoding: <string>
sample_rate_hz: <number or null>
num_frames: <integer>
time_axis: <object>
bands: <array>
amplitude_scale: <string>
reconstruction_expectation: "not-reconstructable" | "approximate"
```

The `time_axis` metadata MUST define either:

* `hop_length_samples`, when sample-rate based timing is known, or
* `frame_spacing_sec`, when frames are already expressed in seconds.

Each band MUST define enough frequency metadata to map CAS frequency indices to approximate physical frequencies or declared symbolic bins.

An interpretable CAS MAY be approximate. It does not need to preserve all information required for inverse reconstruction.

###### 4.2.3 Reconstructable CAS

A **reconstructable CAS** provides enough metadata and coefficient data to reconstruct a time-domain signal for all bands marked `reconstructable: true`.

It is suitable for:

* signal interchange,
* archival preservation,
* round-trip encoding,
* playback,
* high-confidence scientific reuse.

A reconstructable CAS MUST declare:

```yaml
cas_conformance_level: "reconstructable"
transform_type: <string>
transform_profile_id: <string>
transform_parameters: <object>
sample_rate_hz: <number>
analysis_window_length_samples: <integer>
hop_length_samples: <integer>
num_frames: <integer>
coefficient_encoding: "complex-real-imag" | "complex-mag-phase"
amplitude_scale: <string>
bands: <array>
reconstruction_expectation: "exact-within-numerical-precision" | "lossy-bounded" | "approximate"
```

For frame-based transforms, a reconstructable CAS MUST also define:

```yaml
window_function: <string>
window_parameters: <object or null>
boundary_mode: <string>
padding_mode: <string>
normalization: <string>
```

For STFT-like transforms, it MUST also define:

```yaml
fft_size: <integer>
phase_unit: "radians"
```

A reconstructable CAS MUST NOT omit phase information unless the declared transform profile defines a valid reconstruction method without stored phase.

---

##### 4.3 General Transform Requirements

For every CAS instance, the declared transform metadata MUST be sufficient for the declared `cas_conformance_level`.

A producer SHOULD provide as much transform metadata as it reasonably knows.

A consumer MUST NOT assume missing metadata.

A CAS band marked `reconstructable: true` MUST have a transform definition sufficient to perform a numerically stable inverse operation within the limits of the transform.

A CAS band marked `reconstructable: false` MAY contain derived, lossy, visual-only, analysis-only, or summary data. Such a band MUST NOT be required for reconstructing the source time-domain signal.

If any required reconstruction metadata is missing, the affected band MUST be treated as non-reconstructable.

---

##### 4.4 Analysis and Inverse Relationship

A transform profile defines an analysis operation:

```math
C = T(x)
```

where:

* `x` is the scalar time-domain signal,
* `T` is the declared analysis transform,
* `C` is the resulting CAS coefficient structure.

For reconstructable bands, the profile also defines an inverse operation:

```math
x_hat = T^{-1}(C)
```

where `x_hat` is the reconstructed time-domain signal.

The inverse MAY be exact or approximate, depending on the transform profile. The profile MUST state its reconstruction expectation using one of the following values:

* `"exact-within-numerical-precision"`
* `"lossy-bounded"`
* `"approximate"`
* `"not-reconstructable"`

If the reconstruction expectation is `"lossy-bounded"`, the profile SHOULD define an error metric or tolerance.

If the reconstruction expectation is `"not-reconstructable"`, consumers MUST NOT present the CAS as a reversible encoding of the source signal.

---

##### 4.5 Required Transform Parameters

Required transform parameters depend on conformance level.

###### 4.5.1 Descriptive Minimum

A descriptive CAS MUST provide enough information to identify the data as a CAS-like time-frequency surface.

Minimum fields:

```yaml
cas_conformance_level: "descriptive"
transform_type: <string or "unknown">
coefficient_encoding: <string>
num_frames: <integer>
bands: <array>
```

###### 4.5.2 Interpretable Minimum

An interpretable CAS MUST provide enough information to map frame and frequency indices to meaningful coordinates.

Minimum fields:

```yaml
cas_conformance_level: "interpretable"
transform_type: <string>
coefficient_encoding: <string>
num_frames: <integer>
time_axis: <object>
bands: <array>
amplitude_scale: <string>
```

The `time_axis` object SHOULD include one of:

```yaml
hop_length_samples: <integer>
sample_rate_hz: <number>
```

or:

```yaml
frame_spacing_sec: <number>
frame_zero_time_offset_sec: <number>
```

###### 4.5.3 Reconstructable Minimum

A reconstructable CAS MUST define all parameters needed to interpret and invert the transform.

For frame-based transforms, this includes at least:

```yaml
sample_rate_hz: <number>
analysis_window_length_samples: <integer>
hop_length_samples: <integer>
num_frames: <integer>
window_function: <string>
window_parameters: <object or null>
boundary_mode: <string>
padding_mode: <string>
normalization: <string>
coefficient_encoding: "complex-real-imag" | "complex-mag-phase"
amplitude_scale: <string>
```

For frequency-domain transforms, this includes at least:

```yaml
frequency_layout: <string>
bin_mapping_params: <object>
frequency_unit: "Hz"
phase_unit: "radians"
```

For STFT-like transforms, this also includes:

```yaml
fft_size: <integer>
```

The default physical units are:

* frequency in Hertz,
* time in seconds,
* phase in radians,
* sample counts as dimensionless integers.

Profiles MAY define additional parameters when required.

---

##### 4.6 Coefficient Encoding

A transform profile MUST declare how complex coefficients are stored.

Valid coefficient encodings for v0.1 are:

* `"complex-real-imag"`
* `"complex-mag-phase"`

For `"complex-real-imag"`, each coefficient is represented as:

```text
(real, imaginary)
```

For `"complex-mag-phase"`, each coefficient is represented as:

```text
(magnitude, phase_radians)
```

A profile MUST define whether magnitude values are linear, power-scaled, logarithmic, or otherwise transformed.

If logarithmic or relative scaling is used, the profile MUST specify the reference value and conversion rule back to linear magnitude when reconstruction is claimed.

A descriptive or interpretable CAS MAY use non-reconstructable coefficient encodings, but such encodings MUST NOT be marked reconstructable unless their inverse meaning is fully defined.

---

##### 4.7 Boundary and Padding Rules

A reconstructable transform profile MUST define how the beginning and end of the source signal are handled.

The `boundary_mode` field SHOULD use one of the following values unless a custom profile defines otherwise:

* `"none"` — no implicit extension of the signal.
* `"zero-pad"` — missing samples are treated as zero.
* `"reflect"` — missing samples are reflected from the signal boundary.
* `"wrap"` — missing samples wrap around from the opposite boundary.
* `"custom:<profile-id>"` — behavior is defined by a custom profile.

The profile MUST also state whether frame times correspond to:

* window start,
* window center,
* or another explicitly defined reference point.

For v0.1, CAS frame time SHOULD be interpreted as the nominal center time of the analysis frame unless the transform profile explicitly declares otherwise.

Descriptive and interpretable CAS instances SHOULD declare boundary and padding rules when known, but they MAY omit them if reconstruction is not claimed.

---

##### 4.8 Baseline STFT Profile: AGF-STFT-BASE-V0

AuralGlyph v0.1 defines one baseline transform profile:

```text
AGF-STFT-BASE-V0
```

This profile is intended to provide a minimal interoperable STFT-based transform for scalar sampled signals across domains.

###### 4.8.1 Profile Identity

```yaml
transform_type: "stft"
transform_profile_id: "AGF-STFT-BASE-V0"
```

###### 4.8.2 Required Parameters

A reconstructable `AGF-STFT-BASE-V0` CAS instance MUST declare:

```yaml
sample_rate_hz: <number>
analysis_window_length_samples: <integer>
hop_length_samples: <integer>
fft_size: <integer>
window_function: <string>
window_parameters: <object or null>
boundary_mode: <string>
padding_mode: <string>
normalization: <string>
coefficient_encoding: "complex-real-imag" | "complex-mag-phase"
amplitude_scale: <string>
phase_unit: "radians"
```

The following constraints apply:

* `sample_rate_hz` MUST be greater than 0.
* `analysis_window_length_samples` MUST be greater than 0.
* `hop_length_samples` MUST be greater than 0.
* `fft_size` MUST be greater than or equal to `analysis_window_length_samples`.
* `phase_unit` MUST be `"radians"`.
* `window_function` MUST be declared explicitly.

###### 4.8.3 Frequency Bins

For an STFT profile, frequency bins are derived from `sample_rate_hz` and `fft_size` unless overridden by an explicit band mapping.

For a one-sided real-input STFT, the bin center frequency is:

```math
f_k = \frac{k \cdot sample\_rate\_hz}{fft\_size}
```

where `k` is the FFT bin index.

A band MAY expose all FFT bins or a subset of bins. If a band exposes a subset, its metadata MUST define how CAS frequency indices map back to FFT bin indices or physical frequencies.

###### 4.8.4 Reconstruction

An `AGF-STFT-BASE-V0` CAS instance MAY be reconstructable when:

* all reconstructable bands together preserve the required STFT bins,
* the original phase information is available,
* the window and hop configuration support stable overlap-add reconstruction,
* the boundary and padding behavior are known,
* and coefficient scaling is reversible.

A profile using `AGF-STFT-BASE-V0` MUST declare its reconstruction expectation as one of:

```yaml
reconstruction_expectation: "exact-within-numerical-precision" | "lossy-bounded" | "approximate" | "not-reconstructable"
```

If the CAS omits phase, stores only magnitudes, applies irreversible scaling, drops bins required for reconstruction, or stores only visual/summary data, the relevant bands MUST be marked `reconstructable: false`.

---

##### 4.9 Custom Transform Profiles

A custom transform profile MAY be used when a standard profile is insufficient.

A custom profile MUST define enough metadata for its declared conformance level.

For descriptive CAS, a custom profile MAY be minimal.

For interpretable CAS, a custom profile MUST define:

* time-frame meaning,
* frequency-bin meaning,
* coefficient encoding,
* amplitude scale,
* and known limitations.

For reconstructable CAS, a custom profile MUST define:

* transform identity,
* all parameters required to interpret the coefficients,
* coefficient encoding,
* frequency mapping,
* time-frame mapping,
* reconstruction behavior,
* boundary and padding rules,
* normalization rules,
* and extension compatibility rules.

Custom profiles SHOULD use identifiers beginning with:

```text
custom:
```

Example:

```text
custom:taggedzi.experimental-wavelet-v0
```

Consumers that do not understand a custom transform profile MUST NOT claim full reconstruction support for that CAS. They MAY still display metadata or render views when sufficient view metadata is available.

---

##### 4.10 Transform Profile Conformance

A CAS instance conforms to its declared transform profile only if:

1. all fields required for its declared `cas_conformance_level` are present,
2. all declared values are internally consistent,
3. every reconstructable band has sufficient information for inverse transformation,
4. coefficient encoding is explicitly declared,
5. frequency-bin mappings are unambiguous at the declared conformance level,
6. frame-time mappings are unambiguous at the declared conformance level,
7. and unsupported custom behavior is clearly identified.

An implementation MUST reject, warn on, or mark as non-conformant any CAS instance that declares a transform profile but omits required parameters for its declared conformance level.

A consumer MAY still display or inspect a non-conformant CAS, but MUST NOT silently promote it to a higher conformance level.

---

##### 4.11 Non-Normative Notes

Different transforms are useful for different domains:

* STFT is broadly useful for audio, vibration, sonar, and general sampled signals.
* CQT may be useful for music and perceptual pitch layouts.
* Wavelet transforms may be useful for seismic, biological, transient, or multi-scale signals.
* Custom transforms may be useful for specialized instruments or research domains.

AuralGlyph standardizes how transform information is declared. It does not require all domains to use the same transform.

The conformance-level model is intended to keep AuralGlyph usable by both non-expert creators and expert scientific or archival systems. A simple tool may produce descriptive CAS data, while a high-precision encoder may produce reconstructable CAS data.

