# AuralGlyph

**A universal, visual-native signal format that stores audio and other waveforms as a reversible multi-band time–frequency surface.**  
*Status: Draft / Experimental*

---

## What is AuralGlyph?

AuralGlyph is a new kind of signal representation that unifies **audio**, **bioacoustics**, **ultrasonic/infrasonic data**, **sonar**, **vibrations**, and even **electromagnetic time-series** into a single, coherent format.

Unlike traditional audio files, AuralGlyph represents signals as a **Canonical Audio Surface (CAS)** - a 3D time–frequency tensor that is:

- **Reversible** → can reconstruct the original signal  
- **Visual-native** → can be displayed as images or 3D surfaces  
- **Multi-band** → supports audible, infrasonic, and ultrasonic content  
- **Coordinate-agnostic** → can be rendered in linear, spiral, Hilbert, cylindrical, or custom views  
- **Cross-domain** → not restricted to human hearing or audio playback systems  

AuralGlyph bridges the gap between:

- audio engineering  
- bioacoustics  
- scientific analysis  
- signal processing  
- machine learning  
- digital art  

---

## Design principle (v0.1)

In this version of the specification, a CAS instance is intended to represent one scalar-valued signal from one physical or logical sensor. Multi-sensor or multi-component setups SHOULD be modeled as collections of CAS instances, with their relationships defined at the container or session level.

Future revisions MAY introduce aggregate types or profiles that allow vector-valued signals, without changing the semantics of existing scalar CAS instances.

---

## Why AuralGlyph?

Existing audio formats store signals as waveforms (time-domain PCM) or psychoacoustic encodings (MP3, AAC).  
Scientific formats store raw samples but not **time–frequency structures**.  
Image formats visualize spectrograms but cannot be reversed back to audio.

**AuralGlyph is the first format that is *natively* time–frequency based.**

It provides:

- A unified representation for *any* sampled physical phenomenon  
- Direct visualization without decoding  
- Optional extended bands for non-human frequencies  
- A consistent structure for ML models (spectrograms are already the lingua franca of audio ML)  
- A platform for new artistic and scientific visualizations  

---

## 🧠 Core Concept: Canonical Audio Surface (CAS)

At the heart of AuralGlyph is the **CAS**, a 3D tensor:
```math
C[b][t][f]
```

Where:

- **b** - band index (infra / human / ultra / custom)  
- **t** - time frame index  
- **f** - frequency bin index  
- Each value is a **complex coefficient** (magnitude + phase)

This single data structure is the *official* representation of the signal.  
Any visualization or projection (spectrogram, spiral, Hilbert, etc.) is derived from CAS without altering it.

---

## 🔍 Features

### ✔ Multi-Band Frequency Layout

- Infra (<20 Hz)  
- Human audible (20 Hz – 20 kHz)  
- Ultrasonic (20 kHz – 100+ kHz)  
- Arbitrary custom bands  
- Easily extended for sonar, RF, seismic, EEG/MEG, etc.

### ✔ Reversible Time–Frequency Encoding

- CAS contains enough information (given parameters) to reconstruct time-domain signals  
- Inversion requirements defined explicitly in the spec  
- Works with STFT, CQT, wavelet transforms, or future transforms

### ✔ Visual-Native Format

AuralGlyph is designed to look like:

- A spectrogram  
- A spiral cochleagram  
- A “piano roll wheel”  
- A Hilbert image  
- A cylindrical sonar map  

All without losing reversibility.

### ✔ Infinite Duration Support

Long signals are stored with:

- time-indexed frames  
- optional tiling  
- optional multi-resolution overviews  

The **storage** remains a clean tensor; the **display** decides how to render it.

### ✔ Cross-Domain Compatibility

AuralGlyph is not limited to audio.  
It can represent:

- Whale songs  
- Bat chirps  
- Seismic tremors  
- Biomedical signals  
- Machinery vibrations  
- Radio-frequency time-series (after downconversion)  
- Any sampled phenomenon with a frequency structure  

---

## 🚧 Project Status

AuralGlyph is currently in the **design and specification** stage.  
This repository contains:

- The evolving **AuralGlyph Specification (Draft v0.1)**
- Design notes and discussions  
- Planned transform profiles  
- Placeholder sections for future reference implementations  

Nothing here is final yet - expect rapid iteration and breaking changes.

---

## 📁 Repository Structure (planned)

```text
auralglyph-spec/
├─ README.md # Project overview (this file)
├─ spec/
│ ├─ auralglyph-spec-v0.1.md # Main draft specification
├─ notes/
│ ├─ design-journal.md # Rough ideas, explorations, sketches
│ └─ concepts.md # Coordinate systems, transforms, projections
├─ examples/
│ ├─ example-cas-diagrams/
│ └─ example-data/
└─ proto/ # (Future) reference encoder/decoder experiments
```


---

## 🗺 Roadmap

### **Phase 1 - Core Specification**

- [x] Canonical Audio Surface formal definition  
- [ ] Band profiles (infra/human/ultra)  
- [ ] Transform requirements (STFT baseline)  
- [ ] Metadata schema  
- [ ] Container structure  

### **Phase 2 - Visualization Guidelines**

- [ ] Linear spectrogram view  
- [ ] Spiral & cylindrical projections  
- [ ] Hilbert curve view  
- [ ] Multi-scale / overview modes  

### **Phase 3 - Reference Implementations**

- [ ] Python encoder (CAS ← audio)  
- [ ] Python decoder (audio ← CAS)  
- [ ] CAS visualizer (various projection modes)  

### **Phase 4 - Real-World Applications**

- [ ] Music encoding  
- [ ] Bioacoustic datasets  
- [ ] Sonar logs  
- [ ] Machine learning integration  
- [ ] Scientific datasets
- [ ] Seismic data

---

## 🤝 Contributing

This is an open experimental project.  
Ideas, questions, and feedback are welcome.  
If you want to participate in spec design, feel free to open an issue or discussion.

---

## 📜 License

[CC-BY-4.0](LICENSE)

---

## 🌟 Vision

AuralGlyph reimagines what a “signal file format” can be:

- not just for **listening**,  
- not just for **visualization**,  
- not just for **analysis**,  
but a single, unified medium that encodes the *full shape* of sound and other time–frequency signals -  
in a form that is both **human-visible** and **machine-reversible**.

---

## AI-Assisted Development Notice

Portions of this project’s early documentation and conceptual development were created with the assistance of AI tools. These tools helped refine ideas, expand technical discussions, organize drafts, and accelerate the writing process.

The core concept, goals, and architectural direction of AuralGlyph originate with the project’s creator.
AI was used as a collaborative tool - similar to a technical editor - to explore alternatives, clarify reasoning, and improve structure and consistency.

All final decisions, interpretations, and design choices were made by the project founder.

AI involvement is disclosed for transparency, and to ensure contributors understand that while AI participated in the drafting process, the creative and conceptual ownership of AuralGlyph remains firmly human-directed.
