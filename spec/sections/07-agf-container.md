#### 7. AGF Container Structure

##### Document Summary

* **Purpose**: Defines the role and structure of an AuralGlyph File / AGF container.
* **Status**: Normative unless otherwise noted.
* **Applies to**: AGF files, container manifests, CAS storage, Session storage, metadata organization, references, provenance, and derived products.
* **Key concepts**:

  * An AGF container groups one or more CAS instances with metadata.
  * CAS instances remain independent scalar signal streams.
  * Sessions describe relationships among CAS instances.
  * Container metadata describes the AGF file as a whole.
  * Provenance metadata records where data came from and how it was produced.
  * Derived products may be stored, but must be clearly distinguished from raw observations.
  * A container may be simple, such as one CAS, or complex, such as a multi-station sensor dataset.
* **Defines**:

  * AGF container purpose
  * container identity
  * container manifest
  * CAS references
  * Session references
  * metadata organization
  * provenance metadata
  * derived product handling
  * external references
  * integrity metadata
  * logical container versus physical serialization
  * transport and networking boundaries
  * container conformance
* **Does not define**:

  * a final binary serialization format,
  * compression algorithms,
  * encryption behavior,
  * transport protocols,
  * rendering behavior,
  * analysis algorithms,
  * or application-specific interpretation models.

This section is **normative** unless otherwise noted.

An AGF container is the top-level structure used to package AuralGlyph data.

An AGF container MAY contain one CAS instance or many CAS instances. It MAY also contain Sessions, metadata, provenance, derived products, views, profiles, and references needed to interpret the contained data.

The AGF container does not change the meaning of CAS.

A CAS remains one scalar signal stream. The container provides structure around one or more CAS instances.

---

##### 7.1 Logical Container Versus Physical Serialization

In v0.1, an AGF container is a logical structure, not a required physical storage format.

An AGF container defines the required relationships among container metadata, CAS instances, Sessions, profiles, derived products, provenance, and references.

This specification does not require the container to be stored as a ZIP archive, directory tree, single binary file, database, stream, or network object.

Future AuralGlyph versions MAY define one or more physical serialization profiles, such as:

- `AGF-ZIP-V1`
- `AGF-DIR-V1`
- `AGF-STREAM-V1`
- `AGF-BINARY-V1`

A physical serialization profile MUST preserve the logical meaning of the AGF container model.

---

##### 7.2 Container Purpose

An AGF container provides a common structure for storing and exchanging AuralGlyph data.

It answers questions such as:

* What CAS instances are included?
* What Sessions are included?
* What metadata applies to the whole file?
* What metadata applies to specific CAS instances?
* How are CAS instances related?
* What transform, band, timing, sensor, and provenance metadata is available?
* Are there derived products or interpretive outputs?
* Are any data objects stored externally?
* What integrity or validation information is available?

A container MUST NOT imply that all contained CAS instances are related unless a Session or metadata relationship explicitly states that relationship.

---

---

##### 7.3 Container Identity

Each AGF container SHOULD declare a container identity.

Example:

```yaml
container:
  agf_version: "0.1"
  container_id: "agf-example-001"
  container_type: "multi-cas"
  created_at: "2026-06-28T00:00:00Z"
```

Common fields include:

```yaml
container:
  agf_version: <string>
  container_id: <string>
  container_type: <string>
  created_at: <timestamp>
  created_by: <string or null>
```

The `agf_version` field identifies the AuralGlyph specification version targeted by the container.

The `container_id` field SHOULD be stable within the producer’s context.

The `container_type` field SHOULD describe the general container purpose.

Examples:

```yaml
container_type: "single-cas"
```

```yaml
container_type: "multi-cas"
```

```yaml
container_type: "audio-recording"
```

```yaml
container_type: "sensor-session"
```

```yaml
container_type: "multi-station-observation"
```

```yaml
container_type: "derived-analysis-package"
```

---

---

##### 7.4 Container Manifest

An AGF container SHOULD include a manifest.

The manifest lists the major objects contained in or referenced by the container.

Example:

```yaml
manifest:
  cas_instances:
    - cas_id: "cas-left"
      path: "cas/cas-left.agcas"
    - cas_id: "cas-right"
      path: "cas/cas-right.agcas"

  sessions:
    - session_id: "stereo-song-session"
      path: "sessions/stereo-song-session.yaml"

  profiles:
    - profile_id: "AGF-STFT-BASE-V0"
      type: "transform-profile"
```

A manifest MAY list:

* CAS instances,
* Sessions,
* transform profiles,
* band profiles,
* view definitions,
* derived products,
* external resources,
* integrity records,
* provenance records,
* and application-specific extensions.

If a container includes multiple CAS instances, it SHOULD provide a manifest unless the serialization format already defines an equivalent index.

A consumer SHOULD use the manifest as the first source for discovering container contents.

---

---

##### 7.5 CAS Storage

Each CAS instance in an AGF container MUST have a unique `cas_id` within that container.

Example:

```yaml
cas_instances:
  - cas_id: "station-a-audio-mic"
    role: "audible-sound"
    path: "cas/station-a-audio-mic.agcas"

  - cas_id: "station-a-seismic"
    role: "ground-vibration"
    path: "cas/station-a-seismic.agcas"
```

A CAS instance MAY be stored:

* inline within the container metadata,
* as a separate internal object,
* as a referenced internal file,
* or as an external resource.

If a CAS instance is stored externally, the container MUST provide enough reference metadata for a consumer to locate or identify it.

A container MUST NOT store multiple unrelated scalar streams inside a single CAS instance.

Multi-channel, multi-sensor, multi-component, or multi-station data MUST be represented as multiple CAS instances related by Session metadata.

---

---

##### 7.6 Session Storage

Sessions MAY be stored inline or as separate internal objects.

Example:

```yaml
sessions:
  - session_id: "field-observation-session"
    session_type: "multi-station-sensor-network"
    path: "sessions/field-observation-session.yaml"
```

A Session member MUST reference an existing CAS instance by `cas_id`, unless the referenced CAS is external and explicitly declared as external.

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

A container MAY include CAS instances that do not belong to any Session.

A container MAY include multiple Sessions that reference the same CAS instance.

---

---

##### 7.7 Metadata Scope

AGF metadata MAY exist at multiple scopes.

Common scopes include:

```text
container-level metadata
session-level metadata
CAS-level metadata
band-level metadata
sample/coefficient-level metadata
derived-product metadata
```

Container-level metadata applies to the AGF package as a whole.

Session-level metadata applies to relationships among CAS instances.

CAS-level metadata applies to a single scalar signal stream.

Band-level metadata applies to a frequency region or symbolic band within a CAS.

Derived-product metadata applies to analysis outputs, annotations, or interpretive products.

A consumer MUST apply metadata according to its declared scope.

A metadata field declared at container scope MUST NOT be assumed to apply to every CAS instance unless the container explicitly states that inheritance rule.

---

---

##### 7.8 Metadata Inheritance

A container MAY define metadata inheritance rules.

Example:

```yaml
metadata_inheritance:
  container_time_reference_applies_to_all_sessions: true
  container_provenance_applies_to_all_cas: true
```

If inheritance rules are not declared, consumers MUST NOT assume inheritance.

For example, a container-level `time_reference` does not automatically mean every CAS instance is synchronized to that time reference unless the Session or CAS metadata supports that claim.

A producer SHOULD prefer explicit metadata over implicit inheritance when precision matters.

---

---

##### 7.9 Provenance Metadata

An AGF container SHOULD record provenance when known.

Provenance metadata describes where the data came from and how it was produced.

Example:

```yaml
provenance:
  source_type: "audio-file"
  source_media_type: "audio/flac"
  source_file_name: "song.flac"
  source_is_lossy: false
  encoder_name: "example-agf-encoder"
  encoder_version: "0.1.0"
  created_at: "2026-06-28T00:00:00Z"
```

For sensor data, provenance might include:

```yaml
provenance:
  source_type: "sensor-network"
  acquisition_system: "field-observation-recorder"
  acquisition_software_version: "0.1.0"
  operator: null
  processing_steps:
    - step: "raw-sample-ingest"
    - step: "stft-transform"
    - step: "agf-container-write"
```

If source metadata is unknown, a producer SHOULD use explicit unknown values rather than inventing data.

Example:

```yaml
sensor_type: "unknown"
source_file_name: null
operator: null
```

---

---

##### 7.10 Processing History

An AGF container MAY include processing history.

Processing history records transformations applied before or during container creation.

Example:

```yaml
processing_history:
  - step_id: "step-001"
    step_type: "decode-source-audio"
    input: "song.flac"
    output: "pcm-streams"
    software: "example-decoder"
    timestamp: "2026-06-28T00:00:00Z"

  - step_id: "step-002"
    step_type: "stft-transform"
    input: "pcm-streams"
    output: "cas-left, cas-right"
    transform_profile_id: "AGF-STFT-BASE-V0"

  - step_id: "step-003"
    step_type: "container-write"
    output: "song.agf"
```

Processing history SHOULD distinguish between reversible and irreversible steps when known.

Example:

```yaml
reversible: false
reason: "source MP3 decoding cannot recover discarded codec information"
```

A consumer MUST NOT assume that a processing step is reversible unless the metadata declares sufficient information for reversal.

---

---

##### 7.11 Derived Products

An AGF container MAY include derived products.

Derived products include:

* views,
* projections,
* annotations,
* event detections,
* classifications,
* source-location estimates,
* reconstructed timelines,
* rendered audio,
* rendered images,
* summary bands,
* ML embeddings,
* or other analysis outputs.

Derived products MUST be marked as derived.

Example:

```yaml
derived_products:
  - product_id: "event-detection-001"
    product_type: "event-detection"
    derived: true
    source_cas_ids:
      - "station-a-audio-mic"
      - "station-b-seismic"
    interpretation: true
    confidence: 0.82
```

A derived product MUST NOT be presented as raw CAS data.

If a derived product modifies coefficient data or creates a new signal stream, it SHOULD be represented as a new CAS instance or clearly marked derived data object.

---

---

##### 7.12 Views and Projections

An AGF container MAY include views or projections.

Views and projections provide ways to visualize or inspect CAS data.

Example:

```yaml
views:
  - view_id: "spectrogram-view"
    view_type: "spectrogram"
    source_cas_id: "cas-left"
    projection: "time-frequency-image"
```

Views MUST NOT alter the underlying CAS data.

A view that changes, filters, summarizes, or transforms signal data MUST be treated as derived data, not as the original CAS.

A consumer MUST NOT treat a view as equivalent to the source CAS unless the metadata explicitly states that it is a lossless representation.

---

---

##### 7.13 External References

An AGF container MAY reference external resources.

External resources may include:

* externally stored CAS data,
* calibration files,
* source files,
* profile definitions,
* sensor manifests,
* environmental metadata,
* related AGF containers,
* or external analysis outputs.

Example:

```yaml
external_references:
  - reference_id: "source-audio"
    reference_type: "source-file"
    uri: "file://song.flac"
    media_type: "audio/flac"

  - reference_id: "calibration-record"
    reference_type: "calibration-file"
    uri: "calibration/station-a-mic.yaml"
```

If an external reference is required to interpret or reconstruct data, the container MUST declare that dependency.

Example:

```yaml
required_for_reconstruction: true
```

A consumer MUST treat missing required external references as a conformance or completeness problem.

---

---

##### 7.14 Transport and Networking

AuralGlyph v0.1 does not define a required network transport protocol.

AGF containers and CAS data may be transmitted using any transport chosen by an implementation, including file transfer, HTTP, WebSocket, message queues, streaming protocols, or domain-specific systems.

The transport mechanism MUST NOT change the meaning of the contained CAS, Session, timing, transform, band, provenance, or derived-product metadata.

Implementations that transmit partial, streamed, chunked, or live AGF data SHOULD clearly declare:

- chunk ordering,
- timing basis,
- completeness status,
- missing-data behavior,
- retransmission behavior,
- compression behavior,
- encryption or authentication behavior when relevant,
- and whether the received data represents a complete AGF container or an incremental stream.

Future AuralGlyph versions MAY define optional transport profiles for live streaming, distributed sensor networks, or real-time analysis workflows.

Transport profiles MUST remain layered above the core AGF data model.

---

##### 7.15 Integrity Metadata

An AGF container SHOULD support integrity metadata.

Integrity metadata may include hashes, sizes, signatures, or validation records.

Example:

```yaml
integrity:
  objects:
    - object_id: "cas-left"
      path: "cas/cas-left.agcas"
      hash_algorithm: "sha256"
      hash_value: "<hex-string>"
      byte_length: <integer>
```

Integrity metadata may be used to detect corruption, incomplete transfer, or mismatched external references.

If integrity metadata is present, a consumer SHOULD validate it before relying on the associated data.

This specification does not require a specific hash algorithm for v0.1, but producers SHOULD use widely supported cryptographic hash algorithms when integrity matters.

---

---

##### 7.16 Privacy and Redaction

An AGF container MAY omit, reduce, obscure, or redact metadata for privacy, safety, security, proprietary, or operational reasons.

If metadata has been intentionally altered or withheld, the producer SHOULD declare that when possible.

Example:

```yaml
metadata_redaction:
  redacted: true
  reason: "location-privacy"
  affected_fields:
    - "stations.position"
```

A consumer MUST NOT assume that missing metadata means the information never existed.

Missing metadata may mean:

* unknown,
* unavailable,
* intentionally omitted,
* redacted,
* not applicable,
* or unsupported by the producer.

---

---

##### 7.17 Extensions

An AGF container MAY include extension metadata.

Extension metadata MUST be clearly identified.

Example:

```yaml
extensions:
  - extension_id: "custom:example-lab.environment-v0"
    extension_type: "environmental-metadata"
    data:
      temperature_c: 22.5
      humidity_percent: 55
```

Custom extensions SHOULD use identifiers beginning with:

```text
custom:
```

A consumer that does not understand an extension MUST NOT treat extension data as core AGF data.

A consumer MAY preserve unknown extensions when rewriting or transforming a container.

---

---

##### 7.18 Container Conformance

An AGF container conforms to this specification only if:

1. the declared `agf_version` is present,
2. every CAS instance has a unique `cas_id`,
3. every Session member references an existing or explicitly external CAS instance,
4. metadata scopes are unambiguous,
5. derived products are clearly marked,
6. reconstruction dependencies are declared when required,
7. unknown or redacted values are not presented as known,
8. and unsupported extensions are clearly identified.

An implementation MUST reject, warn on, or mark as non-conformant any container that makes unsupported claims about synchronization, reconstruction, provenance, interpretation, or metadata scope.

A consumer MAY display or inspect a non-conformant container, but MUST NOT silently treat unsupported claims as precise, reconstructable, authoritative, or complete.

---

---

##### 7.19 Non-Normative Notes

A minimal AGF container may contain only one CAS instance and basic metadata.

A complex AGF container may contain many CAS instances, multiple Sessions, derived analysis products, profile definitions, external references, integrity metadata, and privacy/redaction records.

The container is not the signal model itself.

The core relationship is:

```text
CAS stores scalar signal data.
Sessions describe relationships between CAS instances.
The AGF container packages CAS, Sessions, metadata, and related objects.
```

AuralGlyph should remain useful for simple audio-like workflows while also supporting complex scientific, industrial, environmental, spatial-audio, and sensor-network use cases.

---
