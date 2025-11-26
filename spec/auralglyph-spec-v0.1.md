### AuralGlyph Specification - Draft v0.1

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
* **Multi-view**, allowing different coordinate system projections (linear, spiral, cylindrical, Hilbert, etc.) without changing the stored data

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
  A 1D sequence of samples $ x[n] $ representing a signal over time at a given sample rate.

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

* **Projection / View**
  Any mapping from CAS into a 2D or 3D visualization (e.g., a rectangular spectrogram image, spiral view, cylindrical view, etc.). Projections do **not** change the stored CAS data.

* **Container**
  The on-disk representation (file format) encapsulating CAS data and associated metadata.

---

#### 3. Canonical Audio Surface (CAS): Formal Definition

This section is **normative** unless otherwise noted.

The Canonical Audio Surface (CAS) is the abstract, canonical representation of a signal in AuralGlyph. All AuralGlyph containers MUST encode exactly one CAS instance for each logical signal stream.

A CAS instance consists of:

1. A complex-valued, multi-band time–frequency tensor (or an equivalent per-band family of matrices).
2. A synchronized time axis shared by all bands.
3. A band configuration describing how each band maps indices to physical frequency.
4. A transform specification describing how the CAS relates to the time-domain signal(s).

---

##### 3.1 Index Sets and Tensor Structure

A CAS instance is defined over three discrete index sets:

- Band index set  
  \[
  \mathcal{B} = \{ 0, 1, \dots, B-1 \}
  \]
  where \(B \in \mathbb{N}_{>0}\) is the number of bands.

- Time-frame index set  
  \[
  \mathcal{T} = \{ 0, 1, \dots, T-1 \}
  \]
  where \(T \in \mathbb{N}_{>0}\) is the number of time frames.

- Per-band frequency index sets  
  For each band \(b \in \mathcal{B}\),
  \[
  \mathcal{F}_b = \{ 0, 1, \dots, F_b - 1 \}
  \]
  where \(F_b \in \mathbb{N}_{>0}\) is the number of frequency bins in band \(b\).

The CAS coefficients are complex values
\[
C_b[t, f] \in \mathbb{C}
\]
defined for all \(b \in \mathcal{B}\), \(t \in \mathcal{T}\), and \(f \in \mathcal{F}_b\).

Equivalently, one may view CAS as a ragged 3D tensor
\[
C[b, t, f]
\]
with a band-dependent frequency dimension. The representation details (e.g., packed tensor vs. per-band matrices) are container-specific and non-normative; the abstract model above is normative.

Each complex coefficient MAY be interpreted as:

- magnitude–phase: \(|C_b[t, f]|, \arg C_b[t, f]\), or
- real–imaginary: \(\Re C_b[t, f], \Im C_b[t, f]\).

> **Normative requirement:**  
> Every AuralGlyph container MUST unambiguously specify which representation (real/imag or mag/phase) is used for stored coefficients and how to convert them into \(\mathbb{C}\).

---

##### 3.2 Time Axis

All bands share a single, synchronized time axis. Let:

- \(f_s\) denote the sample rate in Hz of the underlying time-domain signal.
- \(H\) denote the hop length in samples between successive analysis frames.
- \(N\) denote the analysis window length in samples.

These parameters MUST be recorded in CAS metadata.

The analysis frame \(t \in \mathcal{T}\) corresponds to a time interval in the original signal. The nominal center time (in seconds) of frame \(t\) is:

\[
t_{\mathrm{sec}}(t) = \frac{t \cdot H}{f_s}.
\]

> **Normative requirements:**
>
> 1. The time-frame index set \(\mathcal{T}\) MUST be strictly ordered and contiguous starting at 0.  
> 2. All bands MUST use the same \(\mathcal{T}\), i.e., the same hop length and frame count \(T\).  
> 3. The CAS metadata MUST include at least:
>    - `sample_rate_hz` \(= f_s\)
>    - `analysis_window_length_samples` \(= N\)
>    - `hop_length_samples` \(= H\)

The exact choice of window function and its parameters (e.g., Hann, Blackman, custom) MUST also be recorded as part of the transform specification (see §4).

---

##### 3.3 Frequency Axis and Band Configuration

Each band \(b \in \mathcal{B}\) is defined by a **band configuration** that maps its discrete frequency indices to physical frequencies in Hz.

For band \(b\), define a mapping:

\[
\phi_b : \mathcal{F}_b \rightarrow \mathbb{R}_{\ge 0}
\]

such that \(\phi_b(f)\) yields the center frequency (in Hz) associated with bin \(f\).

The band configuration MUST specify, at minimum:

- `band_index` — integer \(b \in \mathcal{B}\).
- `band_name` — human-readable identifier (e.g., `"infra"`, `"human"`, `"ultra"`, `"custom-1"`).
- `f_min_hz` — minimum frequency covered by the band (inclusive or approximate).
- `f_max_hz` — maximum frequency covered by the band (inclusive or approximate).
- `num_bins` — \(F_b\).
- `bin_layout` — description of the mapping \(\phi_b\), such as:
  - `"linear-hz"` (approximately uniform spacing in Hz),
  - `"log10-hz"`, `"mel"`, `"bark"`,
  - or `"custom:<profile-id>"`.
- `reconstructable` — boolean indicating whether this band participates in time-domain reconstruction.

When `bin_layout` is `"custom:<profile-id>"`, the spec or profile MUST define:

- whether bins are defined by centers, edges, or a parametric mapping,
- how to compute \(\phi_b(f)\) for any integer \(f \in \mathcal{F}_b\).

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

---

##### 3.4 Transform Relationship

Conceptually, CAS coefficients are produced by applying an analysis transform \(\mathcal{T}\) to a time-domain signal \(x[n]\):

\[
C_b[t, f] = \mathcal{T}_b(x)[t, f]
\]

for each reconstructable band \(b\).

An inverse transform \(\mathcal{T}^{-1}\) produces an approximate or exact time-domain reconstruction \(\hat{x}[n]\) from the CAS coefficients of all reconstructable bands.

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
  - `signal_domain` — descriptive string (e.g., `"acoustic-pressure"`, `"acceleration"`, `"rf-downconverted"`).
  - `sample_rate_hz` — \(f_s\).
  - `num_frames` — \(T\).
  - `analysis_window_length_samples` — \(N\).
  - `hop_length_samples` — \(H\).
  - `transform_type` — see §4.
  - `transform_profile_id` — optional identifier of a standard transform profile (e.g., `"AGF-STFT-48k-v1"`).

- **Per-band fields (for each \(b \in \mathcal{B}\))**
  - `band_index`
  - `band_name`
  - `f_min_hz`
  - `f_max_hz`
  - `num_bins` (\(F_b\))
  - `bin_layout`
  - `reconstructable` (bool)
  - `bin_mapping_params` — optional structure containing parameters required to compute \(\phi_b\) (e.g., start frequency, spacing, log base).

- **Coefficient representation fields**
  - `coefficient_encoding` — one of:
    - `"complex-real-imag"`
    - `"complex-mag-phase"`
    - or a future extension.
  - `amplitude_scale` — description of linear or logarithmic scaling (e.g., `"linear"`, `"power"`, `"db-relative-1.0"`).

> **Normative requirement:**  
> A CAS instance is considered **well-formed** only if:
>
> - all global CAS fields are present and internally consistent,  
> - all bands have valid and complete configurations, and  
> - the transform specification and coefficient representation together are sufficient to interpret the numeric values as a complex-valued time–frequency representation.

---

##### 3.6 Invariance Under Projection (Views)

A CAS instance is the canonical representation of the signal in AuralGlyph. Any visualization or projection (“view”) — such as a rectangular spectrogram, spiral representation, cylindrical plot, Hilbert curve mapping, or custom artwork — MUST be defined as a mapping:

\[
\mathcal{P} : \{C_b[t, f]\} \rightarrow \text{image(s) or geometry}
\]

that does **not** modify the underlying CAS coefficients.

> **Normative requirement:**  
> Projections and views MUST treat CAS as read-only. Any processing that modifies the coefficients (e.g., filtering, masking, enhancement) MUST be described as a transform or processing step applied *before* visualization and MUST result in a new CAS instance or a derived data product, not an in-place mutation of the original CAS.

---

##### 3.7 Multi-Channel and Multi-Stream Signals (Forward-Looking, Non-Normative)

This draft focuses on single-stream CAS (e.g., mono or mixed-down representations). Future revisions of the spec MAY extend CAS to support:

- Multiple synchronized CAS streams (e.g., stereo, multichannel, or multi-sensor arrays).
- Explicit spatial metadata and per-channel band configurations.

Until such extensions are standardized, implementations that encode multiple channels SHOULD treat each channel as a separate CAS instance and define the channel relationships at the container or metadata level.

---

#### 4. Transform Requirements (High-Level)

> **Note:** This section is just sketched; we’ll flesh it out in later iterations.

AuralGlyph does not hard-code a single transform (like “must be STFT”), but it **does** require:

* A well-defined analysis transform $ \mathcal{T} $ mapping time-domain signals to CAS:
  $$
  C = \mathcal{T}(x)
  $$

* A corresponding inverse transform $ \mathcal{T}^{-1} $ (for applicable bands) that approximately or exactly reconstructs the time-domain signal:
  $$
  \hat{x} = \mathcal{T}^{-1}(C)
  $$

Implementations MUST declare:

* The transform type (e.g. `"stft"`, `"cqt"`, `"wavelet"`, `"custom"`).
* All parameters needed to reconstruct:

  * sample rate
  * window length and shape
  * hop length
  * FFT size (for STFT)
  * frequency/band mapping details

Transform choice affects:

* Frequency resolution vs. time resolution
* CAS tensor shape
* Reconstruction fidelity

Later spec versions can define **standard transform profiles**, e.g.:

* `AGF-STFT-48k-v1` - standardized STFT config for 48 kHz audio
* `AGF-STFT-192k-v1` - for extended ultrasonic capture
* `AGF-WAVELET-SONAR-v1` - for sonar / whale song work
* `AGF-WAVELET-SIEZEMIC-v1` - for siezemic data

---

#### Authorship and AI-Assistance Disclosure

Portions of this specification were drafted or refined using AI-assisted writing tools. These tools were employed to improve clarity, organization, and completeness in the presentation of ideas. The underlying concepts, design decisions, and direction of the AuralGlyph format originate with the project’s author, who reviewed and approved all final content.

This disclosure is provided to ensure transparency in the development of the specification.
