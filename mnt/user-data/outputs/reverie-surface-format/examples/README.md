# RSF Examples

Real-world surface definitions across industries.

---

## Live Events — Three-Wall LED Stage

`examples/live-events-three-wall.venue.rsf.json`

```json
{
  "rsf_version": "1.0",
  "venue_id": "stage_15",
  "venue_name": "Stage 15 — Three Wall LED",
  "vocabulary_profile": "live_events",
  "surfaces": [
    {
      "rsf_version": "1.0",
      "surface_id": "wall_a",
      "display_name": "Wall A — Upstage Left",
      "category": "primary",
      "geometry": { "width_px": 3840, "height_px": 1080, "width_m": 8.0, "height_m": 2.25, "shape": "flat" },
      "content": { "accepts": ["video", "image", "ndi"], "default_fit": "fill", "missing_behavior": "black" },
      "tags": ["upstage", "left", "primary"]
    },
    {
      "rsf_version": "1.0",
      "surface_id": "wall_b",
      "display_name": "Wall B — Upstage Center",
      "category": "primary",
      "geometry": { "width_px": 3840, "height_px": 1080, "width_m": 12.0, "height_m": 2.25, "shape": "flat" },
      "content": { "accepts": ["video", "image", "ndi"], "default_fit": "fill", "missing_behavior": "black" },
      "tags": ["upstage", "center", "primary"]
    },
    {
      "rsf_version": "1.0",
      "surface_id": "wall_c",
      "display_name": "Wall C — Upstage Right",
      "category": "primary",
      "geometry": { "width_px": 3840, "height_px": 1080, "width_m": 8.0, "height_m": 2.25, "shape": "flat" },
      "content": { "accepts": ["video", "image", "ndi"], "default_fit": "fill", "missing_behavior": "black" },
      "tags": ["upstage", "right", "primary"]
    }
  ],
  "groups": [
    { "group_id": "all_upstage", "display_name": "All Upstage", "surface_ids": ["wall_a", "wall_b", "wall_c"] }
  ]
}
```

---

## Virtual Production — LED Volume

```json
{
  "rsf_version": "1.0",
  "venue_id": "led_volume_stage",
  "venue_name": "Virtual Production Volume",
  "vocabulary_profile": "virtual_production",
  "surfaces": [
    {
      "rsf_version": "1.0",
      "surface_id": "background_wall",
      "display_name": "Background Wall",
      "category": "primary",
      "geometry": { "width_px": 7680, "height_px": 2160, "shape": "curved", "curve_radius_m": 9.0 },
      "hardware": { "type": "led_wall", "pixel_pitch_mm": 2.8, "brightness_nits": 2500, "hdr": true, "color_volume": "rec2020" },
      "content": { "accepts": ["video", "generative"], "default_fit": "fill", "color_space": "rec2020" },
      "tags": ["background", "icvfx", "primary"]
    },
    {
      "rsf_version": "1.0",
      "surface_id": "ceiling_panel",
      "display_name": "Ceiling LED",
      "category": "overhead",
      "geometry": { "width_px": 3840, "height_px": 1920, "shape": "flat" },
      "hardware": { "type": "led_wall", "pixel_pitch_mm": 3.9 },
      "content": { "accepts": ["video", "generative"], "default_fit": "fill" },
      "tags": ["ceiling", "overhead", "bounce"]
    }
  ]
}
```

---

## Surgical Theater

```json
{
  "rsf_version": "1.0",
  "venue_id": "or_suite_7",
  "venue_name": "OR Suite 7",
  "vocabulary_profile": "surgical_theater",
  "surfaces": [
    {
      "rsf_version": "1.0",
      "surface_id": "imaging_panel_primary",
      "display_name": "Imaging Panel",
      "category": "primary",
      "geometry": { "width_px": 3840, "height_px": 2160, "width_m": 1.2, "height_m": 0.675, "shape": "flat" },
      "hardware": { "type": "medical_display", "brightness_nits": 1000, "hdr": true },
      "content": { "accepts": ["data_viz", "video", "generative"], "color_space": "rec709", "missing_behavior": "black" },
      "vocabulary": {
        "profile": "surgical_theater",
        "custom_fields": { "sterility_zone": "non_sterile", "regulatory_class": "class_ii" }
      },
      "tags": ["imaging", "primary", "surgeon_facing"]
    },
    {
      "rsf_version": "1.0",
      "surface_id": "consultation_screen",
      "display_name": "Consultation Screen",
      "category": "remote",
      "geometry": { "width_px": 1920, "height_px": 1080, "shape": "flat" },
      "hardware": { "type": "lcd_panel" },
      "content": { "accepts": ["video", "data_viz"], "missing_behavior": "hold" },
      "tags": ["remote", "consultation", "observer"]
    }
  ]
}
```

---

## Emergency Operations Center

```json
{
  "rsf_version": "1.0",
  "venue_id": "eoc_main",
  "venue_name": "Emergency Operations Center",
  "vocabulary_profile": "emergency_response",
  "surfaces": [
    {
      "rsf_version": "1.0",
      "surface_id": "situation_wall",
      "display_name": "Situation Wall",
      "category": "primary",
      "geometry": { "width_px": 7680, "height_px": 2160, "shape": "flat" },
      "content": { "accepts": ["data_viz", "video", "web"], "default_fit": "fill", "missing_behavior": "warn" },
      "states": { "allow_live_override": true, "blackout_participates": false },
      "tags": ["situation", "primary", "command"]
    }
  ]
}
```
