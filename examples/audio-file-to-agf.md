# Converting a Stereo Audio File to AGF

This example is non-normative.

A conventional audio file such as WAV, FLAC, or MP3 is not stored directly as a CAS. An encoder first decodes the file into time-domain sample streams, then transforms each scalar stream into a CAS instance.

```text
song.flac
    ↓ decode
PCM sample streams
    ├─ left channel
    └─ right channel
    ↓ transform
AGF container
    ├─ CAS 0: left channel
    ├─ CAS 1: right channel
    └─ session metadata: stereo relationship and time alignment
```

To play or export the AGF file, a decoder performs the reverse process:

```text
AGF container
    ↓ read CAS instances
inverse transform
    ↓
PCM sample streams
    ├─ left channel
    └─ right channel
    ↓ encode/play
WAV / FLAC / MP3 / audio output
```

If the source file is MP3 or another lossy format, AGF can only preserve the decoded signal that remains after lossy compression. It cannot recover information already discarded by the original codec.

If the source file does not contain microphone or sensor details, the encoder MUST NOT invent them. It SHOULD record known channel roles, source format, and provenance instead.

## Example Metadata

```yaml
source_media_type: "audio/flac"
source_channel_count: 2

cas_instances:
  - cas_id: "cas-left"
    channel_role: "left"
    sensor_type: "unknown"

  - cas_id: "cas-right"
    channel_role: "right"
    sensor_type: "unknown"

session:
  session_type: "stereo-audio"
  members:
    - cas_id: "cas-left"
    - cas_id: "cas-right"
  alignment:
    time_relationship: "sample-synchronous"
```

The CAS instances remain independent scalar streams. The session metadata defines that they belong together and should be interpreted as synchronized stereo channels.
