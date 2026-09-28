# Changelog

All notable changes to survey-layout. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.1.0] - 2026-09-28

### Added
- Platform presets: `tiktok`, `instagram-reel`, `youtube-shorts`
  (1080×1920, union-safe 9:16 region set), `x-portrait` (1080×1350),
  `square` (1080×1080), `widescreen` (1920×1080), `legacy`
  (pixel-identical to survey-viz ≤ 0.17).
- `layout_for(platform, content=None)`: curated region rects
  (title/map/chart/caption/footer) as figure fractions, validated
  before return — unknown platforms/content fail fast.
- `validate_layout()`: the three invariants (inside canvas, clear of
  platform chrome, no region collisions; the caption chip may float
  over content but never over chrome).
- `fit_frame(frame, w, h, mode)`: `contain` / `cover` / `blur-fill`
  frame adaptation (Pillow; `pip install "survey-layout[fit]"`).
- `to_viz_canvas()` / `validate_canvas()`: duck-typed canvas dicts for
  `viz.render_viz(canvas=...)` — viz never imports this package.
- 38 tests; docs (README, PLATFORMS, INTEROP, ARCHITECTURE).
