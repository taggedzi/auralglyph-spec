# **AuralGlyph Development Journal — 2025-11-25**

## **Summary of Work Completed**

Today we refined and formalized the core of the AuralGlyph specification, particularly the **Canonical Audio Surface (CAS)** definition, metadata structure, and early spatial concepts. Significant clarity, rigor, and future-proofing were added, and the draft was promoted to a stable v0.1 experimental form pending two small micro-fixes.

---

# **1. CAS Formal Definition Completed**

### **Decisions Made**

* Finalized the structure of CAS as a **multi-band, complex-valued time–frequency tensor**.
* Confirmed CAS is an **abstract mathematical model**, not bound to a particular container format or data layout.

### **Why**

* A precise definition is required for interoperability.
* Keeps CAS mathematically clean and implementation-agnostic.

---

# **2. Adopted the “Single-Sensor CAS” Constraint**

### **Decision**

CAS represents **exactly one scalar signal from one physical or logical sensor**.

### **Rationale**

* Makes CAS atomic, predictable, and composable.
* Supports extremely broad sensor domains (audio, seismic, hydrophones, radio, astronomy).
* Enables powerful multi-sensor workflows *outside* CAS through grouping.
* Dramatically simplifies transform profiles and visualization logic.

### **Future-Proofing**

* Added explicit note that future versions *may* introduce aggregate types without breaking existing CAS semantics.

---

# **3. Added Full Sensor Metadata Layer**

### **Decision**

CAS metadata now includes:

* `sensor_id`
* `sensor_type`
* `reference_frame`
* `sensor_pose_model`
* `sensor_position`
* `sensor_orientation`
* `sensor_pose_track_id`

### **Rationale**

* Needed to support multi-sensor setups (arrays, boats, mobile recorders, astronomy).
* Required for future spatial reconstructions, time-of-arrival calculations, and synchronization.
* Ensures CAS is domain-agnostic and can be used in fields far beyond audio.

---

# **4. Introduced Pose Models (static / moving / unknown)**

### **Decision**

Sensors can have:

* `"static"` pose
* `"time-varying-external"` pose track
* `"unknown"` pose

### **Rationale**

* Needed for moving platforms (boats, drones, performers onstage).
* Avoids bloating CAS with per-frame positions.
* Keeps movement represented cleanly at the session/container level.

---

# **5. Added Time Synchronization Metadata**

### **Decision**

Time metadata now includes:

* `time_reference_type`
* `time_reference_value`
* `frame_zero_time_offset_sec`
* `time_sync_quality`

### **Why**

* Essential for coherent multi-sensor alignment.
* Supports absolute (UTC), relative-session, or relative-arbitrary recordings.
* Necessary for spatial processing (triangulation, localization, beamforming).

---

# **6. Added Unit Consistency Requirements**

### **Decision**

All physical quantities MUST use SI units unless declared otherwise.

### **Why**

* Prevents ambiguity across fields (audio, sonar, RF, seismic).
* Ensures smooth integration with scientific and engineering tools.

---

# **7. Added Cross-References and Consistency Fixes**

### **Changes**

* Added “See §3.5…” links for index sets, time axis, and bands.
* Clarified the relationship between metadata fields and CAS abstractions.

### **Why**

* Helps readers understand where conceptual definitions map into concrete metadata.
* Reduces confusion for implementers.

---

# **8. Improved §3.7: Multi-Sensor and Spatial Reconstruction**

### **Decision**

Rewrote section to:

* Emphasize one-CAS-per-sensor.
* Explain how arrays, hydrophone grids, seismic networks, and astronomical baselines are handled.
* Report how moving sensors refer to pose tracks.
* Clarify how spatial reconstructions produce either new CAS or time-domain signals.

### **Why**

* Ensures CAS stays clean.
* Moves array logic to container/session level where it belongs.

---

# **9. Minor Fixes & Cleanups**

* Corrected typos (“seiezmic” → “seismic”).
* Clarified band-mapping and transform wording.
* Ensured transform profiles list is consistent.

---

# **10. Versioning Decision**

### **Decision**

Once the two final micro-fixes are applied, the file should be renamed:

> **auralglyph-spec-v0.1.1.md**

### **Why**

* Follows semver-like patching semantics.
* Reflects small corrections without structural changes.
* Signals to readers that v0.1.1 is the authoritative version.

---

# **11. Outstanding Work for Tomorrow**

1. Apply the final two micro-fixes:

   * Correct blockquote formatting in the SI units section.
   * Clarify §3.7 bullet regarding static vs. moving sensor pose.

2. Rename the spec to `auralglyph-spec-v0.1.1.md`.

3. Begin the next major spec components:

   * **Container Manifest / Session Structure** (where pose tracks, multi-sensor grouping live).
   * Optional: Start sketching first transform profile (`AGF-STFT-48k-v1`).
