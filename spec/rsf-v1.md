# RSF v1.0 — Reverie Surface Format Specification

**Status:** Draft  
**Version:** 1.0.0  
**Date:** 2026  
**License:** CC0 1.0 Universal

---

## 1. Overview

RSF (Reverie Surface Format) is a JSON-based open standard for describing physical display surfaces. A surface is any real-world object that accepts and renders visual content.

RSF separates three concerns that all existing systems conflate:

1. **Identity** — what the surface is (permanent)
2. **State** — what the surface is doing (runtime)
3. **Content** — what the surface shows (cue-time)

---

## 2. File Format

RSF documents are UTF-8 encoded JSON. File extension: `.rsf.json`

Venue files (multiple surfaces) use extension: `.venue.rsf.json`

---

## 3. Root Object

```json
{
  "rsf_version": "1.0",
  "surface_id": "string",
  "display_name": "string",
  "category": "SurfaceCategory",
  "geometry": { ... },
  "position": { ... },
  "hardware": { ... },
  "content": { ... },
  "states": { ... },
  "vocabulary": { ... },
  "tags": ["string"],
  "meta": { ... }
}
```

### Required Fields

| Field | Type | Description |
|---|---|---|
| `rsf_version` | string | Spec version. Must be `"1.0"` |
| `surface_id` | string | Machine-readable unique identifier. Lowercase, underscores. Never changes. |
| `display_name` | string | Human-readable name shown in UI |
| `category` | SurfaceCategory | Surface role (see enum) |
| `geometry` | Geometry | Physical dimensions and shape |

### Optional Fields

All other root fields are optional. Implementations must not error on missing optional fields.

---

## 4. Enumerations

### 4.1 SurfaceCategory

The role of the surface in the installation.

| Value | Description |
|---|---|
| `primary` | Main content surface. Audience-facing. |
| `deck` | Floor or raked surface |
| `overhead` | Ceiling or above-audience surface |
| `remote` | Off-site or broadcast-only surface |
| `reference` | Internal/technical reference display |
| `specials` | Supplemental, non-primary audience surface |
| `custom` | User-defined role |

### 4.2 SurfaceShape

| Value | Description |
|---|---|
| `flat` | Planar surface |
| `curved` | Single-axis curve |
| `cylindrical` | Full or partial cylinder |
| `spherical` | Dome or sphere segment |
| `irregular` | Defined by mesh reference |

### 4.3 ContentFit

How content fills the surface when aspect ratios differ.

| Value | Description |
|---|---|
| `fill` | Scale to fill, crop if needed |
| `fit` | Scale to fit, letterbox if needed |
| `stretch` | Distort to fill exactly |
| `tile` | Repeat content to fill |
| `center` | Center at native resolution |

### 4.4 MissingBehavior

What happens when a cue assigns no content to this surface.

| Value | Description |
|---|---|
| `black` | Render black |
| `hold` | Hold last frame |
| `warn` | Render black and log warning |
| `block` | Prevent cue from firing |

### 4.5 SurfaceState (Runtime)

Runtime states are bitfield — a surface can be in multiple states simultaneously.

| Value | Description |
|---|---|
| `exists_in_library` | Known to platform. Permanent. |
| `enabled_in_venue` | Active in current physical installation |
| `visible_in_ui` | Shown to operator |
| `active_in_cue` | Has content assigned in current cue |
| `currently_playing` | Actively rendering |
| `overridden_live` | Operator has manual control |

---

## 5. Geometry Object

```json
{
  "geometry": {
    "width_px": 3840,
    "height_px": 1080,
    "width_m": 12.0,
    "height_m": 3.375,
    "shape": "flat",
    "orientation": "landscape",
    "aspect_ratio": "16:9",
    "curve_radius_m": null,
    "mesh_ref": null
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `width_px` | integer | Yes | Native width in pixels |
| `height_px` | integer | Yes | Native height in pixels |
| `width_m` | float | No | Physical width in metres |
| `height_m` | float | No | Physical height in metres |
| `shape` | SurfaceShape | No | Default: `flat` |
| `orientation` | string | No | `landscape`, `portrait`, `square` |
| `aspect_ratio` | string | No | Human-readable. e.g. `"16:9"` |
| `curve_radius_m` | float | No | Radius if shape is `curved` or `cylindrical` |
| `mesh_ref` | string | No | Path to mesh file if shape is `irregular` |

---

## 6. Position Object

World-space position of the surface. Coordinate system: Z-up, right-handed.

```json
{
  "position": {
    "x_m": -6.0,
    "y_m": 0.0,
    "z_m": 0.0,
    "rotation_x_deg": 0.0,
    "rotation_y_deg": 0.0,
    "rotation_z_deg": 0.0,
    "anchor": "center"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `x_m` | float | Horizontal offset from venue origin |
| `y_m` | float | Depth offset from venue origin |
| `z_m` | float | Vertical offset from venue origin |
| `rotation_x/y/z_deg` | float | Euler rotation in degrees |
| `anchor` | string | `center`, `bottom_left`, `top_left` |

---

## 7. Hardware Object

Physical hardware properties. Informational — does not affect content routing logic.

```json
{
  "hardware": {
    "type": "led_wall",
    "manufacturer": "string",
    "model": "string",
    "pixel_pitch_mm": 2.9,
    "brightness_nits": 2000,
    "refresh_rate_hz": 60,
    "hdr": true,
    "color_volume": "rec2020",
    "input_signal": "displayport_1.4",
    "processing_unit": "string"
  }
}
```

### Hardware Types

`led_wall` · `lcd_panel` · `projection_flat` · `projection_curved` · `projection_dome` · `ar_headset` · `holographic` · `architectural` · `medical_display` · `vehicle_display` · `custom`

---

## 8. Content Object

Rules for how content is assigned and displayed on this surface.

```json
{
  "content": {
    "accepts": ["video", "image", "ndi", "generative", "data_viz", "web"],
    "default_fit": "fill",
    "color_space": "rec709",
    "missing_behavior": "black",
    "max_fps": 60,
    "preferred_codec": "h264",
    "audio_passthrough": false
  }
}
```

### Content Types

| Value | Description |
|---|---|
| `video` | File-based video (mp4, mov, etc.) |
| `image` | Static image |
| `ndi` | NDI network stream |
| `sdi` | SDI hardware input |
| `generative` | Real-time generated content |
| `data_viz` | Data-driven visualization |
| `web` | Web renderer / CEF |
| `passthrough` | Direct hardware passthrough |

---

## 9. States Object

Declarative state rules. Distinct from runtime state (which is live).

```json
{
  "states": {
    "default_state": "enabled_in_venue",
    "visible_in_ui_default": true,
    "enabled_by_default": true,
    "allow_live_override": true,
    "blackout_participates": true,
    "save_with_show": true
  }
}
```

---

## 10. Vocabulary Object

Relabels surface identifiers for specific industries without changing underlying data.

```json
{
  "vocabulary": {
    "profile": "surgical_theater",
    "labels": {
      "primary": "Imaging Panel",
      "deck": "Floor Reference",
      "overhead": "Ceiling Navigation",
      "remote": "Consultation Screen",
      "reference": "Calibration Display"
    },
    "custom_fields": {
      "sterility_zone": "non_sterile",
      "regulatory_class": "class_ii"
    }
  }
}
```

### Standard Vocabulary Profiles

| Profile | Industry |
|---|---|
| `live_events` | Concerts, festivals, theater |
| `virtual_production` | Film and TV LED volume |
| `surgical_theater` | Medical and surgical navigation |
| `architecture` | Building visualization |
| `emergency_response` | EOC and tactical operations |
| `space_operations` | Mission control |
| `custom` | User-defined |

---

## 11. Venue File

A venue file describes all surfaces in a single physical installation.

```json
{
  "rsf_version": "1.0",
  "venue_id": "string",
  "venue_name": "string",
  "created": "ISO8601",
  "vocabulary_profile": "live_events",
  "coordinate_system": "z_up_right_handed",
  "unit_system": "metric",
  "surfaces": [
    { ... RSF surface object ... },
    { ... RSF surface object ... }
  ],
  "groups": [
    {
      "group_id": "upstage_walls",
      "display_name": "Upstage",
      "surface_ids": ["wall_a", "wall_b", "wall_c"]
    }
  ]
}
```

---

## 12. Complete Example

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
    "orientation": "landscape",
    "aspect_ratio": "32:9"
  },
  "position": {
    "x_m": -6.0,
    "y_m": 0.0,
    "z_m": 0.0,
    "rotation_z_deg": 0.0,
    "anchor": "center"
  },
  "hardware": {
    "type": "led_wall",
    "pixel_pitch_mm": 2.9,
    "brightness_nits": 2000,
    "refresh_rate_hz": 60,
    "hdr": true,
    "color_volume": "rec2020"
  },
  "content": {
    "accepts": ["video", "image", "ndi", "generative"],
    "default_fit": "fill",
    "color_space": "rec2020",
    "missing_behavior": "black",
    "max_fps": 60
  },
  "states": {
    "visible_in_ui_default": true,
    "enabled_by_default": true,
    "allow_live_override": true,
    "blackout_participates": true,
    "save_with_show": true
  },
  "tags": ["stage", "primary", "upstage", "audience_facing"]
}
```

---

## 13. Extensibility

Any implementation may add fields prefixed with `x_`:

```json
{
  "x_reverie_lumen_gi": true,
  "x_disguise_mapping_id": "layer_03",
  "x_brompton_panel_id": "BR-001"
}
```

Unknown `x_` fields must be preserved by parsers and ignored by validators.

---

## 14. Versioning

RSF uses semantic versioning. Breaking changes increment the major version. The `rsf_version` field in every document enables forward compatibility.

Parsers must reject documents with unknown major versions.

---

*RSF is published by Reverie under CC0 1.0 Universal. No rights reserved.*
