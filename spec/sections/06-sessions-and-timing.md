#### 6. Sessions and Cross-CAS Timing

---

Summary: This document outlines the structure and rules for defining sessions in the AuralGlyph format, which involves relating multiple CAS (Common Acquisition Stream) instances together. Key points include:

- **Session Identity**: Each session must have a unique session_id, type (session_type), and list of member CAS instances (members).
- **Standard Session Types**: Defines types like "single-recording," "stereo-audio," etc., to categorize the relationship between CAS instances.
- **Timing Relationships**: Describes how member CAS timelines relate, including synchronization, offsets, and uncertainties.
- **Sensor Timing Metadata**: Provides details about how each CAS's time coordinate relates to its physical sensor, acquisition system, and recorded timestamps.
- **Spatial Metadata**: Includes information on station, sensor, device, or source positions for geospatial context.
- **Derived Interpretations**: Allows referencing derived analysis products that interpret the raw observations.
- **Conformance Rules**: Ensures that sessions are correctly formatted with sufficient metadata to support the relationships described.

In essence, it's a specification for organizing and relating multiple CAS instances into meaningful sessions, complete with timing, spatial, and interpretation information.

---

This section is **normative** unless otherwise noted.

A Session defines relationships among multiple CAS instances.

A CAS represents one scalar signal stream. A Session describes how two or more CAS instances are related in time, space, source context, observation context, or intended use.

Sessions are required when an AGF container represents multi-channel, multi-sensor, multi-station, multi-modal, or distributed observations.

---

##### 6.1 Session Purpose

A Session answers questions such as:

* Which CAS instances belong together?
* Were they captured during the same observation period?
* Are they synchronized?
* Do they share a common clock?
* Do they come from the same station, device, array, experiment, event, recording, or scene?
* How should consumers interpret timing relationships between streams?
* What spatial or sensor relationships are known?

A Session does not define conclusions, classifications, or interpretations.

A Session describes relationships among observations.

---

##### 6.2 Session Identity

Each Session MUST declare:

```yaml
session_id: <string>
session_type: <string>
members: <array>
```

Where:

* `session_id` is a stable identifier unique within the AGF container.
* `session_type` describes the general kind of relationship.
* `members` lists the CAS instances included in the Session.

Example:

```yaml
session:
  session_id: "stereo-song-session"
  session_type: "stereo-audio"
  members:
    - cas_id: "cas-left"
      role: "left"
    - cas_id: "cas-right"
      role: "right"
```

A CAS instance MAY belong to more than one Session.

For example, a microphone CAS may belong to both a station-level session and a larger field-observation session.

---

##### 6.3 Standard Session Types

AuralGlyph v0.1 defines the following standard session types:

```yaml
session_type: "single-recording"
```

A group of CAS instances derived from one recording or capture.

```yaml
session_type: "stereo-audio"
```

A two-channel audio relationship.

```yaml
session_type: "multi-channel-audio"
```

A conventional multi-channel audio relationship.

```yaml
session_type: "sensor-array"
```

Multiple sensors arranged as a known array.

```yaml
session_type: "multi-station-sensor-network"
```

Multiple observation stations contributing related CAS instances.

```yaml
session_type: "experiment"
```

A scientific, engineering, or controlled observation session.

```yaml
session_type: "derived-analysis"
```

A session grouping original CAS instances with derived products.

```yaml
session_type: "custom"
```

A producer-defined session type.

Custom session types SHOULD include enough metadata for consumers to understand the relationship being expressed.

---

##### 6.4 Session Membership

Each Session member MUST identify a CAS instance.

```yaml
members:
  - cas_id: <string>
```

Session members SHOULD declare their role when known.

```yaml
members:
  - cas_id: "cas-left"
    role: "left"

  - cas_id: "cas-right"
    role: "right"
```

For sensor sessions, members SHOULD identify the station, sensor, or device relationship when known.

```yaml
members:
  - cas_id: "station-a-audio-mic"
    station_id: "observation-station-a"
    sensor_role: "audible-sound"

  - cas_id: "station-a-seismic"
    station_id: "observation-station-a"
    sensor_role: "ground-vibration"
```

A consumer MUST NOT assume that two CAS instances are synchronized, co-located, or causally related merely because they appear in the same AGF container.

Those relationships MUST be declared by Session metadata.

---

##### 6.5 Time Reference

A Session SHOULD declare a shared time reference when timing relationships matter.

Example:

```yaml
time_reference:
  time_standard: "UTC"
  clock_source: "gnss"
  synchronization_method: "gnss"
  timing_accuracy_sec: 0.000001
```

Common fields include:

```yaml
time_reference:
  time_standard: <string>
  clock_source: <string>
  synchronization_method: <string>
  timing_accuracy_sec: <number or null>
```

The `time_standard` field SHOULD describe the time basis, such as `"UTC"`, `"TAI"`, `"GPS"`, `"local-clock"`, `"sample-index"`, or `"unknown"`.

The `clock_source` field SHOULD describe the source of timing, such as `"gnss"`, `"ptp"`, `"ntp"`, `"device-clock"`, `"manual"`, `"estimated"`, or `"unknown"`.

The `synchronization_method` field SHOULD describe how timing alignment was established.

If no reliable shared time reference exists, the Session MUST NOT claim precise synchronization.

---

##### 6.6 CAS Observation Time

Unless explicitly stated otherwise, CAS time coordinates describe the producer-declared observation timeline for the receiving sensor.

A CAS timestamp represents when a signal was measured, sampled, recorded, or corrected according to the declared time basis.

A CAS timestamp MUST NOT be interpreted as the originating source-event time unless additional metadata explicitly supports that interpretation.

The default interpretation is:

```text
CAS time = observation timeline at the receiving sensor
```

Not:

```text
CAS time = time the source event occurred
```

For example, a sound, seismic vibration, radio-frequency signal, thermal signal, or environmental measurement may be related to the same external event while arriving at different sensors at different times.

Source-time estimation is an interpretive operation.

---

##### 6.7 Sensor Timing Metadata

Each CAS MAY declare sensor timing metadata.

This metadata describes how the CAS time axis relates to the physical sensor, acquisition system, and recorded timestamps.

Example:

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
  sensor_latency_sec: 0.0025
  latency_correction_applied: false
  latency_uncertainty_sec: 0.0002
```

The `timestamp_basis` field SHOULD use one of the following values unless a custom profile defines otherwise:

```yaml
timestamp_basis:
  - "recorded-sample-time"
  - "sensor-reported-time"
  - "latency-corrected-observation-time"
  - "estimated-arrival-time"
  - "sample-index"
  - "unknown"
```

Where:

* `"recorded-sample-time"` means the time coordinate reflects when the sample was recorded by the acquisition system.
* `"sensor-reported-time"` means the sensor device supplied the timestamp.
* `"latency-corrected-observation-time"` means known sensor or acquisition latency has been corrected.
* `"estimated-arrival-time"` means the timestamp estimates when the signal arrived at the sensor.
* `"sample-index"` means timing is expressed relative to sample number rather than an external clock.
* `"unknown"` means the producer does not know the timestamp basis.

If the producer knows sensor or acquisition latency, it SHOULD declare:

```yaml
sensor_latency_sec: <number>
latency_correction_applied: <boolean>
latency_uncertainty_sec: <number or null>
```

If latency is unknown, the producer SHOULD use `null` rather than inventing a value.

---

##### 6.8 Integration Windows

Some sensors do not produce instantaneous measurements.

Examples include:

* radiation counters,
* air-quality sensors,
* thermal sensors,
* chemical sensors,
* environmental sensors,
* averaged power measurements,
* rolling spectral measurements,
* and other integrating instruments.

Such sensors SHOULD declare the integration or averaging window when known.

Example:

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
  integration_window_sec: 1.0
  sample_time_reference: "window-center"
```

The `sample_time_reference` field SHOULD use one of:

```yaml
sample_time_reference:
  - "instantaneous"
  - "window-start"
  - "window-center"
  - "window-end"
  - "unknown"
```

A consumer MUST NOT assume that a timestamp represents an instantaneous value unless the metadata declares that or the sensor profile defines it.

---

##### 6.9 Session-Level Timing Relationships

A Session SHOULD declare how member CAS timelines relate when timing matters.

Example:

```yaml
members:
  - cas_id: "station-a-audio-mic"
    time_relationship: "synchronized"
    timing_offset_sec: 0.0000
    timing_uncertainty_sec: 0.0001

  - cas_id: "station-a-seismic"
    time_relationship: "synchronized"
    timing_offset_sec: 0.0125
    timing_uncertainty_sec: 0.0010

  - cas_id: "station-a-radiation-count"
    time_relationship: "sampled-independent"
    timing_offset_sec: null
    timing_uncertainty_sec: 0.5
```

The `time_relationship` field SHOULD use one of the following values unless a custom profile defines otherwise:

```yaml
time_relationship:
  - "synchronized"
  - "offset-known"
  - "offset-estimated"
  - "sample-synchronous"
  - "sampled-independent"
  - "unsynchronized"
  - "unknown"
```

Where:

* `"synchronized"` means the streams share a common time reference within declared uncertainty.
* `"offset-known"` means a fixed timing offset is known.
* `"offset-estimated"` means an offset has been estimated but may be uncertain.
* `"sample-synchronous"` means streams share sample timing, such as left and right channels from one decoded audio file.
* `"sampled-independent"` means streams are part of the same session but sampled independently.
* `"unsynchronized"` means no synchronization is claimed.
* `"unknown"` means the timing relationship is not known.

If a timing offset is known or estimated, it SHOULD be declared as:

```yaml
timing_offset_sec: <number or null>
timing_uncertainty_sec: <number or null>
```

The sign convention for `timing_offset_sec` MUST be declared by the Session or profile if nonzero offsets are used.

For v0.1, the default convention is:

```text
corrected_time = cas_time + timing_offset_sec
```

relative to the Session timeline.

---

##### 6.10 Propagation Delay

Propagation delay is the time required for a signal to travel from a source to a receiving sensor.

Propagation delay is distinct from sensor latency.

```text
propagation delay = source to sensor travel time
sensor latency    = sensor/acquisition internal delay
```

AuralGlyph may store metadata needed to estimate propagation delay, such as:

* station position,
* sensor position,
* sensor orientation,
* medium,
* environmental conditions,
* timing accuracy,
* sensor latency,
* and uncertainty metadata.

However, AGF/CAS MUST NOT automatically assert source-emission time from observation time.

Source-time estimation, source localization, causal reconstruction, and delay correction are interpretive operations performed by downstream tools.

---

##### 6.11 Spatial Metadata

A Session MAY declare spatial metadata for stations, sensors, devices, or sources.

Example:

```yaml
stations:
  - station_id: "observation-station-a"
    station_label: "Observation Station A"
    position:
      coordinate_system: "WGS84"
      latitude_deg: 35.000000
      longitude_deg: -78.000000
      elevation_m: 40.0
```

Sensor-level spatial metadata MAY include:

```yaml
sensor:
  sensor_type: "microphone"
  station_id: "observation-station-a"
  position_reference: "station"
  orientation:
    azimuth_deg: 90
    elevation_deg: 0
```

If spatial metadata is approximate, estimated, intentionally obscured, or unavailable, that uncertainty SHOULD be declared.

A consumer MUST NOT assume precise sensor position or orientation unless the metadata supports that assumption.

---

##### 6.12 Derived Interpretations

A Session MAY reference derived analysis products.

Derived products may include:

* event detections,
* source-location estimates,
* classifications,
* alerts,
* annotations,
* confidence scores,
* fused signal products,
* reconstructed timelines,
* or human analyst notes.

Derived products MUST be clearly marked as derived or interpretive.

Example:

```yaml
derived_products:
  - product_id: "event-detection-001"
    product_type: "event-detection"
    source_cas_ids:
      - "station-a-audio-mic"
      - "station-b-seismic"
    derived: true
    interpretation: true
    confidence: 0.82
```

A derived product MUST NOT be presented as raw CAS observation data.

AuralGlyph stores observations and relationships. Interpretations may be stored, but they must remain distinguishable from measured data.

---

##### 6.13 Session Conformance

A Session conforms to this specification only if:

1. `session_id` is present,
2. `session_type` is present,
3. every member references an existing CAS instance,
4. timing claims are supported by timing metadata,
5. synchronization claims include uncertainty when known,
6. spatial claims are supported by spatial metadata when required,
7. derived products are clearly marked,
8. and unknown or estimated values are not presented as exact.

An implementation MUST reject, warn on, or mark as non-conformant any Session that claims synchronization, location, alignment, or interpretation without sufficient metadata.

A consumer MAY still display or inspect a non-conformant Session, but MUST NOT silently treat unsupported claims as precise or authoritative.

---

##### 6.14 Non-Normative Notes

A Session is the bridge between independent CAS instances.

For simple stereo audio, the Session may only need to state that two CAS instances are sample-synchronous left and right channels.

For a distributed sensor network, the Session may need station positions, clock references, timing uncertainties, sensor latency, integration windows, and environmental context.

The safest general rule is:

```text
CAS records what was measured.
Sessions describe how measurements relate.
Analysis decides what it means.
```

AuralGlyph does not standardize conclusions. It standardizes the preservation of observations and the metadata needed to reason about them.
