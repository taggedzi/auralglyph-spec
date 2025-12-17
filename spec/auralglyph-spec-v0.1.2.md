### AuralGlyph Specification - Draft v0.1.1

**Status:** Experimental draft
**Authors:** taggedzi
**Name:** AuralGlyph (AGF)

---

#### 1. Introduction

AuralGlyph is a **unified time–frequency data format** designed to represent signals as a **canonical time–frequency tensor** that is:

* **Reversible** to time-domain waveforms (where applicable)
* **Directly visualizable** as images or volumetric plots
* **Domain-agnostic**, supporting:

  * audible audio
  * ultrasonic and infrasonic signals
  * sonar
  * seismic and vibration data
  * electromagnetics mapped into time–frequency space
* **Multi-band**, explicitly separating human-audible and non-audible frequency regions
* **Multi-view**, allowing different coordinate system projections (linear, spiral, cylindrical, Hilbert, etc.) without altering CAS coefficients

AuralGlyph is not:

* A psychoacoustic compression format like MP3 or AAC
* A replacement for simple PCM containers like WAV when only basic audio storage is needed
* Tied to any single visualization style

Instead, AuralGlyph defines a **Canonical Audio Surface (CAS)** that acts as a shared language between:

* Signal capture systems
* Visualization tools
* Analysis/ML pipelines
* Playback/rendering engines

---

#### 2. Terminology

This specification uses the following terms:

* **Signal**
  Any real-valued, time-varying phenomenon that can be sampled, such as sound pressure, acceleration, electric field strength, etc.

* **Time-domain signal**
  A 1D sequence of samples $`x[n]`$ representing a signal over time at a given sample rate.

* **Continuous-time vs. discrete-time**
  AuralGlyph operates on **discrete-time** sampled signals (e.g., 48 kHz PCM). References to continuous-time are conceptual.

* **Band**
  A named subrange of the frequency axis with shared semantics, e.g.:

  * *Infrasound band* (0–20 Hz)
  * *Human band* (20 Hz–20 kHz)
  * *Ultrasonic band* (20 kHz–100 kHz)

* **Canonical Audio Surface (CAS)**
  The core AuralGlyph data structure: a 3D tensor of complex coefficients indexed by **band**, **time**, and **frequency**.

* **Coefficient**
  A complex value representing the contribution of a certain frequency bin at a certain time frame (and band) to the overall signal.

* **Transform**
  The mapping from a time-domain signal to the CAS tensor (e.g., an STFT, CQT, or wavelet transform) and its inverse.

* **Pose Track**  
  A time-stamped sequence of sensor positions (and optionally orientations) referenced by a CAS instance when `sensor_pose_model = "time-varying-external"`.

* **Projection / View**
  Any mapping from CAS into a 2D or 3D visualization (e.g., a rectangular spectrogram image, spiral view, cylindrical view, etc.). Projections do **not** change the stored CAS data.

* **Container**
  The on-disk representation (file format) encapsulating CAS data and associated metadata.

---

#### 3. Canonical Audio Surface (CAS): Formal Definition

This section is **normative** unless otherwise noted.

The Canonical Audio Surface (CAS) is the abstract, canonical representation of a signal in AuralGlyph. All AuralGlyph containers MUST encode exactly one CAS instance for each logical signal stream.

> **Normative constraint (single-sensor CAS):**  
> A CAS instance represents exactly one scalar-valued signal from exactly one physical or logical sensor.  
> 
> - “Scalar-valued” means one primary measurement per sample (e.g., acoustic pressure, acceleration magnitude, voltage), not an interleaved or vector of channels.  
> - “One sensor” means a single microphone, geophone, hydrophone, antenna, radio receiver chain, accelerometer, etc.  
> 
> Multi-sensor and multi-channel configurations MUST be represented as collections of CAS instances, with relationships defined at the container or session level, not by extending CAS itself.

Note: This is a design constraint for v0.1. Future versions MAY introduce aggregate structures without changing the meaning of scalar single-sensor CAS instances.

A CAS instance consists of:

1. A complex-valued, multi-band time–frequency tensor (or an equivalent per-band family of matrices).
2. A synchronized time axis shared by all bands.
3. A band configuration describing how each band maps indices to physical frequency.
4. A transform specification describing how the CAS relates to the time-domain signal(s).

---

##### 3.1 Index Sets and Tensor Structure

A CAS instance is defined over three discrete index sets:

- Band index set  
```math
B = { 0, 1, ..., B-1 }
```
  where $`B ∈ N_{>0}`$ is the number of bands.

- Time-frame index set  
```math
T = { 0, 1, ..., T-1 }
```
  where $`T ∈ N_{>0}`$ is the number of time frames.

- Per-band frequency index sets  
  For each band $`b ∈ B`$,
```math
F_b = { 0, 1, ..., F_b - 1 }
```
  where $`F_b ∈ N_{>0}`$ is the number of frequency bins in band $`b`$.

The CAS coefficients are complex values
```math
C_b[t, f] ∈ C
```
defined for all $`b ∈ B`$, $`t ∈ T`$, and $`f ∈ F_b`$.

Equivalently, one may view CAS as a ragged 3D tensor
```math
C[b, t, f]
```
with a band-dependent frequency dimension. The representation details (e.g., packed tensor vs. per-band matrices) are container-specific and non-normative; the abstract model above is normative.

Each complex coefficient MAY be interpreted as:

- magnitude–phase: $`|C_b[t, f]|, \arg C_b[t, f]`$, or
- real–imaginary: $`\Re C_b[t, f], \Im C_b[t, f]`$.

> **Normative requirement:**  
> Every AuralGlyph container MUST unambiguously specify which representation (real/imag or mag/phase) is used for stored coefficients and how to convert them into $`C`$.

See §3.5 for required metadata fields corresponding to these index sets.

---

##### 3.2 Time Axis

All bands share a single, synchronized time axis. Let:

- $`f_s`$ denote the sample rate in Hz of the underlying time-domain signal.
- $`H`$ denote the hop length in samples between successive analysis frames.
- $`N`$ denote the analysis window length in samples.

These parameters MUST be recorded in CAS metadata.

The analysis frame $`t ∈ T`$ corresponds to a time interval in the original signal. The nominal center time (in seconds) of frame $`t`$ is:

```math
t_{sec}(t) = \frac{t \cdot H}{f_s}.
```

> **Normative requirements:**
>
> 1. The time-frame index set $`T`$ MUST be strictly ordered and contiguous starting at 0.  
> 2. All bands MUST use the same $`T`$, i.e., the same hop length and frame count $`T`$.  
> 3. The CAS metadata MUST include at least:
>    - `sample_rate_hz` $`= f_s`$
>    - `analysis_window_length_samples` $`= N`$
>    - `hop_length_samples` $`= H`$
>    - `num_frames` which MUST equal the cardinality of the time-frame index set $`|T| = T`$.

The exact choice of window function and its parameters (e.g., Hann, Blackman, custom) MUST also be recorded as part of the transform specification (see §4).

See §3.5 for fields that define the time axis in container metadata.

---

##### 3.3 Frequency Axis and Band Configuration

Each band $`b ∈ B`$ is defined by a **band configuration** that maps its discrete frequency indices to physical frequencies in Hz.

For band $`b`$, define a mapping:

```math
φ_b : F_b → R_{\ge 0}
```

such that $`φ_b(f)`$ yields the center frequency (in Hz) associated with bin $`f`$.

The band configuration MUST specify, at minimum:

- `band_index` — integer $`b ∈ B`$.
- `band_name` — human-readable identifier (e.g., `"infra"`, `"human"`, `"ultra"`, `"custom-1"`).
- `f_min_hz` — minimum frequency covered by the band (inclusive or approximate).
- `f_max_hz` — maximum frequency covered by the band (inclusive or approximate).
- `num_bins` — $`F_b`$.
- `bin_layout` — description of the mapping $`φ_b`$, such as:
  - `"linear-hz"` (approximately uniform spacing in Hz),
  - `"log10-hz"`, `"mel"`, `"bark"`,
  - or `"custom:<profile-id>"`.
- `reconstructable` — boolean indicating whether this band participates in time-domain reconstruction.

When `bin_layout` is `"custom:<profile-id>"`, the spec or profile MUST define:

- whether bins are defined by centers, edges, or a parametric mapping,
- how to compute $`φ_b(f)`$ for any integer $`f ∈ F_b`$.

> **Normative requirements:**
>
> 1. For any band marked `reconstructable: true`, the combination of:
>    - transform definition, and  
>    - band configuration
>
>    MUST be sufficient to define an invertible or numerically stable mapping back to a time-domain signal over that band’s frequency range, within the limitations of the chosen transform.
>
> 2. Bands MAY overlap in frequency or be disjoint. Overlap behavior (e.g., summation, windowing, weighting across bands) MUST be defined by the transform profile or implementation, not by CAS itself.
>
> 3. Bands that are not reconstructable (e.g., derived, analysis-only, or compressed summary bands) MUST be explicitly marked as such.

Band configuration metadata fields are defined in §3.5 under “Per-band fields.”

---

##### 3.4 Transform Relationship

Conceptually, CAS coefficients are produced by applying an analysis transform $`T`$ to a time-domain signal $`x[n]`$:

```math
C_b[t, f] = T_b(x)[t, f]
```

for each reconstructable band $`b`$.

An inverse transform $`T^{-1}`$ produces an approximate or exact time-domain reconstruction $`\hat{x}[n]`$ from the CAS coefficients of all reconstructable bands.

> **Normative requirements:**
>
> 1. Every CAS instance MUST declare a **transform specification**, identifying:
>    - `transform_type` (e.g., `"stft"`, `"cqt"`, `"wavelet"`, `"custom"`), and  
>    - all parameters required for reconstruction (see §4).
> 2. For transform types that are defined as “invertible” in this specification or in a referenced profile, an implementation that follows the profile MUST be able to reconstruct a time-domain signal from the CAS for all reconstructable bands, up to numerical precision and transform design limits.

---

##### 3.5 Metadata Summary (Non-Container-Specific)

A CAS instance MUST be accompanied by a metadata structure with at least the following fields (names are illustrative; container bindings MAY use different field names but MUST preserve the semantics):

- **Global CAS fields**
  - `cas_version` — semantic version of the CAS spec (e.g., `"0.1.0"`).
  - `signal_domain` — descriptive string (e.g., `"acoustic-pressure"`, `"acceleration"`, `"rf-downconverted"`, `"seismic-displacement"`).
  - `sample_rate_hz` — $`f_s`$.
  - `num_frames` — $`T`$.
  - `analysis_window_length_samples` — $`N`$.
  - `hop_length_samples` — $`H`$.
  - `transform_type` — see §4.
  - `transform_profile_id` — optional identifier of a standard transform profile (e.g., `"AGF-STFT-48k-v1"`).

- **Sensor identity and location (single-sensor CAS)**
  - `sensor_id` — opaque identifier for the sensor (string, globally unique within a recording session).
  - `sensor_type` — human-readable type (e.g., `"microphone"`, `"hydrophone"`, `"geophone"`, `"radio-frontend"`, `"seismic-array-element"`).
  - `reference_frame` — string describing the coordinate frame (e.g., `"ENU@lat=<...>,lon=<...>,alt=<...>"`, `"ECEF"`, `"local-lab-frame"`, `"custom:<frame-id>"`).

  - `sensor_pose_model` — describes how sensor pose is defined:
    - `"static"` — pose is constant for all frames in this CAS.
    - `"time-varying-external"` — pose varies over time and is defined by an external pose track.
    - `"unknown"` — pose information is not known or not provided.

  - `sensor_position` — when `sensor_pose_model = "static"`, a 3-element numeric vector `[x, y, z]` giving the sensor position in the declared `reference_frame`. MAY be omitted or approximate when `sensor_pose_model = "unknown"`.

  - `sensor_orientation` — optional orientation of the sensor (e.g., as Euler angles, quaternion, or a profile-defined representation). When `sensor_pose_model = "static"`, this applies to all frames.

  - `sensor_pose_track_id` — when `sensor_pose_model = "time-varying-external"`, an identifier that refers to a time-stamped pose trajectory defined at the container or session level. The pose track defines position (and optionally orientation) as a function of the same time reference used by this CAS.

> **Normative requirements (sensor pose):**
>
> 1. A CAS intended for spatial or multi-sensor analysis MUST declare `reference_frame` and `sensor_pose_model`.
> 2. If `sensor_pose_model = "static"`, `sensor_position` MUST be present and interpreted as constant over all frames.
> 3. If `sensor_pose_model = "time-varying-external"`, `sensor_pose_track_id` MUST be present, and the referenced pose track MUST be defined in the same container or recording session using the same `time_reference_type` and `time_reference_value`.
> 4. If `sensor_pose_model = "unknown"`, consumers MUST NOT assume any particular spatial relationship to other sensors, except what can be inferred from higher-level metadata or user configuration.

- **Time reference and synchronization**
  - `time_reference_type` — one of:
    - `"absolute-utc"` — times are referenced to UTC,
    - `"relative-session"` — times are relative to a session start,
    - `"relative-arbitrary"` — times are relative to an unspecified origin; only differences are meaningful.
  - `time_reference_value` — optional origin value (e.g., `"2025-11-25T20:15:00Z"` for UTC or an implementation-defined session ID).
  - `frame_zero_time_offset_sec` — the offset (in seconds) of frame `t = 0` relative to the time reference.
  - `time_sync_quality` — qualitative or numeric measure of synchronization quality (e.g., `"gps-disciplined"`, `"ntp-synced"`, `"free-running"`, or a profile-defined structure).

- **Per-band fields (for each $`b ∈ B`$)**
  - `band_index`
  - `band_name`
  - `f_min_hz`
  - `f_max_hz`
  - `num_bins` ($`F_b`$)
  - `bin_layout`
  - `reconstructable` (bool)
  - `bin_mapping_params` — optional structure containing parameters required to compute $`φ_b`$ (e.g., start frequency, spacing, log base).

- **Coefficient representation fields**
  - `coefficient_encoding` — one of:
    - `"complex-real-imag"`
    - `"complex-mag-phase"`
    - or a future extension.
  - `amplitude_scale` — description of linear or logarithmic scaling (e.g., `"linear"`, `"power"`, `"db-relative-1.0"`).

> **Normative requirement (units):**  
> All physical quantities MUST use SI units unless otherwise declared:
> - frequencies in Hertz (Hz),
> - times in seconds (s),
> - positions in meters (m).
>
> Container profiles MAY define scaled encodings but MUST specify conversion back to SI.

---

##### 3.6 Invariance Under Projection (Views)

A CAS instance is the canonical representation of the signal in AuralGlyph. Any visualization or projection (“view”) — such as a rectangular spectrogram, spiral representation, cylindrical plot, Hilbert curve mapping, or custom artwork — MUST be defined as a mapping:

```math
P : \{C_b[t, f]\} → \text{image(s) or geometry}
```

that does **not** modify the underlying CAS coefficients.

> **Normative requirement:**  
> Projections and views MUST treat CAS as read-only. Any processing that modifies the coefficients (e.g., filtering, masking, enhancement) MUST be described as a transform or processing step applied *before* visualization and MUST result in a new CAS instance or a derived data product, not an in-place mutation of the original CAS.

---

##### 3.7 Sensor Arrays and Spatial Reconstruction (Non-Normative)

Although CAS itself is defined strictly for single-sensor scalar signals, AuralGlyph is designed to support multi-sensor scenarios such as:

- multi-microphone recordings for immersive audio or beamforming,
- hydrophone arrays for whale tracking,
- geophone networks for seismic event localization,
- distributed RF receivers for direction finding or interferometry,
- astronomical sensors with known baselines.

For sensors with time-varying positions or orientations, CAS instances SHOULD declare `sensor_pose_model = "time-varying-external"` and reference an external pose track as defined in §3.5.

In such cases, a system SHOULD:

1. Encode **one CAS instance per sensor**, each with:
   - its own `sensor_id`, `sensor_type`, and either a static `sensor_position` or a `sensor_pose_track_id`, depending on its `sensor_pose_model`, and
   - a time reference (`time_reference_type`, `time_reference_value`, `frame_zero_time_offset_sec`) suitable for cross-sensor alignment.

2. Use a **session-level or container-level manifest** (outside the CAS core) to:
   - group CAS instances that belong to the same experiment or recording session,
   - declare the coordinate frame (`reference_frame`) shared by all sensors, and
   - specify any additional information required for spatial processing (e.g., medium properties, speed of propagation, known calibration offsets).

3. Perform spatial reconstruction or multi-sensor processing (e.g., direction-of-arrival estimation, triangulation, synthetic stereo or surround rendering) as a **derived operation** on the set of CAS instances, producing:
   - either new CAS instances (e.g., beamformed virtual channels), or
   - time-domain signals for playback or further analysis.

> **Non-normative note:**  
> AuralGlyph explicitly avoids baking “stereo” or “multi-channel” constructs into the CAS data model. Instead, it provides a principled way to combine independent, well-metadata’d single-sensor CAS instances. Stereo, surround, ambisonics, or arbitrary spatial representations can then be constructed as higher-level products when needed, using sensor positions and precise time alignment.

> **Non-normative note (moving sensors):**  
> Many practical systems involve sensors that move during recording, such as hydrophones on boats, microphones mounted on a performer, drones, or astronomical baselines defined by orbital motion. In such cases:
> - CAS stores single-sensor signals and their time bases.
> - Time-varying sensor pose SHOULD be represented by an external pose track identified by `sensor_pose_track_id`.
> - Spatial reconstruction (e.g., synthetic stereo, beamforming, source localization) SHOULD be performed by combining CAS signals with their corresponding pose tracks in the chosen `reference_frame`.

---

#### 4. Transform Requirements (High-Level)

> **Note:** This section is just sketched; we’ll flesh it out in later iterations.

AuralGlyph does not hard-code a single transform (like “must be STFT”), but it **does** require:

* A well-defined analysis transform $`T`$ mapping time-domain signals to CAS:
```math
C = T(x)
```

* A corresponding inverse transform $`T^{-1}`$ (for applicable bands) that approximately or exactly reconstructs the time-domain signal:
```math
\hat{x} = T^{-1}(C)
```

Implementations MUST declare:

* The transform type (e.g. `"stft"`, `"cqt"`, `"wavelet"`, `"custom"`).
* All parameters needed to reconstruct:

  * sample rate
  * window length and shape
  * hop length
  * FFT size (for STFT)
  * frequency and band-mapping details

Transform choice affects:

* Frequency resolution vs. time resolution
* CAS tensor shape
* Reconstruction fidelity

Later spec versions can define **standard transform profiles**, e.g.:

* `AGF-STFT-48k-v1` - standardized STFT config for 48 kHz audio
* `AGF-STFT-192k-v1` - for extended ultrasonic capture
* `AGF-WAVELET-SONAR-v1` - for sonar / whale song work
* `AGF-WAVELET-SEISMIC-v1` - for seismic data

---

#### 5. Cross-CAS Sessions

##### 5.1 Session (Normative)

A **Session** is a metadata object that relates multiple Canonical Audio Surface (CAS) assets by defining how their observations align in shared reference domains such as **time** and **space**.

A Session answers the question:

> *“Which observations are meaningfully comparable, and how are they aligned?”*

Sessions are a first-class concept in AuralGlyph and are required to support multi-sensor, multi-modal, or distributed observation scenarios.

---

##### 5.1.1 Purpose and Scope

A Session exists to:

* group independent CAS assets that participated in a shared observational context
* define alignment relationships between those assets
* enable synchronized comparison, analysis, and replay
* preserve the independence and integrity of each CAS asset

A Session does **not**:

* merge CAS data into a single signal
* define semantic meaning or causality
* classify or label observed phenomena
* assert that any particular interpretation is correct

CAS assets remain valid, interpretable, and reusable outside of any Session.

---

##### 5.1.2 Relationships Defined by a Session

A Session MAY define one or more of the following:

###### Temporal Relationships

* alignment offsets between CAS timelines
* clock drift models (e.g., linear, segmented)
* synchronization confidence or uncertainty
* reference time domain (e.g., UTC, GPS, monotonic)

###### Spatial Relationships

* sensor positions and orientations
* shared or named coordinate frames
* uncertainty bounds on position and orientation
* static or time-varying placement

###### Membership

* an explicit list of CAS assets participating in the Session

Sessions MAY relate heterogeneous CAS assets, including assets that:

* operate at different sample rates or bandwidths
* observe different physical media
* use different transforms or frequency layouts
* represent non-audio time-series phenomena

---

##### 5.1.3 Sensor Independence

Each CAS asset within a Session retains its own:

* transform definition
* frequency band structure
* calibration and response characteristics
* noise profile
* clock behavior

A Session defines **how observations relate**, not how they should be normalized, interpreted, or explained.

---

##### 5.1.4 Non-Goals

Sessions explicitly do **not**:

* define events
* infer causality
* impose analytical models
* prescribe visualization or user experience

Interpretation of aligned data is intentionally left to downstream systems.

> **Note:** AuralGlyph is designed with interpretive frameworks in mind, but such frameworks are out of scope for this specification. See Appendix A for discussion.

---

### Appendix A (Informative / Non-Normative)

#### Appendix A: Scenes and Interpretive Frameworks (Informative)

##### A.1 Motivation

While CAS assets capture **observations** and Sessions define **alignment**, meaningful understanding of complex, multi-sensor data requires **interpretation**.

Different disciplines, applications, and analytical goals may interpret the same aligned data in fundamentally different ways. Attempting to standardize interpretation would both limit innovation and risk imposing inappropriate assumptions.

For this reason, AuralGlyph intentionally separates **representation** from **interpretation**.

---

##### A.2 Scene (Conceptual)

A **Scene** is a reproducible description of how one or more Sessions are interpreted, examined, or experienced by an observer or analysis system.

A Scene answers the question:

> *“How should this aligned data be examined to highlight emergent or apparent phenomena?”*

Scenes are **not** part of the AuralGlyph core specification and are **not standardized**.

---

##### A.3 Characteristics of a Scene

A Scene is typically:

* **Deterministic**
  Given the same inputs, it produces the same interpretive experience.

* **Reproducible**
  Another observer can recreate how the data was examined or presented.

* **Selective**
  It may focus on specific sensors, time ranges, frequency regions, or spatial frames.

* **Intentional**
  It encodes *how to look*, not *what to conclude*.

Scenes may be authored by humans, generated by software, or produced by analytical or machine learning systems.

---

##### A.4 What Scenes May Include

A Scene MAY describe:

* references to one or more Sessions
* subsets of CAS assets
* time and frequency selections
* spatial framing or projections
* derived overlays or features
* annotations or guidance for observers

Scenes MAY reference emergent **Events**, but Events themselves are not defined by this specification.

---

##### A.5 What Scenes Must Not Assert

Scenes must not be interpreted as asserting:

* causality
* classification
* explanation
* ground truth

Scenes do not define *what happened*.
They define *how the data was examined*.

---

##### A.6 Design Intent

AuralGlyph is intentionally designed to support Scenes without prescribing them.

By standardizing **observations (CAS)** and **alignment (Sessions)** while leaving **interpretation (Scenes)** open, the specification:

* preserves scientific neutrality
* enables cross-disciplinary use
* encourages reproducible interpretation
* allows new analytical paradigms to emerge

Implementers are encouraged to design Scene representations that are transparent, reproducible, and appropriate to their domain.

---

#### Authorship and AI-Assistance Disclosure

Portions of this specification were drafted or refined using AI-assisted writing tools. These tools were employed to improve clarity, organization, and completeness in the presentation of ideas. The underlying concepts, design decisions, and direction of the AuralGlyph format originate with the project’s author, who reviewed and approved all final content.

This disclosure is provided to ensure transparency in the development of the specification.
