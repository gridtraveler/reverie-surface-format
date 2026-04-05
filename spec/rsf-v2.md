# RSF v2.0 — Reverie Surface Format Specification

**Status:** Draft  
**Version:** 2.0.0  
**License:** CC0 1.0 Universal

> *v1 describes what a surface IS. v2 describes what it knows, feels, responds to, and remembers.*

---

## The Leap from v1 to v2

Every existing system — disguise, Notch, QLab, every LED processor, every media server — describes surfaces the same way: pixels wide, pixels tall, maybe a hardware model number. That's a spec sheet, not intelligence.

RSF v2 introduces six new dimensions that have never existed in any standard:

| Dimension | What it captures |
|---|---|
| **Identity** (v1) | What the surface IS |
| **Physics Manifest** (NEW) | How light actually behaves on this surface |
| **Audience Model** (NEW) | Who is watching and with what optics |
| **Semantic Zones** (NEW) | What the surface knows about itself |
| **Intelligence Layer** (NEW) | What the surface responds to |
| **Adjacency Graph** (NEW) | How the surface relates to its neighbors |
| **Temporal Schema** (NEW) | What the surface remembers |

---

## 1. Identity (v1 — unchanged)

```json
{
  "rsf_version": "2.0",
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
    "x_m": -6.0, "y_m": 0.0, "z_m": 0.0,
    "rotation_z_deg": 0.0,
    "anchor": "center"
  },
  "hardware": {
    "type": "led_wall",
    "pixel_pitch_mm": 2.9,
    "brightness_nits": 2000,
    "refresh_rate_hz": 3840,
    "hdr": true
  },
  "content": {
    "accepts": ["video", "image", "ndi", "generative"],
    "default_fit": "fill",
    "missing_behavior": "black"
  }
}
```

---

## 2. Physics Manifest ★ NEW

**The photometric fingerprint of a surface.**

Every LED wall, projection screen, and display has a unique optical signature — color primaries, emission model, peak and black luminance, viewing cone geometry. Currently this data lives in manufacturer PDFs and Brompton processor configs. It has never been part of a surface description standard.

When tools read a surface's Physics Manifest, content can be color-graded to the actual physical display before a single pixel renders. The "looks right on my monitor but wrong on the wall" problem disappears permanently.

```json
{
  "physics": {
    "color_primaries": {
      "red":   [0.708, 0.292],
      "green": [0.170, 0.797],
      "blue":  [0.131, 0.046],
      "white": [0.3127, 0.3290]
    },
    "color_space": "rec2020",
    "transfer_function": "pq",
    "peak_luminance_nits": 2000,
    "black_luminance_nits": 0.001,
    "contrast_ratio": 2000000,
    "bit_depth": 16,
    "viewing_cone": {
      "horizontal_deg": 160,
      "vertical_deg": 140,
      "optimal_angle_deg": 0
    },
    "emission_model": "lambertian",
    "moire_risk": {
      "camera_shutter_hz_min": 48,
      "safe_refresh_rate_hz": 3840,
      "notes": "Refresh rate verified moire-free at 24fps, T1.4"
    },
    "calibration": {
      "profile_id": "brompton_sx40_2025_03_15",
      "calibrated_date": "2025-03-15",
      "calibration_tool": "Brompton Tessera SX40",
      "delta_e_target": 2.0,
      "delta_e_achieved": 0.8
    },
    "power": {
      "peak_watts": 2400,
      "typical_watts": 800,
      "watts_per_sqm_peak": 800,
      "carbon_kg_co2_per_kwh": 0.233
    }
  }
}
```

**Why this is unprecedented:** Brompton, Megapixel VR, and disguise all capture this data internally. None of them export it in a standard format. RSF is the first standard to give this data a home.

---

## 3. Audience Model ★ NEW

**A surface that knows who is watching, from where, and with what optics.**

This is the most radical addition in RSF v2. Currently, no standard captures the relationship between a surface and the humans or cameras watching it. But this relationship changes everything:

- A camera at T1.4 and f35mm 8 meters from a 2.9mm pitch wall has a specific moire profile
- An audience of 2,000 people at 30–80 meters needs different content motion budgets than a close-up camera
- A surgeon 60cm from a display needs different contrast behavior than a conference room

When a surface knows its audience, AI content generators, render engines, and color scientists can make correct decisions automatically — without manual per-show configuration.

```json
{
  "audience": {
    "viewers": [
      {
        "viewer_id": "front_of_house",
        "type": "human",
        "count_estimate": 500,
        "distance_min_m": 15.0,
        "distance_max_m": 80.0,
        "elevation_deg": 2.0,
        "seating": "fixed",
        "notes": "General admission floor, festival layout"
      },
      {
        "viewer_id": "camera_a",
        "type": "camera",
        "count_estimate": 1,
        "distance_m": 8.0,
        "elevation_deg": 0.5,
        "camera": {
          "sensor_format": "full_frame",
          "sensor_width_mm": 36,
          "fps": 24,
          "shutter_angle_deg": 180,
          "lens_focal_length_mm": 35,
          "aperture": "T1.4",
          "primary_use": "broadcast"
        }
      },
      {
        "viewer_id": "camera_b_insert",
        "type": "camera",
        "distance_m": 2.5,
        "camera": {
          "sensor_format": "super_35",
          "fps": 48,
          "lens_focal_length_mm": 85,
          "aperture": "T2.8",
          "primary_use": "close_up"
        }
      }
    ],
    "frustum": {
      "primary_viewer_id": "camera_a",
      "in_frustum_rect": {"x": 0.1, "y": 0.05, "w": 0.8, "h": 0.9},
      "notes": "Central 80% of wall is in primary camera frustum at this lens"
    }
  }
}
```

---

## 4. Semantic Zones ★ NEW

**The surface divided into named regions with meaning and rules.**

This is what makes AI understand surfaces. Currently AI image generators and content tools treat LED walls as blank canvases. They have no way to know that the top 30% of a wall is always sky, that the center zone must never contain high-motion content while cameras are rolling, or that a surgical display's left panel is reserved for imaging data.

Semantic Zones are named regions with bounds, semantic labels, and content rules. Any tool that reads RSF — an AI generator, a media server, a color pipeline — can make correct decisions without human intervention.

```json
{
  "zones": [
    {
      "zone_id": "sky",
      "display_name": "Sky zone",
      "bounds": {
        "x_normalized": 0.0,
        "y_normalized": 0.0,
        "w_normalized": 1.0,
        "h_normalized": 0.35
      },
      "semantic": "background_sky",
      "content_rules": {
        "allow_faces": false,
        "allow_text": false,
        "allow_logos": false,
        "motion_budget_max_percent": 15,
        "preferred_content_tags": ["sky", "clouds", "atmosphere", "stars"],
        "forbidden_content_tags": ["ground", "urban", "faces"]
      },
      "camera_notes": "High-motion content here causes strobing in camera at 24fps"
    },
    {
      "zone_id": "character_space",
      "display_name": "Character interaction zone",
      "bounds": {
        "x_normalized": 0.1,
        "y_normalized": 0.25,
        "w_normalized": 0.8,
        "h_normalized": 0.65
      },
      "semantic": "character_interaction",
      "content_rules": {
        "allow_faces": true,
        "allow_text": true,
        "min_contrast_ratio": 4.5,
        "motion_budget_max_percent": 40,
        "maintain_horizon_line": true,
        "horizon_y_normalized": 0.6
      }
    },
    {
      "zone_id": "ground_plane",
      "display_name": "Ground / floor zone",
      "bounds": {
        "x_normalized": 0.0,
        "y_normalized": 0.75,
        "w_normalized": 1.0,
        "h_normalized": 0.25
      },
      "semantic": "background_ground",
      "content_rules": {
        "allow_faces": false,
        "motion_budget_max_percent": 10,
        "preferred_content_tags": ["ground", "pavement", "grass", "floor"]
      }
    },
    {
      "zone_id": "data_overlay",
      "display_name": "Data / HUD overlay",
      "bounds": {
        "x_normalized": 0.02,
        "y_normalized": 0.02,
        "w_normalized": 0.25,
        "h_normalized": 0.15
      },
      "semantic": "information_layer",
      "content_rules": {
        "allow_text": true,
        "allow_data_visualization": true,
        "allow_faces": false,
        "min_contrast_ratio": 7.1,
        "text_size_min_pt": 18
      }
    }
  ]
}
```

---

## 5. Intelligence Layer ★ NEW

**A surface that responds.**

This is the bridge between a static surface description and a live intelligent actor. The Intelligence Layer declares what external signals a surface responds to, and what it emits.

No standard has ever captured this. Currently every OSC binding, every timecode sync, every DMX trigger is configured manually in each piece of software, re-done for every show. With RSF Intelligence, a surface carries its bindings with it. Any RSF-compatible tool can configure the surface correctly without operator intervention.

```json
{
  "intelligence": {
    "triggers": [
      {
        "trigger_id": "brightness_osc",
        "source": "osc",
        "address": "/surface/wall_a/brightness",
        "type": "float",
        "range": [0.0, 1.0],
        "action": "set_brightness_multiplier",
        "notes": "Controlled by grandMA3 via OSC bridge"
      },
      {
        "trigger_id": "timecode_sync",
        "source": "timecode",
        "format": "smpte_25",
        "action": "sync_content_position",
        "offset_frames": 0
      },
      {
        "trigger_id": "ambient_light_compensation",
        "source": "sensor",
        "sensor_type": "ambient_light_lux",
        "sensor_id": "lux_sensor_foh",
        "action": "auto_brightness_compensation",
        "response_curve": "logarithmic"
      },
      {
        "trigger_id": "ai_attention_zone",
        "source": "ai_inference",
        "model": "audience_attention_map",
        "action": "dynamic_zone_brightness_emphasis",
        "latency_budget_ms": 33
      },
      {
        "trigger_id": "camera_frustum_enter",
        "source": "tracking",
        "tracking_system": "mo_sys",
        "event": "camera_frustum_enter",
        "action": "set_refresh_rate",
        "action_value": 3840
      }
    ],
    "outputs": [
      {
        "output_id": "state_broadcast",
        "type": "osc",
        "address": "/surface/wall_a/state",
        "emits": ["current_content_id", "brightness_level", "active_zone_ids", "playing_state"]
      },
      {
        "output_id": "telemetry",
        "type": "mqtt",
        "topic": "reverie/surfaces/wall_a/telemetry",
        "interval_ms": 100,
        "emits": ["brightness_nits", "temperature_c", "frame_count", "error_flags"]
      }
    ],
    "automation": {
      "allow_ai_control": true,
      "ai_control_scope": ["brightness", "zone_emphasis", "content_selection"],
      "require_human_approval_for": ["content_change", "blackout"],
      "emergency_override": "blackout_immediate"
    }
  }
}
```

---

## 6. Adjacency Graph ★ NEW

**How surfaces relate to their neighbors.**

Three LED walls sharing a stage aren't independent objects — they form a topology. Content can cross from Wall A to Wall B. The ceiling panel bounces light onto actors. Wall B's right edge meets Wall C's left edge with a 3mm gap requiring seam correction.

No standard has ever modeled this. Disguise does it internally. QLab ignores it. nDisplay approximates it. RSF makes it explicit and portable.

```json
{
  "adjacency": [
    {
      "neighbor_surface_id": "wall_b_center",
      "my_edge": "right",
      "neighbor_edge": "left",
      "gap_mm": 0,
      "physical_seam": true,
      "seam_correction": {
        "enabled": true,
        "method": "auto",
        "blend_width_px": 4,
        "notes": "Panels share a cabinet edge. Brompton handles seam calibration."
      },
      "content_continuity": {
        "enabled": true,
        "coordinate_space": "shared_venue",
        "content_wraps": true
      }
    },
    {
      "neighbor_surface_id": "ceiling_panel",
      "my_edge": "top",
      "neighbor_edge": "bottom",
      "gap_mm": 600,
      "physical_seam": false,
      "content_continuity": {
        "enabled": false
      },
      "light_interaction": {
        "role": "bounce_source",
        "my_role_for_neighbor": "illumination_target",
        "estimated_bounce_factor": 0.15,
        "color_cast": "neutral",
        "notes": "Ceiling panel provides practical fill light onto talent below"
      }
    },
    {
      "neighbor_surface_id": "wall_c_right",
      "my_edge": "left",
      "neighbor_edge": "right",
      "gap_mm": 0,
      "physical_seam": true,
      "seam_correction": {
        "enabled": true,
        "method": "manual",
        "blend_width_px": 2
      }
    }
  ]
}
```

---

## 7. Temporal Schema ★ NEW

**A surface that remembers.**

Every show run through Reverie leaves a temporal record — frame-accurate, indexed, exportable. This is the dataset that doesn't exist anywhere. Thousands of shows. Millions of surface-hours. The training data for the first AI that understands spatial storytelling.

The Temporal Schema declares how a surface records itself and what it exports.

```json
{
  "temporal": {
    "record": {
      "enabled": true,
      "resolution": "frame",
      "what": ["content_id", "brightness", "zone_states", "trigger_events", "audience_data"],
      "timecode_sync": "smpte_ltc",
      "timecode_format": "25fps"
    },
    "retention": {
      "policy": "indefinite",
      "storage_location": "reverie_cloud",
      "local_cache_duration_hours": 24
    },
    "export": {
      "formats": ["rsf_timeline", "edl", "csv", "json_stream"],
      "api_endpoint": "https://api.reverie.studio/v1/surfaces/{surface_id}/timeline"
    },
    "playback": {
      "enabled": true,
      "scrub_accuracy": "frame",
      "notes": "Full show replay for client review and post-analysis"
    },
    "analytics": {
      "content_performance_tracking": true,
      "zone_attention_heatmap": true,
      "transition_analysis": true,
      "export_for_ai_training": true,
      "privacy_compliant": true
    }
  }
}
```

---

## Complete RSF v2 Document

A full surface definition combining all seven dimensions:

```json
{
  "rsf_version": "2.0",
  "surface_id": "wall_a",
  "display_name": "Wall A — Stage Left",
  "category": "primary",
  "geometry": { "width_px": 3840, "height_px": 1080, "width_m": 12.0, "height_m": 3.375, "shape": "flat" },
  "position": { "x_m": -6.0, "y_m": 0.0, "z_m": 0.0, "anchor": "center" },
  "hardware": { "type": "led_wall", "pixel_pitch_mm": 2.9, "brightness_nits": 2000, "refresh_rate_hz": 3840, "hdr": true },
  "content": { "accepts": ["video", "image", "ndi", "generative"], "default_fit": "fill", "missing_behavior": "black" },
  "physics": {
    "color_primaries": { "red": [0.708, 0.292], "green": [0.170, 0.797], "blue": [0.131, 0.046], "white": [0.3127, 0.3290] },
    "color_space": "rec2020", "peak_luminance_nits": 2000, "black_luminance_nits": 0.001,
    "viewing_cone": { "horizontal_deg": 160, "vertical_deg": 140 },
    "moire_risk": { "safe_refresh_rate_hz": 3840 },
    "power": { "peak_watts": 2400, "typical_watts": 800 }
  },
  "audience": {
    "viewers": [
      { "viewer_id": "foh", "type": "human", "count_estimate": 500, "distance_min_m": 15, "distance_max_m": 80 },
      { "viewer_id": "cam_a", "type": "camera", "distance_m": 8, "camera": { "sensor_format": "full_frame", "fps": 24, "lens_focal_length_mm": 35, "aperture": "T1.4" } }
    ]
  },
  "zones": [
    { "zone_id": "sky", "bounds": { "x_normalized": 0, "y_normalized": 0, "w_normalized": 1, "h_normalized": 0.35 }, "semantic": "background_sky", "content_rules": { "allow_faces": false, "motion_budget_max_percent": 15 } },
    { "zone_id": "character_space", "bounds": { "x_normalized": 0.1, "y_normalized": 0.25, "w_normalized": 0.8, "h_normalized": 0.65 }, "semantic": "character_interaction", "content_rules": { "allow_faces": true, "min_contrast_ratio": 4.5 } }
  ],
  "intelligence": {
    "triggers": [
      { "trigger_id": "timecode", "source": "timecode", "format": "smpte_25", "action": "sync_content_position" },
      { "trigger_id": "osc_brightness", "source": "osc", "address": "/surface/wall_a/brightness", "type": "float", "action": "set_brightness_multiplier" }
    ],
    "outputs": [
      { "output_id": "state", "type": "osc", "address": "/surface/wall_a/state", "emits": ["current_content_id", "brightness_level"] }
    ]
  },
  "adjacency": [
    { "neighbor_surface_id": "wall_b", "my_edge": "right", "neighbor_edge": "left", "gap_mm": 0, "seam_correction": { "enabled": true, "method": "auto" }, "content_continuity": { "enabled": true } }
  ],
  "temporal": {
    "record": { "enabled": true, "resolution": "frame", "timecode_sync": "smpte_ltc" },
    "retention": { "policy": "indefinite" },
    "analytics": { "content_performance_tracking": true, "export_for_ai_training": true }
  },
  "vocabulary": { "profile": "live_events" },
  "tags": ["stage", "primary", "upstage", "audience_facing"]
}
```

---

## What This Unlocks

| Layer | Problem it eliminates |
|---|---|
| Physics Manifest | "It looked right on my monitor" — forever gone |
| Audience Model | Manual per-show camera moire configuration |
| Semantic Zones | AI guessing where to place content |
| Intelligence Layer | OSC/timecode re-configuration for every show |
| Adjacency Graph | Seam correction and light interaction being proprietary |
| Temporal Schema | Show data disappearing after the event |

---

*RSF v2.0 is published by Reverie under CC0 1.0 Universal. No rights reserved. Build on it.*
