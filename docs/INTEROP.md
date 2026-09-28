# survey-layout interop: peer contracts

## survey-viz ≥ 0.18 (`viz`)

`viz.render_viz(..., canvas=None)` accepts the dict built by
`layout.to_viz_canvas`:

```python
{"platform": "tiktok", "width": 1080, "height": 1920,
 "regions": {"title": [l, b, w, h], "map": [...], ...},
 "unsafe":  [[l, b, w, h], ...]}
```

* `width`/`height` set the figure size (at 100 dpi); `regions`
  overrides the axes rects for the shared chrome helpers
  (`_draw_title_block`, `_draw_caption`, `_draw_footer`) and the
  map/chart panels. Missing region keys fall back to the legacy
  9:16 rects.
* The canvas dict is duck-typed — viz never imports this package.
  Hand-written dicts are validated fail-fast (`validate_canvas`).
* Provenance: viz records `manifest["render"]["canvas"]`
  (platform, width, height, region names).

survey-viz ≤ 0.17 ignores the parameter (it doesn't exist); callers
must capability-check, exactly as reel-studio does for motion/audio.

## survey-animate (no change required)

Frames arrive from viz already at canvas size, so `render_video`
encodes them as-is — any aspect works. `layout.fit_frame` covers the
standalone case (adapting an existing master to another aspect before
encoding).

## reel-studio ≥ 0.5

* `studio/peers.py` wires `layout` as an optional fourth peer.
* `run_pipeline(..., platform="tiktok")` builds the canvas via
  `layout.to_viz_canvas` (graceful fallback to legacy when the peer
  is missing) and forwards it to `render_viz`.
* The app exposes a platform picker; the manifest's canvas record is
  shown under the result.

## Dataset → content-set contract

Dataset *selection* stays in survey-viz (`parse_description` chooses
the input dataset(s) that best answer a description). Layout consumes
the render plan's content set:

| survey-viz variable(s) | content set | notes |
|---|---|---|
| sst, ocean-color, era5-*, fires, sea-ice, precipitation, night-lights, topography, water-storage, currents, streamflow-timeseries | title, map, chart, caption, footer | continuous gridded fields with a context series |
| storm-tracks, streamflow (gage map), earthquakes | title, map, caption, footer | categorical: fixed scientific colors, no chart |

When viz gains a variable, its renderer declares its content set and
this table grows — the engine itself never changes.

## Scaling notes

* `layout_for` is pure and deterministic: same platform + content =
  same rects, safe to memoize per session.
* `fit_frame` is per-frame numpy/Pillow work; batch use should go
  through the same job isolation as encoding (one process per job).
* Region templates are data: adding a platform is a dict entry +
  validation, no code paths.
