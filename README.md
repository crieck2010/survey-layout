> [!IMPORTANT]
> **Consolidated into [EarthWatch-Suite](https://github.com/crieck2010/earthwatch-suite)**  
> This engine has been consolidated into the unified [EarthWatch-Suite monorepo](https://github.com/crieck2010/earthwatch-suite). Active development, bug fixes, releases, and unified testing now live in earthwatch-suite. This repository is preserved as an archive.

# survey-layout

Platform-aware canvas layouts for vertical data-viz reels. Answers one
question: **given a target platform, where on the canvas may content go?**

`survey-layout` sits between the render and encode stages of the
reel-studio pipeline:

```
reel-studio → survey-viz → survey-currents → survey-animate / ffmpeg
                     ↑
              survey-layout (canvas + safe zones)
```

survey-viz renders frames; survey-animate encodes them. survey-layout
tells viz *what shape the frames are and where the title, map, chart,
caption, and footer may go* so platform chrome (TikTok's status bar,
the right-hand action rail, the bottom caption zone) never covers the
content.

## Install

```bash
pip install git+https://github.com/crieck2010/survey-layout
pip install "survey-layout[fit]"   # + Pillow, for fit_frame
```

Core (platforms, regions, geometry, interop) is **stdlib-only** — the
canvas math runs anywhere, including inside survey-viz's optional-peer
path. Only `fit_frame` needs Pillow.

## Quickstart

```python
from layout import get_platform, layout_for, to_viz_canvas, fit_frame

plat = get_platform("tiktok")
print(plat.aspect, plat.width, plat.height)   # 9:16 1080 1920

# Region rects (figure fractions) for a continuous-variable reel:
regions = layout_for("tiktok")
# {'title': (0.02, 0.815, 0.82, 0.095), 'map': ..., ...}

# Categorical variables draw no time-series chart:
regions = layout_for("tiktok", content=("title", "map", "caption", "footer"))

# The duck-typed dict survey-viz >= 0.18 consumes:
canvas = to_viz_canvas("youtube-shorts")
# {"platform": ..., "width": 1080, "height": 1920,
#  "regions": {name: [l, b, w, h]}, "unsafe": [...]}

import viz
frames, manifest = viz.render_viz(spec, field, series, out_dir="frames",
                                  canvas=canvas)

# Adapt an existing 1080x1920 master to square:
import numpy as np
from PIL import Image
frame = np.asarray(Image.open("frame_0001.png"))
sq = fit_frame(frame, 1080, 1080, mode="blur-fill")
```

## Platforms

| key | label | canvas | chrome handled |
|---|---|---|---|
| `tiktok` | TikTok | 1080×1920 | status bar, action rail, caption zone |
| `instagram-reel` | Instagram Reels | 1080×1920 | header, action rail, deep caption zone |
| `youtube-shorts` | YouTube Shorts | 1080×1920 | status bar, action rail, title zone |
| `x-portrait` | X (portrait 4:5) | 1080×1350 | nav bars |
| `square` | Square 1:1 | 1080×1080 | — (full bleed) |
| `widescreen` | Widescreen 16:9 | 1920×1080 | — (full bleed) |
| `legacy` | survey-viz default | 1080×1920 | — (pixel-identical to ≤ 0.17) |

The three 9:16 platforms share one region set that clears the **union**
of their chrome, so a single render is safe on all three. Chrome
measurements are community-measured approximations — see
[docs/PLATFORMS.md](docs/PLATFORMS.md).

## How it stays dataset-aware

The engine never sees a dataset. It takes a *content set* — which blocks
the render plan will draw — and returns regions for exactly those
blocks:

- continuous gridded variables → `title, map, chart, caption, footer`
- categorical variables (storm tracks, streamgages, earthquakes) →
  `title, map, caption, footer` (no chart; their colors carry meaning)

Which variable maps to which content set is survey-viz's render plan;
the contract table lives in [docs/INTEROP.md](docs/INTEROP.md).
Dataset *selection* (which input dataset best answers a description)
remains survey-viz's `parse_description` — layout consumes its output.

## API

- `get_platform(key) -> Platform` / `list_platforms()` / `PLATFORMS`
- `layout_for(platform, content=None) -> {name: (l, b, w, h)}` —
  validated before return; unknown platforms/content fail fast
- `validate_layout(platform, regions) -> [problems]`
- `to_viz_canvas(platform, content=None) -> dict` — the dict
  `viz.render_viz(canvas=...)` accepts; `validate_canvas(d)`
- `fit_frame(frame, w, h, mode="blur-fill"|"contain"|"cover")` — needs Pillow
- `layout.geometry` — `overlaps`, `contains`, `within_canvas`,
  `to_pixels`, `from_top_rect`

All coordinates are figure fractions `(left, bottom, width, height)`,
origin bottom-left — exactly what `fig.add_axes` takes.

## Tests

```bash
pytest tests/   # 38 tests, stdlib + numpy + pillow
```

## Limitations

- Chrome measurements are approximate and drift as platforms redesign;
  presets are versioned data, not ground truth. Override per need.
- `layout_for` returns curated templates, not a constraint solver —
  custom canvases need hand-tuned regions (validate with
  `validate_layout`).
- No text measurement: region rects reserve space; viz's font sizes are
  viz's business.

## License

MIT — see [LICENSE](LICENSE).
