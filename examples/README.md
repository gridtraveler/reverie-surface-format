# RSF — Reverie Surface Format

**An open standard for describing any real-world display surface.**  
Industry-agnostic. Hardware-agnostic. Free forever.

---

> *Physical space has no operating system. RSF is the first step toward one.*

---

## The Problem

Every LED wall, projection screen, AR overlay, architectural facade, surgical display, and dome installation speaks a different language. There is no universal way to say:

> *"This surface exists. It is here. It is this shape. It accepts this content. It behaves this way."*

So every tool reinvents it. Every studio hard-codes it. Every integration breaks when the hardware changes.

RSF solves this once.

---

## What RSF Is

RSF is a JSON-based specification that describes any physical display surface in a format any tool can read, any engine can interpret, and any operator can understand.

One surface. One file. Works everywhere.

```json
{
  "rsf_version": "1.0",
  "surface_id": "wall_stage_left",
  "display_name": "Stage Left LED",
  "category": "primary",
  "geometry": {
    "width_px": 3840,
    "height_px": 1080,
    "width_m": 12.0,
    "height_m": 3.375,
    "shape": "flat",
    "orientation": "landscape"
  },
  "position": {
    "x_m": -6.0,
    "y_m": 0.0,
    "z_m": 0.0,
    "rotation_deg": 0.0
  },
  "hardware": {
    "type": "led_wall",
    "pixel_pitch_mm": 2.9,
    "brightness_nits": 2000,
    "refresh_rate_hz": 60,
    "hdr": true
  },
  "content": {
    "accepts": ["video", "image", "ndi", "generative", "data_viz"],
    "default_fit": "fill",
    "color_space": "rec2020",
    "missing_behavior": "warn"
  },
  "tags": ["stage", "primary", "upstage"]
}
```

---

## Why This Matters

| Before RSF | After RSF |
|---|---|
| Every tool describes surfaces differently | One format every tool reads |
| Changing hardware breaks integrations | Hardware details are separate from surface identity |
| Industry-specific jargon locks out other fields | Universal vocabulary works across film, events, medicine, architecture |
| Operators re-map surfaces for every show | Surface definitions travel with the show file |
| No standard for spatial content routing | Any engine can route content to any surface |

---

## The Six Surface States

RSF models the full lifecycle of a surface — from library to live override:

```
ExistsInLibrary      → known to the platform, permanent
EnabledInVenue       → active for this physical installation  
VisibleInUI          → shown to the operator
ActiveInCue          → assigned content in the current cue
CurrentlyPlaying     → rendering now
OverriddenLive       → operator has taken manual control
```

A surface is never just "on" or "off". It has a position in the hierarchy. This is what makes RSF professional-grade.

---

## Supported Surface Types

RSF describes any physical surface that accepts visual content:

- **LED walls** — direct view, any pixel pitch
- **Projection surfaces** — flat, curved, dome, irregular
- **AR/XR overlays** — headset, holographic, mixed reality
- **Architectural facades** — building exteriors, permanent installations
- **Medical displays** — surgical navigation, imaging panels
- **Vehicle surfaces** — cockpit displays, heads-up, cabin
- **Custom geometry** — any surface defined by mesh

---

## Vocabulary System

RSF includes a vocabulary layer that relabels surfaces for any industry without changing the underlying data:

```json
{
  "vocabulary": "surgical_theater",
  "labels": {
    "primary": "Imaging Panel",
    "deck": "Floor Reference",
    "overhead": "Ceiling Navigation",
    "remote": "Consultation Screen"
  }
}
```

The same RSF file. The same engine. A completely different world.

---

## Specification

→ [RSF v1.0 Specification](spec/rsf-v1.md)  
→ [JSON Schema](schema/rsf.schema.json)  
→ [Examples](examples/)

---

## Implementations

| Tool | Status | Link |
|---|---|---|
| **Reverie** (Unreal Engine 5) | ✅ Reference implementation | [reverie.studio](https://reverie.studio) |
| Your tool here | — | Open a PR |

RSF is designed to be implemented by anyone. If you build an RSF reader or writer, open a PR to add it here.

---

## Roadmap

- **v1.0** — Core surface descriptor (geometry, hardware, content rules) ✅
- **v1.1** — Multi-surface venue files (describe entire installations)
- **v1.2** — Temporal layer (surface state over time)
- **v2.0** — Spatial OS bindings (surface-aware content routing protocol)

---

## Contributing

RSF is governed by an open process. Proposals, corrections, and implementations welcome.

```bash
git clone https://github.com/reverie-platform/reverie-surface-format
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for the proposal process.

---

## License

RSF specification: [CC0 1.0 Universal](LICENSE) — no rights reserved.  
Free for any use, commercial or otherwise, forever.

---

*Published by [Reverie](https://reverie.studio) · Built with Unreal Engine 5*  
*The physical world deserves an operating system.*
