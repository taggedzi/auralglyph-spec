# AuralGlyph

**A visual-native representation for audio and other scalar sampled signals.**

**Status:** Experimental v0.1 working draft, consolidated for review. The logical model is documented; portable payload encoding and numerical interoperability still require work.

## Start here

- [Current specification](spec/auralglyph-spec.md) — authoritative entry point and reading order.
- [Conformance checklist](spec/sections/08-conformance.md) — requirements across the model.
- [Review register](notes/spec-review.md) — proposed clarifications, remaining gaps, and completion criteria.
- [TODO](notes/todo.md) — current work list.

The versioned specification files are historical drafts. Their superseded text does not add requirements to the current specification.

## What AuralGlyph represents

AuralGlyph organizes time–frequency data for audio, bioacoustics, sonar, vibration, seismic, biological, RF-derived, and other scalar sampled measurements. It separates the stored signal representation from its visual projection and downstream interpretation.

| Layer | Responsibility |
| --- | --- |
| Canonical Audio Surface (CAS) | One scalar signal stream, organized by band, time frame, and bin |
| Session | Relationships between CAS instances, including channels, timing, and sensor positions |
| AGF container | Logical packaging of CAS instances, Sessions, profiles, provenance, and related data |

Stereo uses two CAS instances. Sensor arrays and multicomponent instruments use one CAS per scalar stream. Different CAS instances may use different sample rates and transforms.

The core coefficient structure is:

```math
C[b,t,f]
```

Each band may have a different number of bins, while all bands in one CAS share the same frame axis. Reconstructable transforms preserve the coefficient information and metadata required for inversion. Derived or descriptive data may preserve less information.

## Explicit capability levels

- **Descriptive:** Supports display or inspection with limited metadata; does not claim reconstruction.
- **Interpretable:** Describes frame timing and band meaning, including explicitly symbolic or unknown frequency mappings.
- **Reconstructable:** Supplies the coefficients, mappings, transform parameters, and dependencies needed for its declared inverse and fidelity.

Reconstruction is conditional on the declared profile and retained information. It cannot restore information discarded by an earlier lossy codec. The baseline STFT profile currently needs additional numerical conventions before independent implementations can reliably exchange coefficients.

## Design principles

- Each CAS represents one physical or logical scalar stream.
- Views do not modify source coefficients. Processing produces explicitly derived data.
- Band labels help readers; metadata defines their meaning. Audible, infrasonic, and ultrasonic bands are optional conventions.
- Unknown metadata remains unknown. Shared container membership does not establish synchronization or location.
- Observation time, sensor latency, integration windows, and propagation delay have distinct meanings.
- Source localization, rendering, classification, and Scene interpretation remain downstream activities.
- AGF v0.1 defines a logical container. It does not require a particular file archive, binary encoding, or network transport.

## Repository contents

```text
auralglyph-spec/
├─ spec/
│  ├─ auralglyph-spec.md                 # Current specification, core, glossary
│  ├─ sections/                         # Normative sections 4–8
│  └─ auralglyph-spec-v0.1*.md           # Historical drafts
├─ examples/                            # Informative usage and metadata fragments
├─ notes/                               # Review register, TODO, development journals
├─ CONTRIBUTING.md                      # DCO and Markdown/math conventions
├─ GOVERNANCE.md                        # Stewardship and review process
└─ LICENSE                              # CC BY 4.0
```

There is no reference encoder, decoder, visualizer, or executable validation schema in this repository yet.

## Examples

- [Stereo audio conversion](examples/audio-file-to-agf.md)
- [Source-preserving spatial audio](examples/source-preserving-spatial-audio.md)
- [Multi-station sensor network](examples/multi-station-sensor-network.md)

These examples explain intended use. They are not complete interchange files or validated reconstruction fixtures.

## Next milestone

Finish the logical coefficient payload contract, precise STFT conventions, metadata binding, and complete fixtures. Review pose/clock extension boundaries and validate the result before freezing a release. Detailed scope and acceptance criteria are in the [review register](notes/spec-review.md).

Physical serialization, transport, rendering, and Scene profiles are separate future work.

## Contributing and license

AuralGlyph uses [CC BY 4.0](LICENSE). Contributions follow [CONTRIBUTING.md](CONTRIBUTING.md), including commit sign-offs. Specification changes are reviewed under [GOVERNANCE.md](GOVERNANCE.md).

## AI-Assisted Development Notice

Portions of this project’s early documentation and conceptual development were created with the assistance of AI tools. These tools helped refine ideas, expand technical discussions, organize drafts, and accelerate the writing process.

The core concept, goals, and architectural direction of AuralGlyph originate with the project’s creator.
AI was used as a collaborative tool - similar to a technical editor - to explore alternatives, clarify reasoning, and improve structure and consistency.

All final decisions, interpretations, and design choices were made by the project founder.

AI involvement is disclosed for transparency, and to ensure contributors understand that while AI participated in the drafting process, the creative and conceptual ownership of AuralGlyph remains firmly human-directed.
