 A simple app that detects bumps and potholes on the road.

RoadBumps turns any phone into a road-quality sensor. As you drive, it reads the
device's motion sensors to detect sharp vertical spikes — the signature of a
pothole, expansion joint, or broken pavement — and tags each event with GPS
coordinates and a severity score. Aggregated across many trips, those events
render as a **heatmap of rough roads**, giving cities and contractors an
objective, crowdsourced map of where the pavement actually needs work.

---

## How it works

```
┌─────────────┐      ┌──────────────┐      ┌───────────────┐      ┌─────────────┐
│   Phone     │      │   Detection  │      │    Backend     │      │  Heatmap    │
│ accel + GPS │ ───▶ │  spike +     │ ───▶ │  store +       │ ───▶ │  visualize  │
│  sampling   │      │  threshold   │      │  cluster/agg   │      │  + export   │
└─────────────┘      └──────────────┘      └───────────────┘      └─────────────┘
```

1. **Sense** — Sample vertical acceleration from the phone's accelerometer
   (`DeviceMotionEvent`, or the Generic Sensor API where available). GPS position
   and speed are captured alongside each reading.
2. **Detect** — A high-pass filter + threshold flags vertical acceleration spikes.
   A bump produces a sharp spike that stands out from normal road vibration and
   suspension travel.
3. **Report** — Confirmed events `{ lat, lng, severity, speed, timestamp }` are
   batched and sent to the backend.
4. **Aggregate** — Nearby events are clustered into road segments so raw points
   become an actionable "this stretch is bad" signal rather than noise.
5. **Visualize** — Clustered severity renders as a heatmap; segment data can be
   exported for city maintenance departments and contractors.

---

## Features

- 📱 **Phone-based sensing** — no dedicated hardware; uses the accelerometer and
  GPS already in the device.
- 🕳️ **Bump / pothole detection** — vertical-acceleration spike detection with a
  configurable severity threshold.
- 🗺️ **Heatmap** — geospatial view of where the roughest roads are.
- 📊 **Exportable data** — aggregated segment data intended for cities and
  construction companies.

---

## Detection, in brief

A pothole hit shows up as a sharp spike in vertical (Z-axis) acceleration that
exceeds the surrounding baseline. The core loop is:

1. Sample vertical acceleration at a fixed rate.
2. High-pass filter to remove gravity/DC offset and slow body roll.
3. Flag samples whose magnitude exceeds a threshold as candidate events.
4. Attach the current GPS fix and speed to each event.

Severity scales with spike magnitude and is normalized against speed — the same
pothole reads very differently at 15 mph vs. 60 mph, so speed context matters for
comparability.

---

## Known caveats

These are real-world gotchas that shape the design:

- **iOS permission gate** — iOS requires a user gesture plus
  `DeviceMotionEvent.requestPermission()` before motion data is available.
- **GPS precision** — phone GPS is accurate to ~5–15 m, often wider than a single
  lane. Snapping events to the nearest road segment (map-matching) improves the
  map.
- **False positives** — speed bumps, railroad crossings, and hard braking can
  mimic potholes. A classification/heuristic layer is needed to filter these.
- **Speed dependence** — event severity must be interpreted relative to vehicle
  speed.

---
