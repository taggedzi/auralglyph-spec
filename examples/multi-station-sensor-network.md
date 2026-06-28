# Multi-Station Sensor Network

This example is non-normative.

AuralGlyph can represent data from a distributed network of observation stations when each sensor output is modeled as one or more scalar CAS instances and the stations are related through session metadata.

This example describes a field observation network with multiple stations collecting sound, vibration, radio-frequency, thermal, environmental, and other sampled signals.

The purpose of this example is to show how AuralGlyph can store observations and relationships. It does not define intelligence conclusions, classification rules, command decisions, or analysis algorithms.

## Core Principle

AuralGlyph separates observation from interpretation.

```text
CAS stores observation.
AGF organizes observation.
Sessions relate observations.
External tools interpret observations.
```

An AGF container may contain many CAS instances from many sensors, but the meaning of those signals is determined by downstream analysis tools, machine learning models, human analysts, or domain-specific software.

## Example Scenario

A field observation network might include several fixed or mobile stations.

Each station may contain sensors such as:

* audible microphones,
* vibration or seismic sensors,
* radio-frequency receivers,
* radio-frequency intensity or receiver-derived streams,
* infrared intensity sensors,
* radiation counters,
* air-quality sensors,
* temperature sensors,
* pressure sensors,
* humidity sensors,
* or other scalar sampled instruments.

Each scalar sensor stream is represented as an independent CAS instance.

The AGF container uses session metadata to describe how those CAS instances relate.

## Example Container Structure

```text
field-observation.agf
├─ session: field-observation-session
│
├─ station: observation-station-a
│  ├─ CAS: station-a-audio-mic
│  ├─ CAS: station-a-seismic
│  ├─ CAS: station-a-rf-receiver
│  ├─ CAS: station-a-ir-intensity
│  ├─ CAS: station-a-radiation-count
│  └─ CAS: station-a-air-quality
│
├─ station: observation-station-b
│  ├─ CAS: station-b-audio-mic
│  ├─ CAS: station-b-seismic
│  ├─ CAS: station-b-rf-receiver
│  ├─ CAS: station-b-ir-intensity
│  └─ CAS: station-b-air-quality
│
└─ station: observation-station-c
   ├─ CAS: station-c-audio-mic
   ├─ CAS: station-c-seismic
   ├─ CAS: station-c-radiation-count
   └─ CAS: station-c-weather
```

## Session Metadata

The session metadata describes the observation network.

Example:

```yaml
session:
  session_id: "field-observation-session"
  session_type: "multi-station-sensor-network"

  time_reference:
    time_standard: "UTC"
    synchronization_method: "gnss"
    timing_accuracy_sec: 0.000001

  stations:
    - station_id: "observation-station-a"
      station_label: "Observation Station A"
      position:
        coordinate_system: "WGS84"
        latitude_deg: 35.000000
        longitude_deg: -78.000000
        elevation_m: 40.0

    - station_id: "observation-station-b"
      station_label: "Observation Station B"
      position:
        coordinate_system: "WGS84"
        latitude_deg: 35.010000
        longitude_deg: -78.020000
        elevation_m: 42.0

    - station_id: "observation-station-c"
      station_label: "Observation Station C"
      position:
        coordinate_system: "WGS84"
        latitude_deg: 34.990000
        longitude_deg: -77.980000
        elevation_m: 38.0
```

## CAS Membership

Each CAS instance remains an independent scalar signal stream.

The session defines which station produced each CAS and how the streams are related.

```yaml
members:
  - cas_id: "station-a-audio-mic"
    station_id: "observation-station-a"
    sensor_role: "audible-sound"
    time_relationship: "synchronized"

  - cas_id: "station-a-seismic"
    station_id: "observation-station-a"
    sensor_role: "ground-vibration"
    time_relationship: "synchronized"

  - cas_id: "station-a-rf-receiver"
    station_id: "observation-station-a"
    sensor_role: "radio-frequency-signal"
    time_relationship: "synchronized"

  - cas_id: "station-a-radiation-count"
    station_id: "observation-station-a"
    sensor_role: "radiation-count-rate"
    time_relationship: "synchronized"

  - cas_id: "station-b-audio-mic"
    station_id: "observation-station-b"
    sensor_role: "audible-sound"
    time_relationship: "synchronized"

  - cas_id: "station-b-seismic"
    station_id: "observation-station-b"
    sensor_role: "ground-vibration"
    time_relationship: "synchronized"
```

## Observation Time, Sensor Delay, and Source Time

AuralGlyph records measurements on the timeline declared by each CAS producer.

For sensor-network data, a CAS timestamp normally represents when the signal was observed, sampled, recorded, or corrected at the receiving sensor. It does not automatically represent the time at which the originating source event occurred.

This distinction matters because different signal types travel through the environment at different speeds, and different sensors may introduce different internal delays.

For example:

```text
source event
    ↓ propagation through environment
signal arrives at sensor
    ↓ sensor / electronics / firmware delay
sample is recorded
    ↓ encoder timeline choice
CAS time coordinate
```

A CAS may therefore describe one of several possible timelines:

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
```

or:

```yaml
sensor_timing:
  timestamp_basis: "latency-corrected-observation-time"
```

or:

```yaml
sensor_timing:
  timestamp_basis: "unknown"
```

The value of `timestamp_basis` SHOULD describe what the CAS time axis means.

Common values may include:

```yaml
timestamp_basis:
  - "recorded-sample-time"
  - "sensor-reported-time"
  - "latency-corrected-observation-time"
  - "estimated-arrival-time"
  - "unknown"
```

### Sensor Latency

A sensor or acquisition system may have internal delay.

Examples include:

* microphone preamp or ADC latency,
* radar receiver processing delay,
* camera or infrared detector frame delay,
* radiation counter integration interval,
* air-quality sensor response time,
* buffering or firmware delay,
* network transmission delay before recording.

When known, this SHOULD be declared.

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
  sensor_latency_sec: 0.0025
  latency_correction_applied: false
  latency_uncertainty_sec: 0.0002
```

This means the CAS samples are recorded on the raw recorded-sample timeline, and the producer knows the sensor chain introduces about 2.5 ms of latency.

If the producer corrected the timestamps before writing the CAS:

```yaml
sensor_timing:
  timestamp_basis: "latency-corrected-observation-time"
  sensor_latency_sec: 0.0025
  latency_correction_applied: true
  latency_uncertainty_sec: 0.0002
```

If the delay is unknown:

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
  sensor_latency_sec: null
  latency_correction_applied: false
  latency_uncertainty_sec: null
```

### Integration Windows

Some sensors do not measure an instantaneous value. They measure over an interval.

Examples include radiation counters, air-quality sensors, thermal sensors, chemical sensors, and some environmental instruments.

Such sensors SHOULD declare their integration or averaging window when known.

```yaml
sensor_timing:
  timestamp_basis: "recorded-sample-time"
  integration_window_sec: 1.0
  sample_time_reference: "window-center"
```

The `sample_time_reference` field SHOULD describe whether each timestamp refers to:

```yaml
sample_time_reference:
  - "window-start"
  - "window-center"
  - "window-end"
  - "instantaneous"
  - "unknown"
```

### Session-Level Timing Relationships

Each CAS may have its own timing behavior. The session metadata describes how CAS streams relate to one another.

```yaml
members:
  - cas_id: "station-a-audio-mic"
    station_id: "observation-station-a"
    sensor_role: "audible-sound"
    time_relationship: "synchronized"
    timing_offset_sec: 0.0000
    timing_uncertainty_sec: 0.0001

  - cas_id: "station-a-seismic"
    station_id: "observation-station-a"
    sensor_role: "ground-vibration"
    time_relationship: "synchronized"
    timing_offset_sec: 0.0125
    timing_uncertainty_sec: 0.0010

  - cas_id: "station-a-radiation-count"
    station_id: "observation-station-a"
    sensor_role: "radiation-count-rate"
    time_relationship: "sampled-independent"
    timing_offset_sec: null
    timing_uncertainty_sec: 0.5
```

This allows an AGF container to say that streams belong to the same observation session while still preserving differences in clock accuracy, sensor latency, acquisition timing, and sample meaning.

### Propagation Delay

Propagation delay is different from sensor latency.

```text
propagation delay = time required for the signal to travel from source to sensor
sensor latency    = delay inside the sensor or recording system
```

AuralGlyph may store metadata needed to estimate propagation delay, such as station position, sensor position, orientation, medium, environmental conditions, and timing accuracy.

However, the CAS itself does not automatically assert source time.

```text
CAS timestamp:
  when the signal was measured on the declared observation timeline

source time:
  estimated later by analysis, if possible
```

Source-time estimation, delay correction, localization, and causal reconstruction are interpretive operations performed by downstream tools.

The safest rule is:

```text
AuralGlyph records measurement timelines.
Correction, alignment, and source-time reconstruction must be explicit, not assumed.
```

## Example Sensor Metadata

A microphone CAS might include:

```yaml
cas_id: "station-a-audio-mic"
sensor:
  sensor_type: "microphone"
  station_id: "observation-station-a"
  measurement_type: "acoustic-pressure"
  units: "Pa"
  position_reference: "station"
  orientation:
    azimuth_deg: 90
    elevation_deg: 0
```

A seismic CAS might include:

```yaml
cas_id: "station-a-seismic"
sensor:
  sensor_type: "geophone"
  station_id: "observation-station-a"
  measurement_type: "ground-velocity"
  units: "m/s"
  axis: "vertical"
```

A radiation counter CAS might include:

```yaml
cas_id: "station-a-radiation-count"
sensor:
  sensor_type: "radiation-counter"
  station_id: "observation-station-a"
  measurement_type: "count-rate"
  units: "counts/s"
```

An air-quality CAS might include:

```yaml
cas_id: "station-a-air-quality"
sensor:
  sensor_type: "air-quality-sensor"
  station_id: "observation-station-a"
  measurement_type: "particulate-density"
  units: "ug/m3"
```

## Observation Versus Interpretation

The AGF container stores observed signal data and metadata.

Examples of observations:

```text
audio energy increased at station A
seismic vibration changed at station B
radiation count rate rose at station C
air-quality measurements changed over time
RF signal strength changed across stations
```

These are observations represented by CAS data and metadata.

Examples of interpretations:

```text
a vehicle was detected
a source was localized
a plume was inferred
an event was classified
a warning was generated
```

These are not CAS facts. They are conclusions produced by external interpretation.

AuralGlyph may store derived analysis products as additional metadata or derived CAS instances, but those products should be clearly marked as derived, interpretive, or non-authoritative unless validated by the application domain.

## Analysis Use

External tools may use the AGF data to perform tasks such as:

* time correlation,
* cross-station comparison,
* signal fusion,
* anomaly detection,
* source localization,
* environmental monitoring,
* visualization,
* machine learning inference,
* or human review.

These analyses are outside the core AuralGlyph specification.

The core format only provides a structured way to preserve the sampled observations, timing, station metadata, sensor metadata, and relationships needed by those tools.

## Important Limitation

AuralGlyph does not standardize command decisions, threat labels, target classification, tactical conclusions, or automated response behavior.

It standardizes the representation of observations and relationships.

The safest general rule is:

```text
AuralGlyph can preserve what was measured.
Analysis systems decide what it means.
```
