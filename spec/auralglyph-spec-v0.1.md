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

#### 3. Core Data Model: Canonical Audio Surface (CAS)

AuralGlyph’s central object is the **Canonical Audio Surface**, a 3D complex-valued tensor:

$$
C[b, t, f]
$$

Where:

* ( b ) - **band index**

  * Integer in ([0, B-1]), where ( B ) is the number of frequency bands.
  * Each band has a defined frequency range and resolution.

* ( t ) - **time frame index**

  * Integer in ([0, T-1]), where ( T ) is the number of time frames.
  * Each frame corresponds to a time interval (e.g., STFT hop).

* ( f ) - **frequency bin index**

  * Integer in ([0, F_b-1]), where ( F_b ) is the number of frequency bins in band ( b ).

Each element:

$$
C[b, t, f] \in \mathbb{C}
$$

is a **complex coefficient**, which may be interpreted as:

* **Magnitude + phase** ($|C|$, $\arg C)$, or
* **Real + imaginary parts** $Re(C), Im(C)$.

> **Normative requirement:**
> An AuralGlyph implementation MUST be able to interpret the CAS tensor sufficiently to reconstruct a time-domain signal for all bands that correspond to physical audio (where an inverse transform is defined).

##### 3.1 Time Axis

The time index $ t $ corresponds to discrete analysis frames, defined by:

* Sample rate $ f_s $ (Hz)
* Window length $ N $ (samples)
* Hop length $ H $ (samples)

The approximate center time of frame $ t $ is:

$$
t_{\text{sec}} = \frac{t \cdot H}{f_s}
$$

The values $ f_s $, $ N $, and $ H $ are part of the CAS metadata.

##### 3.2 Frequency Axis and Bands

The frequency index $ f $ within band $ b $ corresponds to a specific frequency (or range of frequencies). The mapping from $ (b, f) $ to physical frequency in Hz is defined by the transform and band configuration (see later sections).

A band definition includes:

* **Band name** (e.g. `"infra"`, `"human"`, `"ultra"`)
* **Frequency range** $ [f_{\text{min}}, f_{\text{max}}] $ in Hz (may be open-ended)
* **Bin layout** for that band:

  * Linear (e.g., equally spaced in Hz)
  * Logarithmic / mel / Bark
  * Custom mapping

> **Normative requirement:**
> For bands designated as *reconstructable audio bands*, the combination of band layout + transform MUST allow an invertible mapping back to time-domain samples within the defined frequency range.

##### 3.3 Multi-band CAS

Multiple bands are stacked along the band axis $ b $. Bands may overlap or be disjoint in frequency; the standard will define recommended profiles (e.g., a three-band configuration with infra/human/ultra).

All bands share the same time frame index $ t $; that is, they are synchronized in time.

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
