# Source-Preserving Spatial Audio

This example is non-normative.

AuralGlyph can support **source-preserving, renderer-independent spatial audio** workflows when CAS instances are captured or authored with sufficient timing, source, and spatial metadata.

Instead of storing only a final speaker mix, an AGF container may store the underlying scalar source streams as separate CAS instances, then use session metadata to describe how those streams relate in time and space.

## Concept

Traditional spatial or surround formats often target a specific playback model, speaker layout, or rendering ecosystem.

AuralGlyph can instead describe:

```text
captured/authored sources
+ timing metadata
+ spatial/session metadata
        ↓
open AGF container
        ↓
multiple possible renderers
```

Possible render targets include:

* stereo speakers
* headphones
* surround systems
* height-speaker layouts
* VR/AR environments
* custom speaker arrays
* scientific or analytical tools

## CAS / Session Split

This fits the AuralGlyph data model directly:

* **CAS** stores each scalar signal stream.
* **Session** stores how CAS instances relate in time, space, and recording context.
* **Scene or render profiles** may later describe how the aligned data should be presented, mixed, or experienced.

## Example: Concert Recording

A concert recording might contain:

```text
concert.agf
├─ CAS: lead vocal microphone
├─ CAS: guitar amplifier microphone
├─ CAS: bass direct input
├─ CAS: drum overhead left
├─ CAS: drum overhead right
├─ CAS: front room microphone
├─ CAS: rear room microphone
├─ CAS: audience microphone
└─ session metadata:
   ├─ time alignment
   ├─ stage positions
   ├─ room microphone positions
   ├─ source labels
   ├─ calibration notes
   └─ optional render hints
```

A renderer could then create different playback experiences from the same underlying source-preserving data.

## Interoperability Goal

AuralGlyph should not be framed as directly replacing any existing commercial format.

Better framing:

> AuralGlyph can define an open, source-preserving spatial audio container that allows multiple renderers, playback systems, and analysis tools to work from the same underlying observation data.

This approach may help reduce vendor lock-in and promote interoperability in spatial audio workflows.

## Important Limitations

This example does not define a required renderer, speaker layout, mixing algorithm, or perceptual model.

AuralGlyph stores observations and relationships. Rendering decisions are left to downstream tools, profiles, or future non-core specifications.
