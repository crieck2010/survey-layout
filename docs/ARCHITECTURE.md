# survey-layout architecture

```
src/layout/
├── __init__.py      # exports, __version__
├── platforms.py     # Platform dataclass; PLATFORMS registry (the data)
├── geometry.py      # rect math: overlaps / contains / to_pixels
├── regions.py       # layout_for() + validate_layout()
├── fit.py           # fit_frame(): contain / cover / blur-fill (needs Pillow)
└── interop.py       # to_viz_canvas() / validate_canvas() (duck-typed dicts)
```

## Design decisions

**Data, not code, per platform.** A platform is a canvas size, a list
of unsafe rects, and a region template — all plain data in
`platforms.py`. Adding a platform means adding a dict entry; the
validator proves it safe.

**Curated templates, validated — not a solver.** A constraint solver
would be cleverer and wrong in subtler ways. The region sets are
hand-tuned, and `validate_layout` asserts the three invariants (inside
canvas, clear of chrome, no collisions) on every `layout_for` call, so
curated data can never ship broken silently.

**Fractions everywhere.** Figure fractions `(left, bottom, width, height)`
are what `matplotlib`'s `add_axes` takes, so regions plug into
survey-viz with zero translation. Pixel conversion (`to_pixels`) exists
only for guides/overlays.

**Peers stay optional.** survey-viz consumes a plain dict, never this
package — the same optional-peer pattern as the rest of the suite.
`fit` is an extra because Pillow shouldn't be mandatory for canvas math.

**Honesty about chrome.** Platform UIs change; the measurements are
approximate and documented as such (docs/PLATFORMS.md). The 9:16
template clears the union of three platforms' chrome with margin, so
small redesigns stay safe.
